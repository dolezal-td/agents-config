---
name: mock-strategy
description: Hierarchie volby izolace závislostí v testech (real > fake > stub > mock). Použij když test potřebuje izolovat externí dependency (DB, API, time, network, filesystem).
---

# Mock Strategy

## When to Use

Když test potřebuje izolovat externí dependency a ty si nejsi jistý, jak. Konkrétně:

- Test sahá do databáze
- Test volá HTTP API
- Test závisí na čase / `Date.now()`
- Test čte/zapisuje filesystem
- Test používá `setTimeout` / `setInterval`
- Test používá random hodnoty

**Iron Law:** **Mock = poslední možnost.** Vždy zkus o úroveň výš (real → fake → stub → mock).

## Hierarchie

```
real    → použij produkční dependency v testu (sqlite v RAM, real msw server)
  ↓
fake    → in-memory implementace stejného contractu (memfs, fake DB klient)
  ↓
stub    → canned odpověď pro konkrétní scénář (msw handler vrací JSON)
  ↓
mock    → spy + verify interakci (vi.fn(), jest.fn())
```

Každý krok dolů obětuje fidelity (test míň ověřuje real chování). **Real je nejlepší, mock nejhorší.** Pokud sklouzneš dolů, musíš obhájit, **proč** vyšší level nešel.

---

## Pravidla pro typické dependencies

### 1. Time / Date

```
ANO: vi.useFakeTimers() když test potřebuje deterministicky postoupit čas
NE:  fake timer pro test, který „kdesi uvnitř" něco s časem dělá
```

**Správně (deterministic):**
```ts
test("session expires after 30 minutes", () => {
  vi.useFakeTimers();
  vi.setSystemTime(new Date("2026-01-01T10:00:00"));

  const session = createSession();
  vi.advanceTimersByTime(30 * 60 * 1000 + 1);

  expect(isExpired(session)).toBe(true);
  vi.useRealTimers();
});
```

**Špatně:** mock celého `Date` modulu. Použij `setSystemTime`.

**Lepší vzor:** inject clock/timestamp jako parameter, ať produkce i test používají stejnou seam.

```ts
function createSession(now: number = Date.now()) { ... }

test("...", () => {
  const session = createSession(1704096000000);
  ...
});
```

### 2. HTTP / API klient

```
ANO: msw (Mock Service Worker), intercepting na network úrovni
ANO: nock pro Node.js
NE:  vi.mock("axios") nebo vi.mock("./můj-api-client")
```

**Správně (msw):**
```ts
import { setupServer } from "msw/node";
import { http, HttpResponse } from "msw";

const server = setupServer(
  http.get("https://api.example.com/users/:id", ({ params }) => {
    return HttpResponse.json({ id: params.id, name: "Anna" });
  }),
);

beforeAll(() => server.listen());
afterAll(() => server.close());

test("getUser fetches user data", async () => {
  const user = await getUser("123");
  expect(user.name).toBe("Anna");
});
```

Tvůj `getUser` testuje **real** axios/fetch logiku včetně serializace, headers, error handling. Mock jen network layer.

### 3. Database

```
PRIORITA:
  1. test containers (Postgres / MySQL / Redis ve skutečné instanci)
  2. sqlite v RAM (`:memory:`), pokud schema kompatibilní
  3. fake repository (in-memory implementace stejného interface)
  4. mock DB klient, JEN pro unit test, který testuje error handling DB
```

**Správně (testcontainers):**
```ts
import { PostgreSqlContainer } from "@testcontainers/postgresql";

let container: StartedPostgreSqlContainer;
let db: Db;

beforeAll(async () => {
  container = await new PostgreSqlContainer().start();
  db = await connect(container.getConnectionUri());
  await migrate(db);
});

afterAll(async () => {
  await container.stop();
});

test("findUserById returns null for unknown id", async () => {
  expect(await db.users.findById("unknown")).toBeNull();
});
```

**Pomalý setup, ale**: testuje real SQL, real constraints, real transakce.

### 4. Filesystem

```
ANO: tmpdir + real fs pro integration test
ANO: memfs pro pure logic, který shuffluje cesty bez actual I/O
NE:  vi.mock("fs") pro modul, který je hlavně o filesystem operacích
```

**Správně (tmpdir):**
```ts
import { mkdtemp, rm } from "fs/promises";
import { tmpdir } from "os";
import { join } from "path";

let dir: string;
beforeEach(async () => {
  dir = await mkdtemp(join(tmpdir(), "test-"));
});
afterEach(async () => {
  await rm(dir, { recursive: true, force: true });
});

test("writeReport creates file at expected path", async () => {
  await writeReport(dir, { id: "abc", text: "hello" });
  expect(await readFile(join(dir, "abc.txt"), "utf-8")).toBe("hello");
});
```

### 5. Random

```
PRAVIDLO: random je side-effect. Inject přes parameter.

function generateId(rng: () => number = Math.random) {
  return Math.floor(rng() * 1e9).toString(36);
}

test("...", () => {
  const id = generateId(() => 0.5);
  expect(id).toBe("ksjm12"); // deterministic
});
```

Pokud nemůžeš inject, použij seeded random library (`seedrandom`).

---

## Anti-patterns specifické pro mocking

### „Mock vlastní modul", NIKDY

```ts
// ❌ NIKDY
vi.mock("./můj-pricing-modul", () => ({
  calculateTotal: vi.fn(() => 100),
}));
```

Pokud cítíš nutkání, znamená to:
- buď tvůj kód je špatně strukturovaný (chybí seam pro injekci)
- nebo testuješ příliš vysoko (měl bys testovat na nižší úrovni přímo `calculateTotal`)

**Fix:** restrukturalizuj, předej `calculateTotal` jako parameter, nebo testuj komponentu, která `calculateTotal` používá, s **real** `calculateTotal`.

### Mock + assert na mock = test ničeho

```ts
const fn = vi.fn(() => 42);
expect(fn()).toBe(42); // ← testuje, že vi.fn() vrací to, co jsme mu řekli
```

Pokud jediná assertce je na mock návratovku, test nic neověřuje.

---

## Decision tree

```
Potřebuju test izolovat dependency.
├── Je to TIME?
│   └── inject clock/timestamp jako parameter NEBO vi.useFakeTimers()
├── Je to HTTP?
│   └── msw / nock, neumoc axios/fetch přímo
├── Je to DB?
│   ├── Schema kompatibilní s sqlite? → sqlite :memory:
│   └── Komplexní SQL? → testcontainers
├── Je to FS?
│   └── tmpdir + real fs (integration) NEBO memfs (pure logic)
├── Je to RANDOM?
│   └── inject rng() jako parameter
└── Je to JINÝ MODUL VE STEJNÉM REPO?
    └── STOP. Restrukturalizuj kód, neumoc.
```

## Reference

- [Mock Service Worker (msw)](https://mswjs.io/), HTTP intercepting
- [Testcontainers](https://node.testcontainers.org/), real DB v Dockeru
- [Trail of Bits, testing skills](https://github.com/trailofbits/skills)
- [Christopher Meiklejohn, anti-patterns](https://christophermeiklejohn.com/ai/claude/2026/03/08/claude-tested-everything-except-the-one-thing-that-mattered.html)
