---
name: session-log
description: Zapiš stručné shrnutí právě dokončené session do denního logu, aby se dalo dohledat, co se kdy udělalo. Append posuny do `~/.claude/logs/YYYY-MM-DD.md`. Použij na konci ucelené práce nebo když uživatel řekne „zaloguj", „konec", „hotovo", „to je vše", nebo zadá /session-log.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Session log

Zapiš, co podstatného v téhle session vzniklo, aby to šlo později dohledat. Veškerý zápis je lokální markdown, jeden soubor na den.

## Kdy spustit

- **Automaticky:** uživatel řekne „hotovo", „to je vše", „konec", „zaloguj to".
- **Nabídni:** po dokončení uceleného kusu práce nebo větším commitu. Stručně: „Hotové [souhrn]. Mám to zalogovat?"
- **Explicitně:** přes `/session-log`.

## Postup

### 1. Zrekapituluj session

Aktuální konverzaci máš v kontextu, takže rekapituluješ z ní. Najdi **posuny**: co vzniklo, opravilo se, přestavělo, zdokumentovalo, rozhodlo. Jedna ucelená akce = jeden řádek.

Piš jen ověřitelná fakta, nevymýšlej. Když session byla jen dotazy a odpovědi (žádné změny souborů), zaloguj i tak jedním řádkem, co se zjišťovalo, ať jde session dohledat.

### 2. Zjisti datum a kam logovat

```bash
date +"%Y-%m-%d"   # YYYY-MM-DD pro název souboru
date +"%H:%M"      # aktuální čas pro řádek posunu
```

**Default umístění:** `~/.claude/logs/YYYY-MM-DD.md`. Když složka `~/.claude/logs/` neexistuje, vytvoř ji:

```bash
mkdir -p ~/.claude/logs
```

> Pokud uživatel zmínil vlastní místo na logy (deníkový soubor, složka `docs/`, poznámkový systém), respektuj ho místo defaultu.

### 3. Git commit *(jen když je co)*

Spusť pouze pokud `git status` hlásí necommitnuté změny relevantní k téhle session. Pokud nic, přeskoč.

1. `git status`
2. `git add` konkrétních souborů + commit (stručně, v jazyce uživatele)
3. push jen pokud o něj uživatel stojí (ptej se)
4. Zapamatuj si zkrácený commit hash (7 znaků) pro řádek posunu.

Pro netriviální změny použij skill `git-workflow`, pokud je k dispozici.

### 4. Zapiš posun(y) do logu

Cesta: `~/.claude/logs/YYYY-MM-DD.md`.

**Když soubor neexistuje**, vytvoř ho s hlavičkou:

```markdown
# YYYY-MM-DD

## Posuny
```

**Když existuje**, jen **připoj** nové řádky pod `## Posuny`. Nikdy nepřepisuj, co tam už je.

**Formát řádku:**

```
- **HH:MM** Krátká věta, co se stalo `commit_hash`
```

- Čas je nepovinný, ale užitečný (z kroku 2).
- Commit hash jen když nějaký vznikl.
- Když chceš, můžeš na začátek dát volitelný štítek v hranatých závorkách, např. `[oprava]`, `[nová funkce]`, `[research]`. Není povinný.

**Příklad:**

```markdown
# 2026-06-23

## Posuny

- **14:30** Přidán export do PDF, opraveno zalamování dlouhých tabulek `a1b2c3d`
- **16:00** [research] Ověřeno, že knihovna X už podporuje streamování, není potřeba vlastní řešení
```

### 5. Ověř a vypiš

1. Zkontroluj, že je posun v logu (přečti konec souboru).
2. Vypiš uživateli: co bylo zalogováno a kam (cesta), plus commit hash, pokud nějaký byl.

## Pravidla

- Jedna ucelená akce = jeden řádek posunu.
- **Append, nikdy nepřepisuj** existující obsah.
- Piš jen ověřitelná fakta.
- Komunikuj v jazyce uživatele.
- Žádné externí služby, všechno je lokální markdown.
