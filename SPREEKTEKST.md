# Spreektekst – The Desync Protocol

Team OpenPrompting · HTF2026 · I8C Challenge
Totaal ongeveer 9 minuten. Tekst tussen *[haakjes]* is een aanwijzing en zeg je niet luidop.

---

## Slide 1 – The Desync Protocol

Goeiemiddag allemaal. Wij zijn team OpenPrompting en wij tonen jullie onze oplossing voor The Desync Protocol, de challenge van I8C. We bouwden het Beacon Network van Achrona: een systeem dat meldingen controleert en daarna de juiste mensen waarschuwt. Dat doen we in Workato, samen met Slack, een AI-agent en Elasticsearch met Kibana.

## Slide 2 – Eerst controleren, dan alarmeren

Eerst het verhaal. Achrona is een rijk met vier regio's: Riftmark, Thornvale, Emberhold en Glasswater. Als er iets misgaat, bijvoorbeeld een scheur in een toren of een rift die opent, dan waarschuwt het Beacon Network de bevolking.

Maar er is een probleem: die meldingen komen van buiten en kloppen niet altijd. Sommige komen dubbel binnen, of gaan over een regio die niet bestaat. Andere hebben een ongeldige ernst, of zijn al lang achterhaald. Ons principe is daarom: eerst controleren, dan pas alarmeren.

## Slide 3 – Vier recipes, vijf data tables

We hebben dat opgelost met vier recipes. R1 is de poort: die ontvangt events via een API en controleert ze. R2 beslist wie een alarm krijgt. R3 is een tweede ingang, waar verkenners in gewone taal kunnen melden via Slack. En R4 geeft elke vijf minuten een statusoverzicht.

Daaronder zitten vijf tabellen. Regions en Citizens waren al gevuld. Event Log, Dispatch Log en Failed Events vullen wij zelf. Zo houden we bij wat er binnenkomt, wat er verstuurd wordt en wat er misloopt.

## Slide 4 – Van melding tot alarm

*[Wacht even tot de pijlen verschijnen.]*

Zo hangt alles samen. Links zie je de twee ingangen: de API naar R1 en het Slack-kanaal van de verkenners naar R3. Beide leggen het event vast in de Event Log en roepen dezelfde functie aan: R2. Die verstuurt de alarmen naar Slack en noteert elke verzending in de Dispatch Log.

R4 leest de Event Log en Failed Events, en zet een overzicht in ons ops-kanaal. Onderaan zie je dat alle recipes ook naar Elasticsearch loggen, zodat we in Kibana kunnen zoeken.

## Slide 5 – Hoe ernstiger, hoe meer mensen

Dit is de kern van het systeem. Bij ernst één en twee krijgt alleen het regiokanaal een bericht. Vanaf drie spreken we ook de warden aan, de verantwoordelijke van de regio. Vanaf vier krijgen ook de ingeschreven burgers een bericht. En bij vijf waarschuwen we alle andere regio's en roepen we de raad bijeen.

Helemaal rechts staat all_clear. Dat sluit de openstaande alarmen van die regio en geeft geen nieuw alarm.

## Slide 6 – R1, Watchtower Gate

R1 is onze poort. Elk event eindigt in precies één van deze vijf uitkomsten.

We kijken eerst of het een duplicaat is, want dat is de goedkoopste controle en het voorkomt dubbel werk. Daarna controleren we of het event geldig is. Is het ouder dan 24 uur, dan leggen we het vast maar sturen we geen alarm. De afzender krijgt dan een 202 en geen foutcode, want het event is niet ongeldig, alleen achterhaald.

Pas daarna roepen we R2 aan, in een try/catch. Crasht er iets, dan proberen we het één keer opnieuw. Lukt het dan nog niet, dan blijft het event zichtbaar in Failed Events.

## Slide 7 – R2, Beacon Dispatch

R2 is een recipe function die R1 en R3 allebei gebruiken. Hij zoekt eerst de regio op, zodat het kanaal en de warden altijd uit de tabel komen en nooit vast staan. Een all_clear handelt hij apart af.

Voor alle andere events stuurt hij altijd het basisalarm. Afhankelijk van de ernst komen daar de warden, de burgers, en het hele rijk met de raad bij. Burgers kiezen trouwens zelf vanaf welke ernst ze gewaarschuwd willen worden. En aan het dispatch-ID zie je meteen over welk soort melding het gaat.

## Slide 8 – R3, Oracle Scout Intake

Bij R3 schrijft een verkenner gewoon een berichtje in Slack. Onze AI-agent, de Achrona Oracle, haalt daar de regio, de ernst, het type, een titel en een betrouwbaarheid uit.

Daarna komt de belangrijkste keuze. Is de ernst onder één, de betrouwbaarheid onder zestig procent, of de regio onbekend? Dan gaat het rapport naar een mens. Eén twijfel is genoeg. Zo veroorzaakt de AI niet blind alarmen.

We moesten ook oppassen voor een lus, omdat R2 in hetzelfde kanaal post waar R3 meeluistert. Daarom filteren we berichten van bots weg. En we vergelijken getallen als getallen, want als tekst liep dat fout.

## Slide 9 – R4, Chronicle Monitor

R4 is onze wachter. Elke vijf minuten doet hij drie zoekopdrachten: alle events van het laatste uur, alle mislukte events, en de ernstige meldingen die nog openstaan.

Rechts zie je een echt bericht uit onze demo, van 15 uur 13. Acht events, één fout en één open ernstige melding: iets groots in het kreupelhout. Die ene fout verstoppen we niet. Fouten moeten zichtbaar blijven.

## Slide 10 – Eén logstap in elke recipe

Elke recipe gebruikt dezelfde logstap naar Elasticsearch. Aan service.name zie je welke recipe het bericht schreef, en met team.name filteren we op ons eigen team.

We zijn hier ook eerlijk over wat nog openstaat. De organisatoren bevestigden dat ons veld timestamp mag heten. Of Kibana dat veld ook echt als tijdstempel gebruikt, moeten we nog controleren.

## Slide 11 – Demo 1: Thornvale, ernst 4

Dan nu het resultaat. In Thornvale wordt iets groots gezien in het kreupelhout, met ernst vier.

*[Wijs de kaartjes aan.]*

Het systeem stuurt drie soorten berichten. Eerst het alarm: wat, waar en wanneer. Dan wordt warden Ysolde Marn aangesproken. En daarna krijgen vier ingeschreven burgers elk een eigen bericht, met de instructie om de warden te volgen. Het blijft bij de eigen regio. De namen komen rechtstreeks uit onze tabellen.

## Slide 12 – Demo 2: Glasswater, ernst 5

*[Laat de kring groeien terwijl je praat.]*

In Glasswater is het erger. Het meer trekt zich terug, ernst vijf. Alles van daarnet gebeurt opnieuw: het alarm, warden Nerys Talvi en vijf burgers.

Maar nu komt de escalatie erbij. De drie andere regio's krijgen elk een rijksbrede waarschuwing. En tot slot: de raad wordt bijeengeroepen. Alle vier de regio's zijn nu verwittigd.

## Slide 13 – Problemen die we oplosten

Het ging niet vanzelf. Elasticsearch weigerde onze logs, omdat ons tijdveld met een underscore begon, en zulke namen zijn gereserveerd. We gebruikten een formule die in Workato niet bestaat. En Slack vond een kanaal niet.

Onze wijzigingen verdwenen ook een paar keer, waarschijnlijk door een tweede open editor-tab die de oude versie terugzette. En het grappigste probleem: als je het getal 9 en het getal 60 als tekst vergelijkt, is 9 ineens groter. Van elk van die problemen hebben we iets geleerd.

## Slide 14 – Testen

Om R1 te testen gebruiken we de negen testrequests uit de gids. Samen raken ze elke uitkomst: van een gewone melding en een duplicaat tot een onbekende regio, een ernst van zeven en een leeg event-ID. Zoals je ziet staan alle negen op ok. Ernst vier en vijf zag je daarnet ook in onze demo.

## Slide 15 – Wat we nog zouden verbeteren

We weten ook wat beter kan. Het belangrijkste voorbeeld is onze teller, dispatch_count. Die volgt de gids, maar telt de rijksbrede berichten en het raadsbericht niet mee. Bij ernst vijf komt de teller op zeven, terwijl er elf rijen in de Dispatch Log horen.

Als volgende stap zouden we een R5 bouwen, die mislukte events opnieuw probeert. En we zouden vóór elke verzending het dispatch-ID controleren, zodat een nieuwe poging nooit een dubbel alarm geeft.

## Slide 16 – Het rijk is gewaarschuwd

Het rijk is gewaarschuwd. Elk event gecontroleerd, elke melding gelogd, en elke twijfel bij een mens.

Wij zijn team OpenPrompting. Bedankt om te luisteren. Zijn er vragen?

---

# Mogelijke bijvragen

Bron: onze documentatie en de bouwgids. **[redenering]** betekent: technisch afgeleid, niet door ons getest of gedocumenteerd.
Weet je iets niet? Zeg: "Dat hebben we niet getest, maar volgens het ontwerp zou dit gebeuren: ..."

## A. De 8 die ze het waarschijnlijkst stellen

**1. Wat gebeurt er precies als een event binnenkomt?**
R1 ontvangt het via de API en checkt: duplicaat, geldig, niet te oud. Dan maakt R1 een record `received`, roept R2 aan en zet het op `dispatched`. R2 zoekt de regio op en stuurt berichten naar ernst. De afzender krijgt 202.

**2. Waarom is R2 een aparte recipe function?**
Omdat R1 en R3 hem allebei gebruiken. De alarmlogica staat zo op één plek: één keer aanpassen en beide ingangen volgen.

**3. Waarom eerst de duplicaatcheck?**
Het is de goedkoopste controle, ze voorkomt dubbel werk, en een duplicaat mag nooit opnieuw alarmeren.

**4. Waarom een 202 en geen 200?**
202 betekent "aanvaard". Een late echo krijgt ook 202, want ze is niet ongeldig, alleen achterhaald. 400 = ongeldig of mislukt, 409 = conflict met iets dat al bestaat.

**5. Wat als R2 crasht?**
De try/catch probeert één keer opnieuw na 2 seconden. Lukt het niet: rij in Failed Events, Event Log op `failed`, Slack-bericht, errorlog en 400.

**6. Wat doet de AI, en wat niet?**
De Genie zet vrije tekst om naar regio, type, ernst, betrouwbaarheid en samenvatting. Hij verstuurt niets. Onze eigen regels beslissen: triage of R2.

**7. Waarom OR en geen AND in de triage?**
Eén twijfelpunt is genoeg voor een mens. Met AND zou een rapport met onbekende regio maar 90% betrouwbaarheid gewoon doorgaan.

**8. Waarom telt dispatch_count 7 en niet 11 bij ernst 5?**
De formule uit de gids telt alarm (1), warden (1) en burgers (5). De 3 rijksbrede berichten en de raad tellen niet mee. Het ontwerp geeft wel 11 rijen in Dispatch Log.

## B. R1 – Watchtower Gate

**Wat is `v_error`?** De variabele met de validatiefout. Is ze gevuld, dan wordt het event afgewezen.
**Wat was `RECORDS_LENGTH_CHANGE_ME`?** Een placeholder. Wij zetten er `Records.length == 0` van de Regions-zoekstap: regio niet gevonden.
**Waarom moest de validatie onder de zoekstap?** Een datapill van een latere stap is leeg. Dan lijkt elke regio onbekend.
**Wat is een datapill?** Een verwijzing naar een waarde uit een eerdere stap.
**Waarom overal een Return response?** Die stopt de job. Zonder loopt R1 door en kan een afgewezen event toch alarmeren.
**Hoe bepalen jullie "te oud"?** `occurred_at.to_time < now - 24.hours`, in formulemodus.
**Loggen jullie ook duplicaten?** Ja, met status `duplicate`. Elk event laat een spoor na.
**Wat als de afzender een mislukt event opnieuw stuurt met dezelfde event_id?** [redenering] De duplicaatcheck kijkt niet naar de status, dus hij krijgt 409. Daarom willen we een R5 die Failed Events opnieuw probeert.
**Kan de retry dubbele alarmen geven?** [redenering] Ja. Crasht R2 halverwege, dan draait het try-deel opnieuw en kunnen berichten die al weg waren nog eens vertrekken. Daarom onze volgende stap: controle op `dispatch_id`.
**Waarom `retry_count = 0`?** De gids vraagt 0, maar de kolom is ja/nee. Dat staat bij onze aandachtspunten.
**Wat als twee identieke requests tegelijk komen?** [redenering] Dan kunnen beide de check passeren, want zoeken en aanmaken zijn twee stappen. Een unieke sleutel op `event_id` zou dat oplossen.

## C. R2 – Beacon Dispatch

**Hoe weten we wie een bericht krijgt?** Via `v_eff`, de effectieve ernst: > 2 warden, > 3 burgers, = 5 rijk en raad.
**Verschil tussen `v_sev` en `v_eff`?** Beide stonden al in de start-recipe: `v_sev` is de binnengekomen ernst, `v_eff` de ernst waarop R2 beslist. *[Open stap 1 in Workato om de formule te tonen; ze staat niet in onze documentatie.]*
**Welke burgers?** Drie filters: zelfde regio, `subscribed = true`, `min_severity <= v_eff`. Limiet 100.
**Meer dan 100 burgers?** [redenering] Dan mis je er. Je zou in pagina's moeten ophalen.
**Geen burgers gevonden?** Melding in ops en een log. Geen fout.
**Krijgen burgers echt een privébericht?** Volgens de gids gaat het bericht naar het regiokanaal met hun `slack_handle` erin, en krijgt Dispatch Log `channel_type = slack_dm`. In de demo stonden de berichten in het kanaal. Of de handle echt iemand bereikt, is nog te controleren.
**Krijgt de warden een e-mail?** Er gaat een Slack-bericht naar het regiokanaal. Dispatch Log noteert `channel_type = email` met het e-mailadres, zoals de gids vraagt.
**Hoe slaan jullie de regio van oorsprong over?** In de lus vergelijken we het kanaal van het huidige item met het regiokanaal. Is het hetzelfde, dan overslaan.
**Wat doet all_clear?** Zoekt events van die regio met status `dispatched`, zet ze op `resolved`, meldt SIGNAAL HERSTELD en stopt.
**Waarom stopt hij daar?** Een herstelmelding mag geen nieuw alarm geven. De Return result stopt de recipe.
**Waarom `.to_i`?** Onder ernst 4 draait de burgerstap niet en is Size leeg. `.to_i` maakt daar 0 van.
**Waarom verschillende dispatch_id's?** Zo is elke melding herkenbaar en herleidbaar naar event en ontvanger.

## D. R3 – Oracle Scout Intake

**Hoe komt het AI-antwoord terug?** Als `REGIO|TYPE|ERNST|BETROUWBAARHEID|SAMENVATTING`, dat we opsplitsen in variabelen.
**Wat als de AI een regio verzint?** We zoeken ze op in Regions. Niet gevonden = triage.
**Wat als de AI fout zit maar zeker is?** Dan gaat het door. De drempel van 60% vangt alleen onzekerheid op. [redenering] Bij ernst 5 zou een extra menselijke bevestiging kunnen.
**Waarom 60%?** Dat is de waarde uit de opdracht.
**Hoe ontstaat een lus?** R2 en R3 posten berichten, en R3 luistert mee in hetzelfde kanaal. Zonder filter leest R3 een alarm als nieuw rapport.
**Hoe stoppen jullie dat?** Filter op bots, lege afzender, en bot-emoji vooraan.
**Waarom stopt de job na triage?** Anders loopt de recipe door en wordt het twijfelgeval toch gealarmeerd.
**Gaat R3 door de controles van R1?** Nee, R3 roept R2 rechtstreeks aan. De triage controleert regio, ernst en betrouwbaarheid; `event_id` maken we zelf (`SCOUT-<tijdstempel>`).
**Twee verkenners melden hetzelfde?** [redenering] Dan zijn er twee event_id's en dus twee alarmen. R3 ontdubbelt niet.
**Waarom vergelijken als getal?** Tekst vergelijk je teken per teken. `"9" < "60"` is als tekst onwaar (9 komt na 6), dus 9% zou de triage ontlopen. En `"100" < "60"` is als tekst waar, dus 100% zou onterecht naar triage gaan.
**Maar op jullie slide staat `"5" < "60"`?** Goed gezien: dat voorbeeld klopt toevallig ook als tekst. `"9"` en `"100"` tonen het probleem wel.

## E. R4 – Chronicle Monitor

**Waarom drie zoekopdrachten?** Events, fouten en open ernstige meldingen komen uit andere tabellen of filters.
**Wat is "open ernstig"?** Status `dispatched` en ernst 4 of hoger. Na all_clear wordt het `resolved` en verdwijnt het.
**Waarom `v_worst` eerst leegmaken?** Anders blijft de standaardtekst ervoor staan.
**Waarom `v_worst` vooraan in de formule?** Zo voeg je toe. Anders overschrijft elke ronde de vorige.
**Waarom `severity.to_s`?** Getal + tekst geeft een fout in een formule.
**Wat als `now - 1.hours` niet in formulemodus staat?** Dan zoekt Workato naar die letterlijke tekst en vindt niets.
**Waarom maar 1 open ernstige op 15:13, terwijl er ook een ernst 5 was?** Dat staat niet in onze documentatie. Het is een momentopname: alleen events met status `dispatched` tellen. Glasswater kon later gekomen zijn of al opgelost zijn. *[Kijk in Event Log.]*
**Wat was die ene fout?** *[Kijk in Failed Events naar `error_message`.]*

## F. Logging, Slack en tabellen

**Waarom Elasticsearch als er al tabellen zijn?** Tabellen houden de status van events bij. Elasticsearch is om centraal te zoeken over alle recipes.
**Wat is Kibana?** Het scherm waarmee je in Elasticsearch zoekt en filtert, bij ons op `team.name` en `service.name`.
**Waarom `timestamp` en niet `@timestamp`?** `_timestamp` is geweigerd, en de organisatoren bevestigden dat een eigen naam mag. Een data stream verwacht normaal `@timestamp`; of Kibana ons veld als tijd gebruikt, is nog te controleren.
**Wat is een custom action?** Een zelf ingevuld HTTP-request via een bestaande connectie. Bij ons POST naar `logs-workato-default/_doc`.
**Waarom staat de teamnaam vast?** `project_property('TEAM_NAME')` bestaat niet als formule. Werkt, maar een property was netter.
**Waarom een kanaal-ID in plaats van een naam?** Een naam gaf `channel_not_found` in R4.
**Verschil Event Log en Dispatch Log?** Event Log: één rij per ontvangen event. Dispatch Log: één rij per verstuurde melding.
**Welke statussen bestaan er?** `received`, `dispatched`, `rejected`, `duplicate`, `late`, `failed`, `resolved`.

## G. Kritische vragen

**Wat hebben jullie echt getest?** Een run met de basispayload is gedocumenteerd als geslaagd, en ernst 4 en 5 liepen in de demo. Voor de andere testcases: antwoord alleen wat jullie zelf gedraaid hebben.
**Wat als Slack of Elasticsearch plat ligt?** [redenering] Een fout in R2 wordt in R1 door de catch opgevangen. Faalt een logstap buiten het try-blok, dan kan de job stoppen.
**Grootste zwakte?** De retry kan dubbel alarmeren, en mislukte events worden nog niet automatisch opnieuw geprobeerd.
**"Burgers > 0" staat als tekst: wat is het gevolg?** [redenering] Voor gewone aantallen geeft tekst hier toevallig hetzelfde resultaat, want elk positief getal begint met een cijfer boven 0. Een getalvergelijking is wel correcter.
**Hebben jullie AI gebruikt bij het bouwen?** Ja, een assistent hielp. Die maakte ook fouten: hij zette zelf de onbestaande `project_property`-formule erin. Wij hebben dat gevonden en hersteld.
**Wat zouden jullie anders doen?** Sneller per tak testen, en getallen en teamnaam van in het begin juist instellen.
**Wat leerden jullie het meest?** De volgorde van stappen is logica: een return vergeten of een stap te laat zetten verandert het resultaat.
**Hoe voeg je een regio toe?** [redenering] Een rij in Regions met kanaal en warden, plus burgers. R2 haalt het kanaal uit de tabel. De raadstekst "alle vier de regio's" staat wel vast.
