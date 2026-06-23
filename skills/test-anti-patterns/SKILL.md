---
name: test-anti-patterns
description: Review skill PŘED commitem testů. Detekuje 8 AI antipatternů v testech (mock vlastního kódu, snapshot updated to broken, happy path only, atd.), testy které vypadají dobře, ale netestují nic.
---

# Test Anti-Patterns

## When to Use

**Po napsání nebo úpravě testu, PŘED git commit.** Konkrétně:

- Než spustíš `git add` na test soubory
- Když AI vygenerovala testy a ty si je chceš přečíst „jen jako pojistku"
- Když review dependabot PR / generated PR s testy uvnitř
- Když se ti zdá, že „je toho hodně testů, asi je to dobře pokryté"

**Iron Law:** AI píše testy, které vypadají dobře a netestují nic. Christopher Meiklejohn to zformuloval jako „Claude tested everything except the one thing that mattered." Tento skill detekuje patterny, které tomu vedou.

## Workflow

```
1. Otevři každý nový/upravený test
2. Pro každý test projeď 8 antipatternů níže
3. Pro každý hit: oprav PŘED commitem (ne later, ne TODO)
4. Pokud 3+ antipatterny v jednom testu → smaž test a napiš znovu
```

---

## 8 AI antipatternů

### 1. Test bez assertce nebo assertce na truthiness

**Špatně:**
```ts
test("user creation works", async () => {
  const user = await createUser({ name: "Anna" });
  expect(user).toBeTruthy();
});
```

`toBeTruthy()` projde i když `createUser` vrátí `{}` nebo `"banán"` nebo `42`. **Nic netestuje.**

**Správně:**
```ts
test("createUser persists name and assigns id", async () => {
  const user = await createUser({ name: "Anna" });
  expect(user.id).toMatch(/^usr_/);
  expect(user.name).toBe("Anna");
  expect(await db.users.findById(user.id)).toEqual(user);
});
```

**Detection:** grep `toBeTruthy|toBeDefined|not.toBeNull|not.toBeUndefined`, 90 % výskytů jsou anti-pattern.

---

### 2. Test coupled to implementation (mock vlastního kódu)

**Špatně:**
```ts
import { calculateTotal } from "./pricing";
vi.mock("./pricing", () => ({
  calculateTotal: vi.fn(() => 100),
}));

test("order shows total", () => {
  render(<OrderSummary order={fakeOrder} />);
  expect(screen.getByText("100 Kč")).toBeInTheDocument();
});
```

Mockoval jsi vlastní modul. Test prochází, ale neověřuje **nic**, ani UI, ani pricing. Pokud změníš logiku v `calculateTotal`, test pořád projde.

**Správně:** ne-mockuj. Použij real `calculateTotal` a fake order data.

```ts
test("order shows total of two items at 50 Kč", () => {
  const order = { items: [{ price: 50 }, { price: 50 }] };
  render(<OrderSummary order={order} />);
  expect(screen.getByText("100 Kč")).toBeInTheDocument();
});
```

**Detection:** grep `vi\.mock\(["']\./` nebo `jest\.mock\(["']\./`, mock relativní cesty = mock vlastního kódu = červená vlajka.

---

### 3. Snapshot updated to match broken output

**Symptom:** snapshot test failuje, AI rovnou volá `--updateSnapshot` (nebo `-u`) a commit je „update snapshot".

**Špatně:** AI nikdy nevyšetřila, **proč** snapshot failuje. Možná je to bug v kódu, který právě „uložila" do baseline.

**Správně:**
1. Snapshot fail → otevři diff, podívej se, co se změnilo
2. **Je změna úmyslná?** Pak update.
3. **Není?** Pak fix kódu, ne fix testu.

**Detection:** v PR / commit grep `snapshot updated|--updateSnapshot|jest -u`. Bez explicitního „why" v commit message = anti-pattern.

---

### 4. Test modified instead of fixing implementation

**Symptom:** test failuje. AI změní `expect(x).toBe(5)` na `expect(x).toBe(6)`, protože „kód teď vrací 6". Commit „fix test".

**Iron Law:** Test reprezentuje contract. Pokud failuje, nejdřív se ptej **„proč se změnilo chování?"**. Modifikace testu je akceptovatelná **jen** při explicitním requirement change (a pak commit message musí říct, čí change to byl).

**Detection:** test soubor a implementace soubor v jednom commitu, kde test byl loosened (less strict). Vždy podezřelé.

---

### 5. Flaky retry without root cause

**Špatně:**
```ts
// vitest.config.ts
test: { retry: 3 }
```

Nebo v Playwright `.spec.ts`:
```ts
test.describe.configure({ retries: 3 });
```

Flaky test je **bug**, ne nuance. Retry maskuje bug a generuje noise. Tým se naučí ignorovat fails. Real bug pak proklouzne.

**Správně:**
1. Označ test `test.skip` nebo přidej tag `@flaky`
2. Vytvoř issue „test X is flaky, root-cause it"
3. Fix root cause (race condition, real timer místo fake, network call bez msw, atd.)
4. Pak `test.skip` zruš

**Detection:** `retry: \d+` v test configu nebo `test.describe.configure` s `retries`. Flag for review.

---

### 6. `toHaveBeenCalled()` bez business assertion

**Špatně:**
```ts
test("submit triggers handler", () => {
  const onSubmit = vi.fn();
  render(<Form onSubmit={onSubmit} />);
  fireEvent.click(screen.getByText("Odeslat"));
  expect(onSubmit).toHaveBeenCalled();
});
```

OK, handler se zavolal. Ale **s čím**? Co ten handler předpokládá ve form datech? Test prochází, i když form neposílá nic užitečného.

**Správně:**
```ts
expect(onSubmit).toHaveBeenCalledWith({
  email: "anna@example.com",
  consent: true,
});
```

**Detection:** grep `toHaveBeenCalled\(\)` (bez `With`). Skoro vždy red flag.

---

### 7. Happy path only, žádné edge cases

**Symptom:** AI napíše `test("creates user with valid input")` a hotovo. Žádný test pro: prázdný input, příliš dlouhý input, duplicitní email, race condition, network error, malformed payload.

**Pravidlo:** každá unit dostane **alespoň 1 happy + 2 edge cases**. Pro security-sensitive kód víc.

**Checklist edge cases:**
- Prázdný / null / undefined vstup
- Hraniční hodnoty (0, -1, max int, prázdný string vs. mezera)
- Duplicate / concurrent
- Invalid format / type
- Failure dependencies (DB down, API timeout, network error)

**Detection:** test soubor s 1-2 testy pro modul s víc než trivialní logikou. Flag.

---

### 8. Test, který jen opakuje implementaci jinými slovy

**Špatně:**
```ts
function applyDiscount(price: number, percent: number): number {
  return price * (1 - percent / 100);
}

test("applyDiscount calculates discount", () => {
  const result = applyDiscount(100, 10);
  expect(result).toBe(100 * (1 - 10 / 100));
});
```

Test používá **stejnou formuli** jako implementace. Pokud je formule špatně, test taky přijde špatně. **No value add.**

**Správně:** assert hodnotu, ne výpočet.

```ts
test("10% discount on 100 is 90", () => {
  expect(applyDiscount(100, 10)).toBe(90);
});

test("0% discount returns full price", () => {
  expect(applyDiscount(100, 0)).toBe(100);
});

test("100% discount returns 0", () => {
  expect(applyDiscount(100, 100)).toBe(0);
});
```

**Detection:** test obsahuje stejné výrazy jako implementace (matematické operace, regex, string manipulace). Test má říkat **co**, ne **jak**.

---

## Workflow shrnutí

```
PRO KAŽDÝ NOVÝ / UPRAVENÝ TEST:

1. Hledej assertce → toBeTruthy / toBeDefined → red flag
2. Hledej vi.mock("./...") nebo jest.mock("./...") → red flag
3. Snapshot updates bez explicitního „why" v commit msg → red flag
4. Test loosened ve stejném commitu jako change implementation → red flag
5. retry: N v configu → red flag, root-cause it
6. toHaveBeenCalled() bez With → red flag
7. Test/file ratio: 1 test na modul s ≥3 funkcemi → red flag (chybí edge cases)
8. Test obsahuje stejnou formuli jako kód → red flag

3+ red flags v testu → smaž a napiš znovu od scratch.
```

## Reference

- [Christopher Meiklejohn: Claude Tested Everything Except the One Thing That Mattered](https://christophermeiklejohn.com/ai/claude/2026/03/08/claude-tested-everything-except-the-one-thing-that-mattered.html)
- [Trail of Bits skills](https://github.com/trailofbits/skills), mutation thinking pro testy
- [Matt Pocock, TDD](https://github.com/mattpocock/skills), vertical slicing principle
