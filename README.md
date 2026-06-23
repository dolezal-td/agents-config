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

### NotebookLM (extra)

Skill na ovládání Google NotebookLM (podcasty, kvízy, reporty z tvých zdrojů) tady záměrně **nekopírujeme**, je to cizí balík, který se sám aktualizuje. Nainstaluj si ho přímo:

```bash
pip install notebooklm-py
notebooklm skill install
```

Pak ho v Claude Code vyvoláš třeba „udělej mi podcast o…". Detaily: [notebooklm-py](https://github.com/teng-lin/notebooklm-py).

## Důležité

Tohle je **rozumný a bezpečný začátek**, ne neprůstřelná bezpečnost. `deny` pravidla blokují přístup i v automatickém režimu, ale chytrý útok přes vlastní skript nezastaví (na to je až sandbox). Uprav si konfiguraci podle sebe, klidně přidávej vlastní pravidla přes „Allow always" v promptu.
