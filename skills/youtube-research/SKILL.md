---
name: youtube-research
description: YouTube research, najde relevantní videa k tématu, stáhne přepisy a vytěží znalosti. Použij když potřebuješ zjistit co se říká na YouTube o nějakém tématu, nástroji nebo trendu.
allowed-tools: Bash, WebSearch, WebFetch
---

# YouTube Research

Proveď YouTube research na téma: $ARGUMENTS

> **Závislost:** tento skill stahuje přepisy přes `yt-dlp`. Pokud ho nemáš, nainstaluj:
> `brew install yt-dlp` (macOS) nebo `pip install yt-dlp`.

## Fáze 1: Discovery

### 1a. Vyhledej videa

Spusť paralelně přes WebSearch:
- "$ARGUMENTS site:youtube.com 2026"
- "$ARGUMENTS youtube tutorial review 2026"

> **Volitelné:** Pokud máš nastavený Tavily MCP (`mcp__tavily__tavily_search`), použij ho navíc s `include_domains: ["youtube.com"]`, `max_results: 10`. Není to podmínka, WebSearch stačí.

### 1b. Vyber top 5 videí

Z nalezených videí vyber TOP 5 podle:
1. Relevance k tématu
2. Datum publikace (preferuj novější)
3. Pokud znáš views/engagement, preferuj vyšší

## Fáze 2: Stažení přepisů

Pro KAŽDÉ z top 5 videí stáhni přepis (titulky) přes `yt-dlp`, bez stahování videa:

```bash
yt-dlp --write-auto-subs --write-subs --sub-langs "cs,en" \
  --skip-download --convert-subs srt -o "%(id)s.%(ext)s" "VIDEO_URL"
```

Vznikne soubor `<id>.cs.srt` nebo `<id>.en.srt` (podle dostupných titulků) = přepis videa. Přečti ho.

Pokud `yt-dlp` žádný soubor nevytvoří (video nemá titulky), přeskoč a poznamenej.

## Fáze 3: Knowledge Extraction

Z každého přepisu extrahuj:

### Pro KAŽDÉ video:
- **Video:** [title]
- **URL:** https://youtube.com/watch?v=...
- **Klíčové body:** 3-5 hlavních myšlenek z přepisu
- **Citace:** doslovné citáty, které jsou hodnotné (max 3)
- **Unikátní insight:** co říká toto video, co ostatní ne

### ANTI-HALUCINAČNÍ PRAVIDLA:
1. POUZE informace z přepisů, NIKDY nevymýšlej obsah videa
2. Každý fakt musí odkazovat na konkrétní video [video: URL]
3. Pokud přepis neobsahuje relevantní info → "Video nepokrývá toto téma"

## Fáze 4: Syntéza

### Formát výstupu:

```markdown
# YouTube Research: {téma}
> Datum: YYYY-MM-DD | Videa: N | Přepisy: M

## Executive Summary
[Co se na YouTube říká o tomto tématu, shrnutí napříč videi]

## Videa

### 1. {Název videa}
**URL:** [link] | **Datum:** YYYY-MM-DD

**Klíčové body:**
- ... [video: URL]
- ...

**Citace:**
> "doslovný citát z přepisu" [video: URL]

**Unikátní insight:**
- ...

### 2. {Další video}
...

## Cross-video analýza
[Co říká většina videí shodně, kde se liší, jaké trendy]

## Zdroje
1. [Video title](URL), date
```

## Krok 5: Ulož výstup

Ulož report do aktuální složky: `./youtube-research-{slug}-{date}.md`

Kde `{slug}` je téma převedené na kebab-case (max 40 znaků) a `{date}` je YYYY-MM-DD.

## Pravidla

- Celý výstup česky
- Cituj konkrétní video u každého faktu
- Transkripty jsou primární zdroj, ne popisky z YouTube
- Report musí být použitelný jako znalostní báze
