---
name: end-of-day
description: Finální večerní úklid denního logu. Projde všechny dnešní Claude Code session napříč projekty a dnešní git commity, zkonsoliduje je do `~/.claude/logs/YYYY-MM-DD.md` (merge, nikdy nepřepisuje ruční úpravy) a dotáhne sekci Otevřené smyčky. Použij na konci dne, když uživatel řekne „konec dne", „shrnutí dne", „ukliď den", nebo zadá /end-of-day.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# End of Day

Večerní průchod celým dnem. Posbírá práci ze **všech** dnešních Claude Code session (napříč projekty), zkonsoliduje ji do denního logu a hlavně **dotáhne otevřené smyčky**, aby šlo druhý den plynule navázat. Veškerý zápis je lokální markdown.

## Vztah k session-log

| | `session-log` | `end-of-day` |
|---|---|---|
| Kdy | Po každé session, průběžně | 1× večer, finální |
| Zdroj | Jen aktuální konverzace | Všechny dnešní session, všechny projekty |
| Zápis | Append posunů | Merge + konsolidace |
| Otevřené smyčky | Neřeší | Aktivně udržuje |

`end-of-day` nenahrazuje `session-log`. Bere, co log za den nasbíral, doplní chybějící z ostatních session a gitu a dotáhne smyčky.

## Postup

### 1. Datum

```bash
date +"%Y-%m-%d"   # cesta k ~/.claude/logs/YYYY-MM-DD.md
```

### 2. Načti existující log jako základ pravdy

Přečti `~/.claude/logs/YYYY-MM-DD.md`.

- **Existuje** → tohle je základ. Zachovej všechno: ruční poznámky, vlastní formulace, sekce navíc.
- **Neexistuje** → vytvoř nový s hlavičkou `# YYYY-MM-DD` a sekcí `## Posuny`.

### 3. Posbírej dnešní session napříč projekty

Claude Code ukládá přepisy session jako `.jsonl` do `~/.claude/projects/<projekt>/`. Najdi ty dnešní:

```bash
# Dnešní session soubory napříč všemi projekty
find ~/.claude/projects -name "*.jsonl" -newermt "$(date +%F)T00:00:00" 2>/dev/null
```

Z každého souboru vytáhni, co se dělo. Soubory bývají velké a podrobné, **nečti je celé**. Stačí reálné uživatelské zprávy (zadání) a jejich časy:

```bash
# Lidské prompty s časem z jedné session (vyžaduje jq)
jq -r 'select(.type=="user" and (.message.content|type=="string")) | "\(.timestamp)  \(.message.content)"' SESSION.jsonl 2>/dev/null
```

Filtr `message.content|type=="string"` schválně vynechá řádky, kde je obsah pole (to jsou výsledky nástrojů, ne lidské zadání), takže zůstanou jen skutečné prompty. Časy jsou v UTC (ISO formát s `Z`); pro zápis je převeď na místní čas, nebo použij přibližný čas a označ ho.

Z aktuální session (tu máš v kontextu) bereš posuny rovnou.

### 4. Posbírej dnešní git commity

Adresáře, kde se dnes pracovalo, zjistíš z `cwd` v dnešních session:

```bash
find ~/.claude/projects -name "*.jsonl" -newermt "$(date +%F)T00:00:00" \
  -exec sh -c 'grep -m1 -o "\"cwd\":\"[^\"]*\"" "$1"' _ {} \; 2>/dev/null | sort -u
```

Pro každý takový adresář vytáhni dnešní commity:

```bash
git -C "<adresář>" log --since="$(date +%F) 00:00" --oneline 2>/dev/null
```

### 5. Merge do denního logu (jádro skillu)

**Merge, nikdy replace.** Pravidla:

- Existující obsah je základ pravdy. Když soubor existuje, nikdy nezačínej z prázdna.
- **Přidej** jen posuny, co v logu reálně chybí.
- **Uprav** existující záznam jen když se vyvinul (doplň commit hash, dotažení rozdělané věci).
- **Nesahej** na ruční poznámky a vlastní formulace.
- Deduplikace je **významová**, ne porovnání řádků. Stejný posun ze session i z gitu = jeden řádek.
- **Konsoliduj:** rozházené posuny seskup pod společný projekt, když to dává smysl.

Formát řádku je stejný jako u `session-log`:

```
- **HH:MM** Krátká věta, co se stalo `commit_hash`
```

### 6. Otevřené smyčky

Sekce `## Otevřené smyčky` na konci logu. Udržuj ji aktivně:

- **Vyřešené** během dne → přesuň do Posunů (nebo zruš, pokud tam už jsou).
- **Pokračující** → ponech, doplň poslední stav a další krok.
- **Nové** z dnešních session (čeká na odpověď, „ještě musím", nedotažené rozhodnutí, otázka bez odpovědi) → přidej.
- Každá smyčka: konkrétní akční formulace.

Žádné časové odhady ani prioritizaci, to si dělá uživatel sám.

### 7. Git commit *(jen když je co)*

Denní log je lokální soubor v `~/.claude/`, většinou není ve verzování, takže tenhle krok často odpadne. Pokud uživatel svoje logy verzuje a `git status` hlásí změny, nabídni commit.

### 8. Ověř a vypiš

Vypiš uživateli: kolik session a projektů prošlo, co přibylo nebo se upravilo, stav otevřených smyček (kolik vyřešeno / pokračuje / nových). Přeskočené kroky zmiň proč.

## Časté chyby

| Chyba | Oprava |
|---|---|
| Replace místo merge | Existující log je základ pravdy, jen doplňuj |
| Smazání ruční poznámky | Nesahej na nic, co není z dnešních zdrojů |
| Projetí jen aktuální session | Projdi všechny dnešní `.jsonl` napříč projekty |
| Duplicitní posun ze session i z gitu | Významová deduplikace, jeden řádek |
| Čtení celého velkého `.jsonl` | Jen uživatelské zprávy a časy, ne celý soubor |
| Vymýšlení časů | Z časů v transkriptu, fallback přibližný čas s označením |
