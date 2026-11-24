# Gedachtengang per recept: wat doet elk blokje?
Team OpenPrompting · HTF2026 · The Desync Protocol

**Het grote plaatje:** R1 en R3 laten events binnen, R2 verstuurt de waarschuwingen,
R4 bewaakt het geheel.

```
Gate-event ──► R1 ──┐
                    ├──► R2 ──► scout-kanaal (alarm, warden, burgers, raad)
Scout-rapport ─► R3 ┘
R4 (elke 5 min) ──► ops (status)
```

---

## R1 – Watchtower Gate (de poort)
**Idee:** niets mag binnen zonder controle. Eerst de goedkoopste controles, dan het zware werk.

| Stap | Wat gebeurt er | Waarom |
|---|---|---|
| 1. API-trigger | Een event komt binnen via een HTTP-aanroep | Zo sturen de organisatoren events |
| 2. Duplicaatcheck | Zoek `event_id` in de Event Log | Dubbel verwerken geeft dubbele alarmen |
| 2a. Bij duplicaat | Status `duplicate`, melding in ops, log, antwoord **409** | De zender weet wat er gebeurde |
| 3. Regiovalidatie | Bestaat `region_code` in de Regions-tabel? | Onbekende regio kan niet gealarmeerd worden |
| 4. Validatie | Controle op bijvoorbeeld lege `event_id` of ernst buiten 1-5 | Slechte data tegenhouden |
| 4a. Bij fout | Status `rejected`, melding in ops, log, antwoord **400** | Duidelijke reden terug |
| 5. Late echo | Is `occurred_at` meer dan 24 uur oud? | Oud nieuws is geen alarm |
| 5a. Bij late echo | Status `late`, log, antwoord **202** | Geaccepteerd, maar zonder alarm |
| 6. Monitor-blok (try) | Event opslaan met status `received`, R2 aanroepen, status `dispatched` | De echte verwerking |
| 6a. Monitor-blok (catch) | Bij een fout: naar Failed Events, status `failed`, melding in ops, antwoord **400** | Een event mag niet stil verdwijnen |
| 7. Antwoord | Return **202** | Event is aangenomen |

**Waarom die volgorde?** Goedkoop en snel eerst (duplicaat), daarna de controles,
en pas als alles klopt de dure verwerking met een vangnet.

---

## R2 – Beacon Dispatch (de verzender, recipe function)
**Idee:** hoe ernstiger, hoe meer mensen er gewaarschuwd worden.

| Stap | Wat gebeurt er | Waarom |
|---|---|---|
| 1. Invoer | Ontvangt `event_id`, `region_code`, `severity`, `headline`, `details`, ... | R1 en R3 geven dit door |
| 2. Variabelen | Ernst, titel, tijd (Brusselse tijd) en label (RUIS ... KRITIEK) | Zo hoeft niet elke stap zelf te rekenen |
| 3. Regio opzoeken | Zoek de regio in de Regions-tabel | Kanaal en warden komen uit data, niet uit het recept |
| 4. All_clear? | Zo ja: alarmen van die regio op `resolved`, bericht "SIGNAAL HERSTELD", stop | Een herstel is geen nieuw alarm |
| 5. Basisalarm | Altijd: bericht "DESYNC ALERT", log, Dispatch Log-regel | Elk event verdient minstens een melding |
| 6. Warden (ernst 3+) | Bericht dat de warden aanspreekt, Dispatch Log-regel | Een verantwoordelijke moet het weten |
| 7. Burgers (ernst 4+) | Zoek ingeschreven burgers met voldoende lage minimumernst; bericht per burger | Alleen wie dit wil, krijgt het |
| 7a. Geen burgers | Melding in ops | Het systeem blijft niet stil |
| 8. Rijksbreed (ernst 5) | Voor elke andere regio een waarschuwing (op regiocode vergeleken) | De oorsprongsregio heeft al een alarm gekregen |
| 9. Raad (ernst 5) | Eén melding "DE RAAD WORDT BIJEENGEROEPEN" | Het hoogste niveau |
| 10. Resultaat | `dispatch_count` en `channels_used` teruggeven | De aanroeper ziet wat er gedaan is |

**Ernstregel:**

| Ernst | Wie krijgt het |
|---|---|
| 1-2 | regio |
| 3 | regio + warden |
| 4 | + burgers |
| 5 | + andere regio's + raad |

Alles wat verstuurd wordt, komt ook in de Dispatch Log en in Elasticsearch, zodat we kunnen nagaan wat er gebeurd is.

---

## R3 – Oracle Scout Intake (de vertaler)
**Idee:** scouts schrijven vrije tekst, R2 wil gestructureerde data. De AI vertaalt, een mens beslist bij twijfel.

| Stap | Wat gebeurt er | Waarom |
|---|---|---|
| 1. Slack-trigger | Een nieuw bericht in openprompting-scout-reports | Daar plaatsen scouts hun rapporten |
| 2. Filter | Stop bij andere kanalen, lege tekst, bots, geen afzender en berichten die met een bot-emoji beginnen | R2 post in hetzelfde kanaal, dus anders ontstaat een lus |
| 3. Genie "Achrona Oracle" | Zet de tekst om naar regio, ernst, type, titel, betrouwbaarheid | De AI begrijpt vrije tekst |
| 4. Variabelen | Waarden opslaan | Zo kunnen volgende stappen ze gebruiken |
| 5. Regio opzoeken | Bestaat de regio in de Regions-tabel? | De AI kan een regio verkeerd lezen |
| 6. Triage-check | Ernst < 1, betrouwbaarheid < 60% of onbekende regio? Vergelijking met getallen | Twijfel gaat naar een mens |
| 6a. Twijfelgeval | Bericht in ops: ":mag: Rapport vraagt menselijke triage" met het origineel | Een mens beslist |
| 7. Hoofdroute | Event loggen en R2 aanroepen | Zelfde alarmketen als bij R1 |
| 8. Bevestiging | Bericht in ops: ":white_check_mark: Scout-rapport verwerkt als ..." | Het team ziet wat er verwerkt is |

**Waarom de drempel van 60%?** De AI mag niet blind alarmen veroorzaken. Zekerheid beslist.

---

## R4 – Chronicle Monitor (de bewaker)
**Idee:** één blik op ops moet genoeg zijn om te zien hoe het systeem het doet.

| Stap | Wat gebeurt er | Waarom |
|---|---|---|
| 1. Tijdschema | Elke 5 minuten | Actueel zonder te veel berichten |
| 2. Events van het laatste uur | Zoeken in de Event Log (max. 200) | Hoeveel is er verwerkt? |
| 3. Fouten van het laatste uur | Zoeken in Failed Events (max. 100) | Hoeveel is er misgegaan? |
| 4. Open ernstige meldingen | Status `dispatched`, ernst 4+, aflopend op ernst (max. 5) | Wat vraagt nu aandacht? |
| 5. Variabelen | `v_worst` (standaardtekst) en `v_when` (Brusselse tijd) | Voorbereiden van het bericht |
| 6. Voorwaarde | Zijn er open ernstige meldingen? | Anders blijft de standaardtekst staan |
| 7. Lus | Per melding een regel ". sev 4 - titel" toevoegen | De zwaarste staat bovenaan |
| 8. Slack-bericht | Statusbericht in ops | Eén compact overzicht |
| 9. Elasticsearch | Hetzelfde bericht loggen | Voor het dashboard |

---

## Gemeenschappelijke ontwerpkeuzes
- **Data in tabellen, niet in het recept:** kanalen, wardens en burgers kunnen veranderen zonder dat een recept moet worden aangepast.
- **Getallen vergelijken als getallen:** bij tekst is "5" kleiner dan "60".
- **Niets verdwijnt stil:** elke fout komt in Failed Events en in ops.
- **Alles wordt gelogd:** Slack, Dispatch Log en Elasticsearch laten zien wat er gebeurd is.
- **Kanalen als vast ID:** een naam gaf `channel_not_found`.