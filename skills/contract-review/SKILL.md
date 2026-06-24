---
name: contract-review
description: Hloubková kontrola smluv z pozice protistrany. Kontroluje správnost, vnitřní konzistenci, soulad s legislativou a nevýhodná ustanovení. Použij když uživatel předloží smlouvu (PDF, DOCX, text) ke kontrole, revizi nebo analýze rizik. Triggery: zkontroluj smlouvu, review smlouvy, projdi smlouvu, analýza smlouvy, co říkáš na tuhle smlouvu, je ta smlouva ok.
---

# Kontrola smlouvy

## Workflow

1. **Načtení smlouvy** – přečíst celý dokument (PDF skill pro .pdf, DOCX skill pro .docx, nebo přímo text)
2. **Klasifikace typu** – určit typ smlouvy (SoD, smlouva o spolupráci, NDA, rámcová, nájemní, licenční, jiná)
3. **Systematická kontrola** – projít checklist v [references/checklist.md](references/checklist.md) bod po bodu
4. **Křížová kontrola konzistence** – ověřit, že si ustanovení navzájem neodporují
5. **Legislativní kontrola** – ověřit soulad s relevantní legislativou (zejména NOZ, ZP, GDPR)
6. **Analýza nevýhodnosti** – identifikovat ustanovení znevýhodňující uživatele jako protistranu
7. **Výstupní report** – strukturovaný markdown report

## Pravidla analýzy

- Analyzovat z pozice **protistrany** (uživatel = ten, kdo smlouvu obdržel ke kontrole)
- Každý nález ukotvit v **konkrétním článku/odstavci** smlouvy (citovat číslo + relevantní text)
- Rozlišovat závažnost: **kritické** (blokující podpis), **důležité** (vyžaduje úpravu), **doporučení** (nice-to-have)
- U každého nálezu navrhnout **konkrétní formulaci opravy** nebo alternativní znění
- Nehodnotit jen text, ale i **co ve smlouvě chybí** (viz checklist – chybějící ustanovení)
- Při legislativní kontrole uvádět **konkrétní paragraf** relevantního zákona

## Výstupní formát

```markdown
# Kontrola smlouvy: [název/typ smlouvy]

**Typ smlouvy:** [klasifikace]
**Strany:** [strana A] vs. [strana B – uživatel]
**Datum smlouvy:** [datum, pokud uvedeno]
**Kontrolováno:** [dnešní datum]

---

## Shrnutí

[2–3 věty: celkový dojem, hlavní rizika, doporučení zda podepsat / podepsat s úpravami / nepodepisovat]

## Klíčové parametry

| Parametr | Hodnota | Poznámka |
|---|---|---|
| Cena/odměna | ... | ... |
| Doba trvání | ... | ... |
| Výpovědní lhůta | ... | ... |
| Smluvní pokuty | ... | ... |
| Odpovědnost za škodu | ... | ... |
| Rozhodné právo | ... | ... |

## Kritické nálezy

### [K1] [Název nálezu]
- **Článek:** [číslo článku, citace]
- **Problém:** [popis]
- **Riziko:** [konkrétní dopad na uživatele]
- **Návrh úpravy:** [konkrétní alternativní formulace]

## Důležité nálezy

### [D1] [Název nálezu]
- **Článek:** [číslo článku, citace]
- **Problém:** [popis]
- **Návrh úpravy:** [formulace]

## Doporučení

### [R1] [Název doporučení]
- **Článek:** [číslo článku]
- **Poznámka:** [co zlepšit a proč]

## Chybějící ustanovení

[Seznam důležitých ustanovení, která ve smlouvě chybí a měla by být doplněna, s návrhem znění]

## Vnitřní konzistence

[Případné rozpory mezi jednotlivými články smlouvy]

## Legislativní soulad

[Ustanovení v rozporu s platnou legislativou, s odkazem na konkrétní zákon/paragraf]

## Závěr a doporučení

[Celkové hodnocení, prioritizovaný seznam kroků před podpisem]
```

## Poznámky k typům smluv

Při kontrole zohlednit specifika daného typu:

- **Smlouva o dílo (§ 2586+ NOZ):** zhotovitel vs. objednatel, převzetí díla, vady, záruka
- **Smlouva o spolupráci:** vzájemné povinnosti, dělení příjmů/nákladů, IP práva, exkluzivita
- **NDA (§ 1730 NOZ):** rozsah důvěrných informací, doba trvání, sankce za porušení, výjimky
- **Rámcová smlouva:** podmínky dílčích objednávek, cenotvorba, minimální odběr
- **Licenční smlouva (§ 2358+ NOZ):** rozsah licence, teritorium, sublicence, royalties
- **Nájemní smlouva (§ 2201+ NOZ):** účel nájmu, údržba, podnájem, inflační doložka
