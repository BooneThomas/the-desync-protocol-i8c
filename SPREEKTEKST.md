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

Onze wijzigingen verdwenen ook een paar keer, waarschijnlijk door een tweede open editor-tab die de oude versie terugzette. En het grappigste probleem: als je het getal 5 en het getal 60 als tekst vergelijkt, krijg je een fout resultaat. Van elk van die problemen hebben we iets geleerd.

## Slide 14 – Testen

Om R1 te testen gebruiken we de negen testrequests uit de gids. Samen raken ze elke uitkomst: van een gewone melding en een duplicaat tot een onbekende regio, een ernst van zeven en een leeg event-ID. Zoals je ziet staan alle negen op ok. Ernst vier en vijf zag je daarnet ook in onze demo.

## Slide 15 – Wat we nog zouden verbeteren

We weten ook wat beter kan. Het belangrijkste voorbeeld is onze teller, dispatch_count. Die volgt de gids, maar telt de rijksbrede berichten en het raadsbericht niet mee. Bij ernst vijf komt de teller op zeven, terwijl er elf rijen in de Dispatch Log horen.

Als volgende stap zouden we een R5 bouwen, die mislukte events opnieuw probeert. En we zouden vóór elke verzending het dispatch-ID controleren, zodat een nieuwe poging nooit een dubbel alarm geeft.

## Slide 16 – Het rijk is gewaarschuwd

Het rijk is gewaarschuwd. Elk event gecontroleerd, elke melding gelogd, en elke twijfel bij een mens.

Wij zijn team OpenPrompting. Bedankt om te luisteren. Zijn er vragen?
