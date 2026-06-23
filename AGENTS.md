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
4. **Bezpečnost:** `deny` pravidla jsou baseline, ne neprůstřelná hradba. `cat`/`sed` čtení blokují, ale vlastní skript (`python -c "open(...)"`) je obejde. Na to je až sandbox (OS-level). Pro dárek baseline stačí, je to přiznáno v README.

## Stav

**v1 LIVE:** `CLAUDE.md` + `settings.json` + `README.md`, public. Bezpečnostní a permissions vrstva hotová.

## Co dál

1. **Skills:** doplnit `skills/` o malou sadu **obecných, neosobních** skills (typu research, psaní promptů, práce s gitem, tvorba PRD apod.). **Každý skill před zveřejněním projít a vystřihnout osobní reference** (voice, klienti, cesty do vaultu, konkrétní účty). Zdroj kandidátů: `~/.claude/skills/` na stroji autora. Instalace u uživatele = kopie do `~/.claude/skills/`.
2. **Pluginy:** doporučit pár public pluginů. Oficiální (`claude-plugins-official`) fungují rovnou přes `enabledPlugins`; non-oficiální marketplaces je nutné nejdřív přidat (`claude plugin marketplace add …`), takže to buď zdokumentovat v README, nebo nechat na pozdější vedený onboarding.
3. **Doladit `CLAUDE.md` a `settings.json`** do skvělého stavu (jsou to dvě nejdůležitější věci v repu).
4. Zvážit variantu `AGENTS.md` jako orientaci i pro Codex (stejný obsah, jiný formát konzumace).

## Vazby

Velký bratr je privátní projekt s vedeným onboardingovým agentem (na dlouho, teď pozastavený). Tenhle repo je rychlá, ruční varianta téhož: „tady máš config, nalej si ho".
