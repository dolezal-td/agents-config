# agents-config: kontext pro pokračujícího agenta

Tohle repo je **dárek za absolvované školení od AI pro smrtelníky**: doporučená výchozí konfigurace Claude Code (časem i dalších agentů) pro úplné začátečníky a ne-vývojáře. Cíl: člověk si zkopíruje pár souborů do `~/.claude/` a má rovnou bezpečný, použitelný setup.

Pokud tohle čteš jako agent v nové session, který má pokračovat v práci na repu, tady je všechno potřebné.

## Tvar repa (zrcadlí `~/.claude`)

- `CLAUDE.md`: globální instrukce pro Clauda (jak mluvit, jak pracovat). Generická, nic osobního.
- `settings.json`: bezpečná výchozí pravidla (permissions + `defaultMode`).
- `README.md`: lidský návod, jak si to nalít.
- *(plánováno)* `skills/`: kurátorská hrstka obecných skills.

## Zamčená rozhodnutí

1. **Cílovka:** úplní začátečníci, ne-vývojáři (absolventi školení). Smooth > strict.
2. **Public repo = nic osobního.** Ven nesmí klienti, osobní voice/identita, byznys logika, MCP s osobními účty, osobní cesty a složky, hooky na lokální skripty. Všechno se píše rovnou obecně.
3. **Permissions model** (`deny → ask → allow`, první shoda vyhrává; `deny` drží i v auto/bypass módu jako jediná pojistka):
   - `allow`: čtení (`Read`/`Glob`/`Grep`) + web + read-only git.
   - `ask`: osobní složky (Desktop/Documents/Downloads), `.env`, `git push`.
   - `deny`: tajnosti (`~/.ssh`, klíče, `~/.aws/credentials`, `secrets/**`, `~/.npmrc`), destrukce (`sudo`, `rm -rf`, force push, `reset --hard`), zápis do shell/git rc (`~/.zshrc` aj.).
   - `defaultMode: acceptEdits` (plynulé úpravy souborů).
   - **Cesty kotvit `~/` (domov) nebo bare/gitignore vzorem, NIKDY `./`** (to platí jen z aktuální složky, takže deny pravidlo s `./` je děravé).
   - **`Bash(...)` pravidla = prefix forma s dvojtečkou:** `Bash(rm -rf:*)`, ne `Bash(rm -rf *)`. Glob s mezerou nematchuje spolehlivě (ověřeno v docs). `defaultMode` patří DOVNITŘ objektu `permissions`, ne na top-level.
4. **Bezpečnost:** `deny` pravidla jsou baseline, ne neprůstřelná hradba. `cat`/`sed` čtení blokují, ale vlastní skript (`python -c "open(...)"`) je obejde. Na to je až sandbox (OS-level). Pro dárek baseline stačí, je to přiznáno v README.
5. **Struktura `skills/` = jedna úroveň.** Žádné dělení na „jádro vs power pack". Všechny skilly v jednom adresáři, README odliší co je pro koho. (Rozhodnuto u #5.)

## Stav

**v1 LIVE:** `CLAUDE.md` + `settings.json` + `README.md`, public. Bezpečnostní a permissions vrstva hotová.

**Jádro doladěno (#5):** opraven tvar Bash deny patternů (`:*`), přidán `rm -fr`, do `CLAUDE.md` přidána zásada „nepřitakávej", README vysvětluje, že `CLAUDE.md` je šablona k přepsání. Struktura zamčena na jednu úroveň.

**První skilly (#4):** `skills/contract-review` (kopie, čistá, soběstačná) a `skills/youtube-research` (přepsán na soběstačný: yt-dlp místo lokálního skriptu, Tavily volitelný, výstup do `./` místo vaultu). `notebooklm` se NEkopíruje, jen návod v README na `pip install notebooklm-py` (cizí balík, který se sám aktualizuje). README má sekci „Skilly (volitelné)".

**session-log + end-of-day (#2):** přepsány od základu na generické. Default log `~/.claude/logs/YYYY-MM-DD.md` (mimo projekty, funguje pro oba). session-log rekapituluje z aktuální konverzace; end-of-day čte nativní CC transkripty `~/.claude/projects/<projekt>/*.jsonl` napříč projekty + git (cwd z transkriptů). Žádný SpecStory/vault/Gmail/Calendar/Todoist/wiki/STATUS/klas tagy/wikilinky. jq jednořádky ověřené na reálném `.jsonl` (klíče type/timestamp/cwd/message.content; filtr na string content vynechá tool noise).

**dev-balík (#1):** 11 skillů vyčištěno (paralelně přes 4 subagenty) a přidáno do `skills/`: vyvoj, research, deep-research, grill-me (+ CONTEXT-FORMAT, ADR-FORMAT), write-a-prd, prd-to-issues, testing-strategy, mock-strategy, test-anti-patterns, git-workflow, start-development. Vystřiženo: jména/klienti, vault cesty, Vercel deploy, Notion DB ID, lokální research-lib skripty (research/deep-research přepsány na soběstačné), odkaz na frontend-design (zobecněn). Cross-odkazy na superpowers skilly označeny „(vyžaduje plugin superpowers)". README má podsekci „Vývojové skilly" + závislosti: superpowers (`/plugin install superpowers@claude-plugins-official`), gh CLI, Context7 MCP (volitelné), Tavily (volitelné).

**last30days (#3):** NEextrahováno do vlastního repa. Zjištěno, že originál existuje veřejně ([mvanhorn/last30days-skill](https://github.com/mvanhorn/last30days-skill), 11.9k stars, MIT, marketplace). Stejný pattern jako notebooklm: README jen odkazuje na originál (`/plugin marketplace add mvanhorn/last30days-skill` + `/plugin install last30days`). Tvorba vlastního repa zamítnuta Tomášem.

## Co dál

1. **Skills:** doplnit `skills/` o malou sadu **obecných, neosobních** skills (typu research, psaní promptů, práce s gitem, tvorba PRD apod.). **Každý skill před zveřejněním projít a vystřihnout osobní reference** (voice, klienti, cesty do vaultu, konkrétní účty). Zdroj kandidátů: `~/.claude/skills/` na stroji autora. Instalace u uživatele = kopie do `~/.claude/skills/`. Hotovo z #4: contract-review, youtube-research, notebooklm. Hotovo z #2: session-log, end-of-day (generický rework). Hotovo z #1: dev-balík (11 skillů). Hotovo z #3: last30days odkázán na veřejný originál (neextrahováno). Všechny skill issues uzavřené.
2. **Pluginy:** doporučit pár public pluginů. Oficiální (`claude-plugins-official`) fungují rovnou přes `enabledPlugins`; non-oficiální marketplaces je nutné nejdřív přidat (`claude plugin marketplace add …`), takže to buď zdokumentovat v README, nebo nechat na pozdější vedený onboarding.
3. ~~Doladit `CLAUDE.md` a `settings.json`~~ → **hotovo (#5).**
4. Zvážit variantu `AGENTS.md` jako orientaci i pro Codex (stejný obsah, jiný formát konzumace).

## Vazby

Velký bratr je privátní projekt s vedeným onboardingovým agentem (na dlouho, teď pozastavený). Tenhle repo je rychlá, ruční varianta téhož: „tady máš config, nalej si ho".
