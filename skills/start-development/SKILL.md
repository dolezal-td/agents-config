---
name: start-development
description: Inicializace vývojového projektu. Použij když začínáš nový vývojový projekt a potřebuješ vytvořit PLAN.md a CLAUDE.md.
---

# Start Development

Konverzační inicializace vývojového projektu. Pochop proč projekt existuje, pro koho je a jaký zážitek má přinést, a z toho odvoď architekturu a plán.

Výstup: PLAN.md + CLAUDE.md.

---

## Fáze 1: Pochop záměr

Ptej se **jednu otázku po druhé**. Preferuj multiple choice kde to dává smysl. Jedna otázka na zprávu. Pokud uživatel odpoví na víc věcí najednou, přejdi dál. Jakmile máš dost, řekni "Mám dost, navrhnu řešení" a přejdi na fázi 2.

### Blok A: Proč a pro koho

1. **Co buduješ a proč?**: Jaký problém to řeší? Co se stane, když to nevznikne?
2. **Kdo je uživatel?**: Kdo s tím bude pracovat? (koncový zákazník, interní tým, ty sám, kombinace)
3. **Jak to bude uživatel používat?**: Popiš ideální interakci. Co uživatel udělá, co uvidí, co si odnese.
4. **Co je úspěch?**: Jak poznáš, že to funguje? (metrika, pocit, zpětná vazba)

### Blok B: Scope a omezení

5. **Co musí umět MVP?**: Minimum pro první hodnotu. Co je nice-to-have až později?
6. **Existující materiály**: Máš data, dokumenty, wireframy, kód, inspiraci?
7. **Kdo na tom pracuje?**: Jen ty + agenti, nebo i další lidé?

### Blok C: Technické preference (ptej se stručně, NEŘEŠ dopodrobna)

8. **Máš technické preference nebo omezení?**: Jazyk, framework, hosting, cokoliv. Pokud ne, navrhneš sám.

### Pravidla

- Bloky A a B jsou povinné. Blok C je volitelný, pokud uživatel nemá preference, navrhni sám na základě A+B.
- Technický stack je **důsledek** uživatelského zážitku, ne výchozí bod.
- Nemusel jsi položit všechny otázky, stačí když rozumíš záměru.

---

## Fáze 2: Navrhni řešení

Na základě odpovědí navrhni v tomto pořadí:

1. **Uživatelský zážitek**: Jak to bude vypadat a fungovat z pohledu uživatele (stránky, flow, interakce)
2. **Architektura**: Strom složek s komentáři. Odvoď z UX, co potřebuješ aby ten zážitek fungoval.
3. **Tech stack**: Tabulka (vrstva | technologie | proč právě toto). Odvoď z architektury.
   **Context7 POVINNÝ:** Pro každou klíčovou knihovnu/framework ověř aktuální dokumentaci přes Context7 (`resolve-library-id` → `query-docs`). Nepoužívej jen knowledge cutoff, Context7 má živé docs.
4. **Fáze implementace**: 2-4 fáze, od MVP po vylepšení
5. **Workflow**: Jak vypadá finální provoz (ne vývoj, ale reálné použití)

Prezentuj jako shrnutí (200-300 slov). Na konci:
> "Vypadá to dobře? Chceš něco změnit?"

Opakuj dokud uživatel neschválí.

---

## Fáze 3: Generuj soubory

Po schválení vygeneruj dva soubory.

### PLAN.md

Umístění: tam kde je hlavní kód (root, nebo podsložka pokud multi-repo).

```markdown
# [Název]: Projektový plán

> [1 věta, co a proč]
> Tento soubor je single source of truth pro plánování a tracking projektu.

**Stav:** Fáze 1: [název]
**Založeno:** [datum]
**Poslední aktualizace:** [datum]

---

## Proč to děláme
[Problém, řešení, cílový stav. 2-3 odstavce z konverzace.]

---

## Pro koho to je
[Uživatelé, jejich potřeby, jak s tím interagují.]

---

## Uživatelský zážitek
[Stránky/obrazovky, flow, klíčové interakce. Jak to VYPADÁ a FUNGUJE.]

---

## Architektura
[Strom složek s komentáři. Odvozeno z UX.]

---

## Tech stack
| Vrstva | Technologie | Proč |
[odvozeno z architektury a UX požadavků]

---

## Fáze implementace

### Fáze 1: [název] (aktuální)
- [ ] **1.1** [krok]
...

### Fáze 2: [název] (budoucí)
...

---

## Workflow (provoz)
[Jak to funguje v praxi, od vstupu po výstup. Reálné použití, ne vývoj.]

---

## Verifikace
[Jak ověřit hotový krok, konkrétní příkazy.]

---

## Rozhodnutí a změny
| Datum | Rozhodnutí |
[klíčová rozhodnutí z konverzace]
```

### CLAUDE.md

Umístění: **root projektu** (auto-load). Max 150 řádků.

```markdown
# [Název]: Projektový kontext

## Co je tento projekt
[1-3 věty: co, proč, pro koho]

---

## Architektura
[Stručný strom složek]
Detail: viz **[cesta/PLAN.md]**.

---

## Aktuální stav
**Fáze:** 1: [název]
**Krok:** 1.0: Plánování (hotovo)
**Next step:** 1.1: [první krok]
**Úkoly:** [kde se trackují úkoly tohoto projektu, např. GitHub Issues, Notion, Todoist]

---

## Mapa souborů: kdy co číst
| Soubor | Účel | Kdy číst |
|---|---|---|
| **CLAUDE.md** | Kontext, stav, instrukce | Vždy (auto-loaded) |
| **[PLAN.md]** | Architektura, UX, rozhodnutí | Detail o směřování |
[+ klíčové docs]

---

## Konvence
- Komunikace, commity, docs: čeština. Kód: anglické identifikátory.
- [Per technologie: 1 řádek]
- Git: feature branches, PR do main
- **Context7:** Při implementaci jakékoliv knihovny nebo frameworku vždy nejdřív `resolve-library-id` + `query-docs`, ne knowledge cutoff

---

## Paralelizace a subagenti
[Nezávislé moduly z architektury]
- Soubory měnící subagenti → `isolation: "worktree"`

---

## Workflow (cílový stav)
[Číslovaný seznam: reálný provoz]

---

## Práce se session

### Na začátku
1. Tento soubor (auto-loaded)
2. PLAN.md pro detail

### Během práce
- Úkoly z trackeru, paralelizuj kde lze
- Blocker → zapiš, přejdi dál

### Na konci session
1. Commitni změny
2. Aktualizuj Aktuální stav zde
3. Nabídni /session-log

---

## Verifikace
[Konkrétní příkazy]
```

---

## Fáze 4: Potvrď a zapiš

1. Ukaž shrnutí obou souborů (strukturu, ne celý obsah)
2. "Všechno ok? Zapisuji."
3. Zapiš soubory
4. Ověř: cesty existují, CLAUDE.md pod 150 řádků
5. "Infrastruktura připravena. Úkoly založ ve svém trackeru úkolů."
