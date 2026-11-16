# alp-kartotek-line

Et ALP-linjerepo: **Kartotek — høst LinkedIn-beskeder.**

Linjen åbner hver samtale i LinkedIns indbakke, samler tråden og dens beskeder,
og afleverer resultatet videre til berig-linjen. Den kører på en
**browser-tray** på en rigtig maskine med en rigtig indlogget profil — der er
ingen LLM i kæden, og ingen legitimation forlader maskinen.

Repoet er **offentligt af nødvendighed**: importøren på agentics.dk henter
`.agentics/`-filerne fra `raw.githubusercontent.com` og lister stationerne
gennem GitHubs contents-API **uden token**. Et privat repo kan ikke importeres.
Der ligger derfor ingen hemmeligheder her, og der må aldrig komme nogen.

## Indhold

```
.agentics/assembly-line.json      navn og beskrivelse
.agentics/transitions.json        host → hostet (success), host → fejlet (failure)
.agentics/stations/01-host/       HØST    — dispatch_worker + trigger.recipe
.agentics/stations/02-hostet/     HØSTET
.agentics/stations/03-fejlet/     FEJLET
```

**Mappenavnene bestemmer rækkefølgen.** Importøren sorterer stationsmapperne
alfabetisk og sætter `order` efter den sortering, og den dispatchende kolonne er
den med lavest `order`. Derfor talpræfikserne: uden dem ville `fejlet` sortere
først og blive den kolonne der starter linjen. Station-`id`'et kommer fra
`station.json`, ikke fra mappenavnet, så transitionerne peger på `host`,
`hostet` og `fejlet`.

## Hvorfor `goto` og ikke `click`

Trinnet der åbner en samtale er bevidst en **navigation**, ikke et klik:

```json
{ "id": "aabn-traad", "do": "goto", "url": "{{t.href}}" }
```

Trayens tredje spærre kræver et `human`-trin før det første *handlende* trin, og
`click` tælles som handlende — med god grund, for et klik kan ramme "Send". En
høst der kun læser må til gengæld køre uovervåget. Da samtalerne i listen er
links, og hver indsamlet række i forvejen bærer en absolut `href`, kan tråden
åbnes med `goto`, som er et læse-verbum. Hele høsten er derfor rene læse-verber.

Første udgave klikkede, og den blev afvist af trayen hver gang den kørte — tre
gange på en time den 16-11-2026, hver gang med *"'click' handler, og der står
intet 'human'-trin før det"*. Rettelsen er det ene trin ovenfor.

**Bemærk:** `{{t.href}}` holdes ikke mod profilens domæne-allowlist. Trayens
allowlist gælder adresser der står ordret i opskriften, og `goto`s egen
`assertReachable` er en egress-spærre mod lokale adresser, ikke en
domæneliste. Adressen her kommer fra LinkedIns egen samtaleliste; vil man have
den bundet, hører håndhævelsen i motorens `goto`.

## Import

Importér fra `/import` på agentics.dk eller med MCP-værktøjet `line.import`, med
dette repo og grenen `main`. Importen opretter en **ny** linje — den opdaterer
ikke en eksisterende — så en tidsplan der peger på den gamle linje skal pege om
bagefter.

`publish.handoff.assemblyLineId` i stationens opskrift peger på berig-linjen i
det miljø linjen blev høstet i. Det er miljøspecifikt: importerer du linjen et
andet sted, skal det id rettes til den tilsvarende berig-linje.
