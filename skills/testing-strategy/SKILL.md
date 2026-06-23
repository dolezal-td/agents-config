---
name: testing-strategy
description: Router skill PŘED jakýmkoliv testovacím úkolem. Klasifikuje úkol (deterministic vs. non-deterministic, pure vs. I/O, UI vs. API vs. logic), vybere vrstvu testovací pyramidy, rozhodne mock strategii a nasměruje na konkrétní specializovaný skill.
---

# Testing Strategy

## When to Use

Vždy **před** jakýmkoli testovacím úkolem. Konkrétně když:

- Začínáš implementovat feature a přemýšlíš, jaký test napsat první
- Reaguješ na bug a chceš ho pokrýt regression testem
- Refactoruješ a chceš ujistit, že nerozbíjíš chování
- Někdo (uživatel) řekne „napiš testy" / „přidej coverage" / „proč to nefunguje"

**Bez tohoto skillu** AI default sklouzne k „mock everything → unit test → 100% coverage", což znamená testy, které validují mock místo kódu.

## Iron Law

```
PŘED napsáním prvního testu odpověz na 3 otázky:
1. Co testuji, chování, nebo implementaci?
2. Jakou vrstvu pyramidy potřebuji?
3. Mock = poslední možnost. Jakou mám alternativu?
```

Pokud nedokážeš odpovědět, **STOP**. Test napsaný bez těchto odpovědí bude buď k ničemu, nebo aktivně škodlivý.

## Klasifikační rozhodovací strom

### 1. Klasifikuj kód, který testuješ

| Kategorie | Příklad | Doporučená vrstva |
|---|---|---|
| **Pure logic** (in → out, žádný I/O) | parser, validator, formatter, calculator | Unit (rychlé, hodně cases), zvaž property-based testing |
| **I/O bound** (DB, HTTP, fs) | repository, API client, file processor | Integration (s real DB / msw) |
| **UI komponenta** (state, rendering) | React component, page | Component test (testing-library) + E2E pro flows |
| **End-to-end flow** (multi-page, auth, atd.) | „uživatel se zaregistruje, uvidí dashboard" | E2E (Playwright), málo, ale široké |
| **Non-deterministic** (LLM, ML, random) | prompt engineering, scoring | Eval harness, ne assert-on-string |
| **Security-sensitive** | auth, RBAC, input validation | + dedikovaný security review testů |

### 2. Vyber vrstvu pyramidy

```
        /\
       /E2\        ← málo, široké flows (Playwright)
      /----\
     /Compo\       ← UI komponenty (testing-library)
    /------\
   /Integr  \      ← I/O bound s real deps (DB, msw)
  /----------\
 /   Unit     \    ← pure logic, hodně, rychlé
/--------------\
```

Pravidlo: **začni co nejníž**. Pokud je to pure logic, nikdy ho netestuj E2E. Pokud je to UI flow, nikdy ho netestuj jen unit testem na handler.

### 3. Rozhodni mock strategii

```
real > fake (in-memory) > stub (canned response) > mock (verify interaction)
```

Zkus nejdřív **real** (sqlite v RAM, msw pro HTTP, fake clock). Mock až když nic jiného nejde. Pokud cítíš nutkání mockovat **vlastní modul** (`vi.mock('./můj-helper')`), **STOP**, restrukturalizuj kód, nebo testuj přes seam, který už máš (function parameter, dependency injection).

→ Pro detail viz skill `mock-strategy`.

## Workflow

```
1. Přečti zadání → klasifikuj kód podle tabulky výše
2. Vyber vrstvu pyramidy (preferuj nižší)
3. Rozhodni mock strategii (viz mock-strategy skill)
4. Napiš JEDEN failing test (vertical slice, jedno chování end-to-end)
5. Spusť ho, ujisti se, že failuje ze správného důvodu
6. Implementuj minimum, ať projde
7. Refactor (kód i test)
8. PŘED commitem: spusť test-anti-patterns skill na nové testy
```

## Decision shortcuts

- **„Píšu novou feature"** → start s vertical slice E2E nebo komponentovým testem (vidět flow, pak unit testy na detaily uvnitř)
- **„Fixuju bug"** → nejdřív regression test, který bug reprodukuje (red), pak fix (green)
- **„Refactoruju"** → před refactorem ujisti, že existující testy pokrývají chování. Pokud ne, dopiš je. Pak refactor.
- **„AI feature"** → ne assert-on-string. Eval harness s rubric grader.

## Doporučené skills navazující

- [`mock-strategy`](../mock-strategy/SKILL.md), když test potřebuje izolovat dependency
- [`test-anti-patterns`](../test-anti-patterns/SKILL.md), review PŘED commitem
- Pro TDD workflow s explicitním vertical slicing viz `test-driven-development` (vyžaduje plugin superpowers)
- Pro gate „done" claim před uzavřením práce viz `verification-before-completion` (vyžaduje plugin superpowers)

## Reference

- [Matt Pocock, TDD skill](https://github.com/mattpocock/skills), vertical slicing manifest
- [Christopher Meiklejohn: Claude Tested Everything Except the One Thing That Mattered](https://christophermeiklejohn.com/ai/claude/2026/03/08/claude-tested-everything-except-the-one-thing-that-mattered.html), proč mock-everything ničí value
- [Anthropic Red: Property-based testing](https://red.anthropic.com/2026/property-based-testing/)
