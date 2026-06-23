---
name: research
description: Rychlý research tématu s aktuálními zdroji (2-3 min). Použij když uživatel potřebuje rešerši AI tématu, nástroje, technologie nebo trendu. Pro hloubkový výzkum použij /deep-research.
allowed-tools: WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Quick Research

Proveď rychlý research na téma: $ARGUMENTS

## Krok 1: Rozlož téma a prohledej web

Rozlož téma na 3 search queries (overview, tools/funkce, pricing) a spusť je paralelně přes WebSearch. Cílem je rychlý, ale čerstvý přehled.

- Preferuj zdroje z posledních zhruba 30 dní (date-gating). U rychle se měnících témat (ceny, verze) je čerstvost klíčová.
- Pokud máš k dispozici search MCP (např. Tavily), můžeš ho použít pro hlubší pokrytí a freshness skóre. Není podmínkou.
- Výsledky deduplikuj a seřaď podle relevance a data.

## Krok 2: Doplňkové WebSearch dotazy

Spusť 2-3 doplňkové WebSearch dotazy zaměřené na oblasti, které první kolo nemuselo pokrýt:
- Oficiální dokumentace a changelog
- Srovnávací články a benchmarky
- Komunitní diskuse

## Krok 3: Pokud je téma technické (knihovna, framework, API)

Použij Context7:
1. `resolve-library-id`: najdi knihovnu
2. `query-docs`: stáhni aktuální dokumentaci

## Krok 4: Pro klíčové stránky (ceníky, product pages)

Pokud výsledky obsahují URL ceníkových nebo produktových stránek,
použij WebFetch na 2-3 nejdůležitější a vytáhni aktuální data (ceny, plány, features).

## Krok 5: Syntéza

### ANTI-HALUCINAČNÍ PRAVIDLA (POVINNÁ)

```
KRITICKÉ INSTRUKCE:

1. Používej VÝHRADNĚ data z nalezených zdrojů (WebSearch, WebFetch, Context7).
2. NIKDY nedoplňuj informace z vlastních znalostí. Tvá tréninková data jsou zastaralá.
3. Každý fakt (cena, název modelu, funkce, datum) MUSÍ mít zdroj: [zdroj: URL]
4. Pokud informace NENÍ v nalezených zdrojích, napiš: "Nezjištěno z aktuálních zdrojů"
5. Pokud se dva zdroje LIŠÍ (jiná cena, jiný název), uveď OBĚ verze s odkazy.
6. NIKDY nepoužívej fráze jako "podle mých znalostí" nebo "pokud vím".
7. Názvy modelů, verzí a balíčků piš PŘESNĚ jak je uvádí zdroje, nehádej verze.
```

### Formát výstupu

```markdown
# Research: {téma}
> Datum: YYYY-MM-DD | Mód: quick | Zdroje: N stránek

## Executive Summary
[2-3 věty, hlavní zjištění, POUZE z nalezených zdrojů]

## Přehled

### {Položka 1}
- **Co to je:** [popis ze zdroje] [zdroj: URL]
- **Cena:** [ze zdroje] [zdroj: URL]
- **Klíčové funkce:** [ze zdroje]
- **Ideální pro:** [syntéza]

### {Položka 2}
...

## Srovnávací tabulka
| Nástroj | Cena | Klíčová funkce | Zdroj |
|---|---|---|---|

## Trendy a závěry
[Syntéza POUZE z nalezených zdrojů]

## Zdroje
- [URL], stručná anotace
```

## Krok 6: Ulož výstup

Ulož report do pracovní složky projektu (např. `working/research-{slug}-{date}.md`).

Kde `{slug}` je téma převedené na kebab-case (max 40 znaků) a `{date}` je YYYY-MM-DD.

## Pravidla

- Celý výstup česky
- Cituj zdroje u každého faktu
- Rozlišuj fakta (ze zdrojů) a syntézu (tvoje propojení)
- Zaměř se na praktické aplikace
- Preferuj čerstvé zdroje
- Report musí být použitelný jako znalostní báze, konkrétní, ne vágní
