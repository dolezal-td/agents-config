# agents-config

Doporučená výchozí konfigurace **Claude Code** pro začátečníky, od [AI pro smrtelníky](https://aiprosmrtelniky.cz). Dva soubory, které z Claude Code udělají použitelného parťáka: jak s tebou má mluvit (`CLAUDE.md`) a co smí dělat bez ptaní (`settings.json`).

## Co je uvnitř

- **`CLAUDE.md`**: globální instrukce: mluv česky, bez žargonu, ptej se, neimprovizuj, nedělej nevratné věci bez svolení. Je psaný v první osobě, jako bys ho diktoval ty: klidně si ho přepiš pod sebe, hlavně sekci **Kdo jsem** (jestli nejsi začátečník nebo děláš něco jiného než já).
- **`settings.json`**: bezpečná výchozí pravidla:
  - **čtení souborů a web** smí bez ptaní,
  - **osobní složky** (Desktop, Dokumenty, Stažené) a **`.env`** se nejdřív zeptá,
  - **nebezpečné věci** (mazání přes `rm -rf`, force push, `sudo`, čtení SSH klíčů a hesel, přepis shell configu) jsou zakázané.
  - Spouští se v režimu „accept edits": úpravy souborů jdou plynule, ale spuštění příkazů a citlivé akce se hlídají.

## Jak si to nalít

Obojí patří do složky `~/.claude` ve tvém domově (na Windows `%USERPROFILE%\.claude`).

```bash
# 1. Ujisti se, že složka existuje
mkdir -p ~/.claude

# 2. Zazálohuj si stávající (kdybys už něco měl, ať o to nepřijdeš)
cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.zaloha 2>/dev/null
cp ~/.claude/settings.json ~/.claude/settings.json.zaloha 2>/dev/null

# 3. Zkopíruj soubory
cp CLAUDE.md settings.json ~/.claude/
```

Pak **restartuj Claude Code** a je to.

## Skilly (volitelné)

Ve složce `skills/` je pár hotových **dovedností** (skill = návod, který Claude sám vytáhne, když se hodí). Nejsou povinné. Když nějakou chceš, zkopíruj její složku do `~/.claude/skills/`:

```bash
mkdir -p ~/.claude/skills
cp -r skills/contract-review ~/.claude/skills/
cp -r skills/youtube-research ~/.claude/skills/
cp -r skills/session-log ~/.claude/skills/
cp -r skills/end-of-day ~/.claude/skills/
```

- **`contract-review`**: kontrola smlouvy z pozice protistrany (rizika, nevýhodná ustanovení, co chybí). Funguje rovnou.
- **`youtube-research`**: najde k tématu videa, stáhne přepisy a vytěží z nich znalosti. Potřebuje nástroj `yt-dlp` (`brew install yt-dlp` na macOS, nebo `pip install yt-dlp`).
- **`session-log`**: na konci práce zapíše stručně, co se udělalo, do denního logu `~/.claude/logs/YYYY-MM-DD.md`, ať se to dá dohledat. Funguje rovnou.
- **`end-of-day`**: večerní úklid. Projde všechny dnešní Claude Code session napříč projekty i git a zkonsoliduje je do denního logu, plus drží seznam „otevřených smyček" (co zbývá dotáhnout). Funguje rovnou.

### Vývojové skilly (pro pokročilé)

Tahle sada je pro lidi, kteří s Claude Code reálně programují. Je to ucelený vývojový postup: od nápadu přes návrh, rozdělení práce a testy až po Git. Jádro je router `vyvoj`, který ostatní volá ve správném pořadí.

```bash
cp -r skills/vyvoj skills/grill-me skills/research skills/deep-research ~/.claude/skills/
cp -r skills/write-a-prd skills/prd-to-issues ~/.claude/skills/
cp -r skills/testing-strategy skills/mock-strategy skills/test-anti-patterns ~/.claude/skills/
cp -r skills/git-workflow skills/start-development ~/.claude/skills/
```

Co který dělá:

- **`vyvoj`**: router celého vývoje, volá ostatní skilly ve správném pořadí a hlídá brány (Git, testy, review).
- **`grill-me`**: adverzariální stress-test nápadu nebo plánu, než začneš stavět.
- **`research`** / **`deep-research`**: rychlá / hloubková rešerše tématu s aktuálními zdroji.
- **`write-a-prd`**: sepíše zadání nové funkce (PRD) přes interview.
- **`prd-to-issues`**: rozloží PRD na samostatné GitHub Issues.
- **`testing-strategy`** / **`mock-strategy`** / **`test-anti-patterns`**: jak testovat, jak izolovat závislosti, čeho se v testech vyvarovat.
- **`git-workflow`**: strukturovaný Git postup (branch, issue, commit, merge) s vysvětlením proč.
- **`start-development`**: inicializace nového vývojového projektu.

**Co k tomu potřebuješ:**

- **Plugin `superpowers`** (povinné pro plný postup): `vyvoj`, `testing-strategy` a `git-workflow` se opírají o skilly z tohoto pluginu (TDD, brainstorming, code review, Git worktrees). V Claude Code napiš `/plugin install superpowers@claude-plugins-official` a restartuj. Detaily: [obra/superpowers](https://github.com/obra/superpowers).
- **GitHub CLI `gh`** (povinné pro práci s issues): `write-a-prd`, `prd-to-issues` a `git-workflow` zakládají a spravují GitHub Issues. [Instalace gh](https://cli.github.com/).
- **Context7 MCP** (volitelné, doporučené): `vyvoj`, `research`, `deep-research` a `start-development` z něj tahají aktuální dokumentaci knihoven, ať Claude nepracuje ze zastaralé paměti.
- **Search MCP typu Tavily** (volitelné): `research` a `deep-research` ho umí využít pro hlubší pokrytí. Fungují i bez něj přes běžný web search.

### NotebookLM (extra)

Skill na ovládání Google NotebookLM (podcasty, kvízy, reporty z tvých zdrojů) tady záměrně **nekopírujeme**, je to cizí balík, který se sám aktualizuje. Nainstaluj si ho přímo:

```bash
pip install notebooklm-py
notebooklm skill install
```

Pak ho v Claude Code vyvoláš třeba „udělej mi podcast o…". Detaily: [notebooklm-py](https://github.com/teng-lin/notebooklm-py).

## Důležité

Tohle je **rozumný a bezpečný začátek**, ne neprůstřelná bezpečnost. `deny` pravidla blokují přístup i v automatickém režimu, ale chytrý útok přes vlastní skript nezastaví (na to je až sandbox). Uprav si konfiguraci podle sebe, klidně přidávej vlastní pravidla přes „Allow always" v promptu.
