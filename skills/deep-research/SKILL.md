---
name: deep-research
description: Hloubkový research tématu (15-20 min) s paralelním search, verifikací faktů a strukturovaným výstupem. Použij pro brutální rešerši, kde záleží na přesnosti a aktuálnosti dat.
allowed-tools: WebSearch, WebFetch, Agent, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Deep Research

Proveď hloubkový research na téma: $ARGUMENTS

Tento skill trvá 15-20 minut. Pracuje ve 4 fázích:
1. **Data Collection**: paralelní web search + Context7
2. **JSON Extraction**: strukturovaná extrakce claimů z nalezených dat
3. **Verification**: WebFetch ověření klíčových faktů
4. **Synthesis**: finální report s dvouvrstvým výstupem

---

## Fáze 1: Data Collection (paralelně)

### 1a. Rozlož téma a prohledej web

Rozlož téma na zhruba 8 search queries pokrývajících overview, funkce, ceny, alternativy, novinky a limitace. Spusť je paralelně přes WebSearch.

- Preferuj zdroje z posledních zhruba 30 dní (date-gating).
- Pokud máš k dispozici search MCP (např. Tavily), použij ho pro hlubší pokrytí, freshness skóre a cílené hledání na konkrétních doménách přes `include_domains`. Není podmínkou.
- Výsledky deduplikuj a seřaď podle relevance a data.

### 1b. Cílené WebSearch dotazy (spusť SOUČASNĚ)

Spusť 3-5 doplňkových WebSearch dotazů zaměřených na:
- **Oficiální stránky**: "{téma} official site pricing"
- **Srovnání**: "{téma} vs {hlavní alternativa} comparison"
- **Novinky**: "{téma} latest update release"
- **Reviews**: "{téma} review honest opinion"
- **Limitace**: "{téma} limitations downsides problems"

### 1c. Context7 (pokud je téma technické)

Pokud se téma týká knihovny, frameworku nebo API:
1. `resolve-library-id`: najdi knihovnu
2. `query-docs`: stáhni aktuální dokumentaci (zaměř se na getting started, pricing, API reference)

### 1d. Social Sources (POVINNÉ)

Spusť paralelně s ostatními searches:

**Reddit:**
- WebSearch: "{téma} site:reddit.com discussion opinions"
- WebSearch: "{téma} reddit best recommendations"
- Pokud search výsledky obsahují Reddit URLs, prioritizuj je ve výstupu

**X/Twitter:**
- WebSearch: "{téma} site:x.com OR site:twitter.com thread"

**YouTube (top 3 videa):**
1. Najdi relevantní videa přes WebSearch (`site:youtube.com`) nebo search MCP s `include_domains: ["youtube.com"]`
2. Vyber top 3 podle relevance a data
3. Pro stažení a vytěžení přepisů použij skill `youtube-research`
4. Z přepisů extrahuj klíčové body a citace, zahrň do syntézy jako plnohodnotný zdroj
5. Ve výstupním reportu uveď YouTube videa v sekci "Z kontextu zdrojů" s odkazem na video

**Proč social sources:** Reddit, YouTube a X poskytují komunitní perspektivu, reálné zkušenosti, problémy a doporučení, které oficiální dokumentace neobsahuje.

---

## Fáze 2: JSON Extraction

Ze VŠECH nalezených dat (web search + Context7) extrahuj strukturované informace.

### Pro KAŽDÝ produkt/nástroj/koncept vytvoř JSON:

```json
{
    "name": "Název produktu",
    "category": "Kategorie (AI video, AI photo, ...)",
    "url": "https://official-site.com",
    "claims": [
        {
            "fact": "Konkrétní, měřitelný fakt",
            "source_url": "https://zdroj-odkud-fakt-pochazi",
            "confidence": "verified",
            "date_source": "2026-03-15"
        }
    ]
}
```

### Confidence levels:
- **verified**: fakt přímo z oficiální stránky nebo produktové dokumentace
- **high**: z důvěryhodného review/článku s odkazem na zdroj
- **medium**: z komunitní diskuse nebo sekundárního zdroje
- **low**: zmíněno v jednom zdroji bez potvrzení

### KRITICKÉ PRAVIDLO:
**Claim BEZ `source_url` = NEVALIDNÍ.** Pokud nemáš URL zdroje, nepiš claim, napiš "Nezjištěno z aktuálních zdrojů".

---

## Fáze 3: Verification

Pro TOP 5-10 nejdůležitějších claimů (ceny, klíčové funkce, limity) proveď verifikaci:

1. **WebFetch** na `source_url`: stáhni aktuální obsah stránky
2. **Porovnej** claim s aktuálním obsahem
3. **Ohodnoť:**
   - ✅ **Confirmed**: claim odpovídá aktuálnímu obsahu
   - ⚠️ **Differs**: claim se liší od aktuálního obsahu (uveď obě verze)
   - ❌ **Not found**: claim se nepodařilo ověřit (stránka neobsahuje info)

### Priorita verifikace:
1. Ceny a plány (nejvyšší priorita, mění se nejčastěji)
2. Názvy modelů a verzí
3. Klíčové limity (maximální délka videa, rozlišení, atd.)
4. Dostupnost (free tier, waitlist, regiony)

### Pokud WebFetch vrátí chybu:
- 403/429: Zkus alternativní URL z jiného zdroje
- 404: Označ claim jako ❌ Not found
- Timeout: Přeskoč a poznamenej

---

## Fáze 4: Synthesis

### ANTI-HALUCINAČNÍ PRAVIDLA (ABSOLUTNĚ POVINNÁ)

```
KRITICKÉ INSTRUKCE PRO SYNTÉZU:

1. Používej VÝHRADNĚ data z nalezených a ověřených zdrojů.
2. NIKDY nedoplňuj informace z vlastních tréninkových dat. Jsou ZASTARALÁ.
3. Každý fakt MUSÍ mít [zdroj: URL] nebo [ověřeno: URL, datum].
4. Pokud informace NENÍ v nalezených zdrojích → "Nezjištěno z aktuálních zdrojů"
5. Pokud se zdroje LIŠÍ → uveď OBĚ verze s odkazy.
6. NIKDY nepoužívej fráze: "podle mých znalostí", "pokud vím", "obvykle".
7. Názvy, verze, ceny piš PŘESNĚ jak je uvádí zdroje.
```

### Dvouvrstvý výstup

Pro KAŽDÝ produkt/nástroj rozlišuj:

**Ověřená data** (claim má source_url + prošel verifikací):
- Fakt ✅ [ověřeno: URL, datum]

**Z kontextu zdrojů** (cenné informace z nalezených zdrojů, ale neověřeno WebFetchem):
- Insight [zdroj: URL]

### Formát výstupního reportu

```markdown
# Deep Research: {téma}
> Datum: YYYY-MM-DD | Mód: deep | Zdroje: N stránek | Ověřeno: M faktů

## Executive Summary
[3-5 vět, hlavní zjištění, POUZE z nalezených zdrojů]

## Detailní přehled

### {Produkt/Nástroj 1}
**Oficiální web:** [URL]

**Ověřená data:**
- Cena: ... ✅ [ověřeno: URL, 2026-03-18]
- Klíčové funkce: ... ✅ [ověřeno: URL, 2026-03-18]
- Limity: ... ✅

**Z kontextu zdrojů:**
- Kvalita: ... [zdroj: URL]
- Srovnání: ... [zdroj: URL]
- Insighty z komunity: ... [zdroj: URL]

### {Produkt/Nástroj 2}
...

## Srovnávací tabulka
| Nástroj | Cena | Klíčová funkce | Free tier | Ideální pro | Zdroj |
|---|---|---|---|---|---|

## Trendy a závěry
[Syntéza POUZE z nalezených zdrojů, co se mění, kam směřuje trh]

## Verifikační log
| Claim | Zdroj | Status | Poznámka |
|---|---|---|---|
| Cena X je $Y/měs | URL | ✅ Confirmed | |
| Model Z má 4K | URL | ⚠️ Differs | Aktuálně 2K |

## Zdroje
1. [URL]: anotace (datum, freshness tag)
2. ...

## Metadata
- Queries: [seznam všech queries]
- Verified claims: X/Y
- Unverified insights: [počet]
- Stránky fetchnuté pro verifikaci: [počet]
```

---

## Krok 5: Ulož výstup

Ulož report do pracovní složky projektu (např. `working/deep-research-{slug}-{date}.md`).

Kde `{slug}` je téma převedené na kebab-case (max 40 znaků) a `{date}` je YYYY-MM-DD.

---

## Pravidla

- Celý výstup česky (pokud uživatel neřekne jinak)
- Cituj zdroje u KAŽDÉHO faktu, žádný fakt bez URL
- Dvouvrstvý výstup: ověřená data vs kontext ze zdrojů
- Verifikuj alespoň 5 nejdůležitějších claimů přes WebFetch
- Zaměř se na praktické aplikace, ceny, limity, použitelnost
- Report musí být použitelný jako znalostní báze, konkrétní, ne vágní
- Preferuj FRESH zdroje (< 7 dní) před OK (< 30 dní) před OLD
