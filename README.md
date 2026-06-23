# agents-config

Doporučená výchozí konfigurace **Claude Code** pro začátečníky, od [AI pro smrtelníky](https://aiprosmrtelniky.cz). Dva soubory, které z Claude Code udělají použitelného parťáka: jak s tebou má mluvit (`CLAUDE.md`) a co smí dělat bez ptaní (`settings.json`).

## Co je uvnitř

- **`CLAUDE.md`**: globální instrukce: mluv česky, bez žargonu, ptej se, neimprovizuj, nedělej nevratné věci bez svolení.
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

## Důležité

Tohle je **rozumný a bezpečný začátek**, ne neprůstřelná bezpečnost. `deny` pravidla blokují přístup i v automatickém režimu, ale chytrý útok přes vlastní skript nezastaví (na to je až sandbox). Uprav si konfiguraci podle sebe, klidně přidávej vlastní pravidla přes „Allow always" v promptu.
