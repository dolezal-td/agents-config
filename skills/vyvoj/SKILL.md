---
name: vyvoj
description: Orchestrátor vývojového lifecyclu. Router, který u netriviální vývojové práce volá správné dílčí skilly ve správném pořadí a hlídá povinné brány (git-workflow, testy, review). Použij když uživatel začíná vývoj nad rámec jednořádkové opravy, nebo zmíní „udělej feature", „naprogramuj", „implementuj", „postav appku", „postav web", „rozjeď projekt", „pracuj na issue", „oprav bug" (víc souborů), „zrefaktoruj", „pojď to naprogramovat".
---

# Vývoj: orchestrátor vývojového lifecyclu

Router pro veškerou netriviální vývojovou práci. Sám nic neimplementuje. Pozná, kde v lifecyclu jsi, předá řízení správnému dílčímu skillu a u každé brány čeká na tvoje slovo. Běží v hlavním vlákně.

## Quick start

1. **Triviální, nebo ne?** Překlep, 1 řádek, config hodnota → tohle přeskoč, rovnou `git-workflow` quick fix. Feature, bugfix přes víc souborů, nový projekt, refactor → pokračuj.
2. **Nový projekt?** Když ještě nic nestojí, nejdřív `start-development` (PLAN.md + CLAUDE.md), pak zpět sem.
3. **Povinná brána:** před JAKOUKOLI změnou kódu invokuj `git-workflow` (rozhodne branch/issue/worktree). Není volitelné. (Tvrdé vynucení dodá budoucí PreToolUse hook, zatím to drží tahle instrukce.)
4. **Najdi fázi** v mapě níž a předej řízení jejímu skillu. Fáze se přeskakují vědomě, ne vynechávají naslepo.

## Lifecycle mapa

| Fáze | Skill(y) | Kdy přeskočit |
|---|---|---|
| Nápad | (tvůj vstup) | nikdy |
| Presearch | `research` / `deep-research` / Context7 | u známého tématu; u nového nebo rychle se měnícího hledej (princip „Ověřuj než tvrdíš") |
| Grill | `grill-me` | u jasného malého úkolu; jinak na mlhavý záměr nebo stress-test plánu |
| PRD | `write-a-prd` | u jednoduchého bugfixu; jinak u feature s víc rozhodnutími |
| Issues | `prd-to-issues` | když není PRD ke krájení na vertical slices |
| TDD | `testing-strategy` (router → `test-driven-development`, `mock-strategy`, `test-anti-patterns`) | nikdy u logiky; nech testing-strategy vybrat vrstvu |
| PR | `git-workflow` + `subagent-driven-development` (paralelní slices) | nepřeskakuj |
| Review | `requesting-code-review` + skill `code-review` | nepřeskakuj |
| Merge | `git-workflow` (squash) + `finishing-a-development-branch` | nepřeskakuj |
| Loop | zpátky na Nápad na další slice | když je dílo hotové |

> Pozn.: skilly `test-driven-development`, `subagent-driven-development`, `requesting-code-review`, `finishing-a-development-branch` a `using-git-worktrees` pocházejí z pluginu **superpowers** (musí být nainstalovaný).

## Větve

- **Frontend (web/React/HTML/CSS):** před implementací nejdřív promysli vizuální design (layout, typografie, hierarchie), až pak kóduj. Testuj přes `npx playwright` CLI, ne MCP.
- **Knihovna/framework/SDK/CLI:** Context7 (`resolve-library-id` → `query-docs`), ne knowledge cutoff.
- **Paralelní práce / subagenti:** `git-workflow` zvolí worktree (`using-git-worktrees`, plugin superpowers).

## Checkpointy (čekej na souhlas)

- Po PRD: „Sedí to, krájím na issues?"
- Po breakdownu issues: schválení granularity.
- Před push/merge: `git-workflow` se ptá sám, nech ho.

## Common mistakes

| Chyba | Oprava |
|---|---|
| Reimplementuju, co umí leaf skill | Zavolej skill a předej řízení, neopisuj jeho obsah sem |
| Spustím celý lifecycle na drobnou opravu | Triviální → `git-workflow` quick fix, žádné PRD/issues |
| Přeskočím `git-workflow` a začnu editovat | Brána je první, vždy. Bez branch/issue se kódu nedotýkej |
| Tiše vynechám fázi | Přeskočení řekni nahlas i s důvodem |
