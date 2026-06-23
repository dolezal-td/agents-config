---
name: write-a-prd
description: Vytvoření Product Requirements Document přes interview, průzkum codebase a návrh modulů, výstup jako GitHub Issue. Použij když chce uživatel napsat PRD, naplánovat novou feature, nebo zmíní "PRD" / "requirements".
---

# Write a PRD

Vytvoř PRD přes strukturovaný rozhovor. Výstup = GitHub Issue. Kroky můžeš přeskočit pokud nejsou potřeba.

## Process

### 1. Popiš problém

Požádej uživatele o detailní popis problému a nápadů na řešení. Pokud popis chybí, zeptej se: "Jaký problém řešíme a jaké máš nápady?"

### 2. Prozkoumej repo

Pokud existuje kódový repo, prozkoumej ho a ověř tvrzení uživatele. Pochop aktuální stav.

### 3. Interview (grill-me)

Interview the user relentlessly about every aspect of this plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

If a question can be answered by exploring the codebase, explore the codebase instead.

### 4. Navrhni moduly

Sketch out the major modules you will need to build or modify. Actively look for opportunities to extract **deep modules**, modules that encapsulate a lot of functionality in a simple, testable interface which rarely changes.

Check with the user that these modules match their expectations. Check which modules they want tests written for.

### 5. Napiš PRD a vytvoř GitHub Issue

Once you have a complete understanding, use the template below. Create via `gh issue create`.

Pokud projekt vede issue v externím nástroji pro správu úkolů, přidej do něj odkaz na GitHub Issue.

<prd-template>

## Problem Statement

The problem from the user's perspective.

## Solution

The solution from the user's perspective.

## User Stories

A LONG, numbered list of user stories:

1. As an <actor>, I want a <feature>, so that <benefit>

This list should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

- Modules to build/modify
- Interface changes
- Architectural decisions
- Schema changes
- API contracts

Do NOT include specific file paths or code snippets, they go out of date fast.

## Testing Decisions

- What makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art (similar tests in the codebase)

## Out of Scope

What is explicitly NOT part of this PRD.

## Further Notes

Any additional context.

</prd-template>

## Rules

- PRD = **destinace** (co chceme), ne **cesta** (jak se tam dostaneme). Cestu řeší `/prd-to-issues`.
- Žádné file paths ani code snippety, rychle zastarají.
- User stories popisují chování, ne implementaci.
- Po vytvoření issue vypiš URL a stručné shrnutí.
