---
name: git-workflow
description: Use when starting any code change, bug fix, or feature work in a git repository. Also use at end of session for GitHub housekeeping (issue cleanup, release check). Triggers before writing code, after merging, and when session-log calls it.
---

# Git Workflow

Strukturovaný Git/GitHub workflow. Claude Code je developer, který Git používá správně, vysvětluje proč, a ptá se jen na nevratné akce.

## Kdy se aktivuje

- Před jakoukoli změnou kódu
- Když session-log volá GitHub housekeeping
- Když vzniká plán a je potřeba vytvořit issues

**Výjimka:** Čistě obsahové edity (text v MDX, hodnota v configu) jsou triviální, Quick Fix cesta.

## Začátek práce

### 1. Git stav (vždy, bez ptaní)

```bash
git status && git branch --show-current
gh issue list --limit 10 --state open
git tag --sort=-v:refnum | head -3
git log $(git describe --tags --abbrev=0 2>/dev/null || echo HEAD~10)..HEAD --oneline
```

Řekni uživateli:
```
Git: branch main, 3 otevřené issues (#7, #8, #9), poslední tag v1.2.0, 5 commitů od tagu.
```

### 2. Klasifikace změny

**Triviální** (překlep, 1 řádek CSS, config hodnota):
→ Oprav, commitni na main, hotovo.

**Netriviální** (víc souborů, 30+ min, nová feature, bugfix):
→ Pokračuj krokem 3.

**Nejsi si jistý?** Řekni:
```
Tahle změna vypadá na [popis]. Řeším to jako quick fix na main,
nebo založím issue + branch? (Doporučuji [volbu], protože [důvod].)
```

### 3. Issue

Zkontroluj, jestli existuje:
```bash
gh issue list --search "klíčová slova"
```

**Existuje:** "Pracuji na issue #X: [title]."

**Neexistuje:** Vytvoř (revertovatelné, bez ptaní):
```bash
gh issue create --title "[title]" --body "[body]"
```
Řekni: "Vytvořil jsem issue #X: [title], protože [důvod, proč to není triviální]."

### 4. Branch nebo worktree

Default je běžná branch. Worktree zvol, když platí aspoň jedno:

**Worktree (izolovaný pracovní adresář na vlastní branchi):**
- Pustíš subagenty / paralelní agenty (editují soubory) → povinné, jinak si přepíšou práci.
- Potřebuješ main živou zároveň: běžící dev server, demo, porovnání chování, hotfix na main uprostřed feature.
- Delší feature, kterou budeš prokládat jinou prací → main zůstane čistý a hned použitelný (žádný stash dance).
- Background / asynchronní agent.

**Běžná branch:** rychlá sekvenční změna, doděláš a mergneš teď, main mezitím nepotřebuješ.

```bash
git checkout -b [type]/[popis]   # běžná branch
```

Typy: `fix/`, `feat/`, `refactor/`, `chore/`

Řekni proč: "Zakládám branch `feat/executive-summary`, protože zasáhne 5 souborů." nebo "Beru worktree místo branch, protože budu pouštět subagenty."

**Worktree mechaniku** předej skillu `using-git-worktrees` (umístění, kontrola `.gitignore`, `npm install`, baseline testy). Tento skill vyžaduje plugin **superpowers**; pokud ho nemáš, založ worktree ručně přes `git worktree add`.

**Pozor u web/JS projektů:** čerstvý worktree nemá gitignorované `.env` / `.env.local` ani `node_modules`. using-git-worktrees spustí `npm install`, ale env soubory zkopíruj ručně, jinak dev server v worktree spadne:
```bash
cp .env .env.local .worktrees/<branch>/ 2>/dev/null
```

### 5. Práce na branchi

- Commituj volně (WIP commity jsou OK na feature branch)
- Referuj issue: `WIP: scatter plot barvy (#17)`
- Na branchi se neboj experimentovat, nic z toho nejde na main

## Konec práce

### 6. Squash merge (výchozí)

Squash merge = všechny WIP commity na branchi se sloučí do jednoho čistého commitu na main.

```bash
git checkout main
git merge --squash [branch-name]
git commit -m "feat: popis změny (closes #17)"
```

Řekni: "Squash merge, 8 WIP commitů se sloučí do jednoho: 'feat: scatter plot barvy (closes #17)'. Issue #17 se zavře automaticky."

**Vždy používej `closes #X` nebo `fixes #X`** v commit message, GitHub automaticky zavře issue.

### 7. Push (ptej se)

```
Hotovo, testy prochází. Mám pushnout na main?
(Pokud máš nastavený automatický deploy, push ho spustí.)
```

Po schválení:
```bash
git push origin main
```

### 8. Release check

```bash
git log $(git describe --tags --abbrev=0)..HEAD --oneline | wc -l
```

**Navrhni release když:**
- 5+ commitů od posledního tagu
- Dokončena velká feature
- Výrazný bugfix v produkci

```
Od posledního tagu (v1.2.0) je 7 commitů:
- feat: executive summary
- fix: PDF export
- feat: nová sekce reportu
Navrhuji vydat v1.3.0. Mám vytvořit tag?
```

Po schválení:
```bash
git tag -a v1.3.0 -m "popis změn"
git push --tags
```

**Nenavrhuj release** po sérii drobných oprav (CSS, texty, překlepy).

### 9. Cleanup

```bash
git branch -d [branch-name]
```

## Session-log integrace

Když session-log volá git-workflow, proveď:

### GitHub housekeeping

```bash
gh issue list --state open
```

1. **Vyřešené issues:** Zkontroluj, jestli jsme v session vyřešili něco, co je ještě otevřené. Pokud ano, zavři:
   ```bash
   gh issue close [číslo] --comment "Vyřešeno v [commit hash]"
   ```

2. **Stale issues:** Issues otevřené déle než 7 dní bez aktivity, upozorni:
   ```
   Issue #3 (index rozptylu) je otevřený 5 dní bez aktivity. Mám zavřít?
   ```

3. **Release check:** Stejný jako krok 8.

4. **Stale branches:**
   ```bash
   git branch --merged main
   ```
   Smaž branches, které jsou už v main.

## Propojení s plánováním

Když vznikne plán s tasky (např. přes skill `writing-plans`, ten vyžaduje plugin **superpowers**):
1. Vytvoř issue pro každý logický celek (ne pro každý řádek)
2. Issues odkazují na plán: "Plán: docs/plans/2026-03-24-feature.md"
3. Řekni: "Vytvořil jsem 5 issues (#18–#22) z plánu. Plán zůstává primární zdroj pravdy, issues jsou pro tracking."

## Vzdělávací role

U KAŽDÉHO Git rozhodnutí řekni **proč**:
- "Zakládám branch, protože tahle změna zasáhne 3 soubory a bude trvat přes 30 minut."
- "Squash merge, 8 WIP commitů se sloučí do jednoho čistého. Na main bude vidět jen výsledek."
- "Navrhuji release, protože od posledního tagu je 7 commitů včetně nové feature."
- "Tohle je triviální (jeden překlep), commituju rovnou na main."
- "Worktree místo branch, protože budu pouštět subagenty a potřebuji izolaci."

## Co je revertovatelné (dělej sám)

- Vytvoření issue (`gh issue close` to vrátí)
- Vytvoření branch (`git branch -d` to smaže)
- Commity na feature branch (nejsou na main)
- Vytvoření tagu (`git tag -d` + `git push --delete origin tag`)

## Co vyžaduje potvrzení

- Push na main (může spustit automatický deploy)
- Merge do main
- Zavření issue (pokud nejsi jistý, že je vyřešený)
- Force push (nikdy bez explicitního souhlasu)
