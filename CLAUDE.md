# Context
We bouwen "The Desync Protocol" (opdracht van I8C) in Workato.
- Workspace: HTF2026, omgeving: Development, datacenter EU (app.eu.workato.com)
- Ons project: [HTF-i8c] OpenPrompting
- De volledige specificatie staat in opdracht.pdf in deze map. Lees die altijd eerst
  vóór je aan een recipe werkt, en volg ze letterlijk (veldnamen, statussen,
  Slack-teksten, HTTP-codes, dispatch_id-formaten).
- 4 recipes: R1 Watchtower Gate, R2 Beacon Dispatch, R3 Oracle Scout Intake,
  R4 Chronicle Monitor. 5 data tables: Regions, Citizens, Event Log, Dispatch Log,
  Failed Events.

# Harde regels
- Werk ALLEEN in project [HTF-i8c] OpenPrompting. Raak [ALL] Common nooit aan.
- Wijzig nooit connecties, project properties (TEAM_NAME) of data table-schema's
  of -inhoud van Regions/Citizens.
- Start/activeer nooit een recipe tenzij ik dat expliciet vraag.
- Verwijder geen bestaande stappen uit de START-versie; vul enkel aan of verplaats
  waar de opdracht dat zegt.
- Bouw één tak per keer. Toon na elke tak een korte samenvatting van wat je
  toevoegde (stapnummer, actie, belangrijkste datapills) en wacht op mijn OK.
- Slack-berichten exact zoals in de opdracht; de emoji-prefixen
  (:white_check_mark:, :mag:) zijn functioneel en mogen niet wegvallen.
- Formules (now, now - 1.hours, .to_i, .to_s, .blank?) altijd in formulemodus.
- In lussen altijd datapills van het current item gebruiken.
- Elke tak in R1 eindigt met een Return response; in R2 eindigt de all_clear-tak
  met Return result.
- De Elasticsearch-logstap: custom action POST op logs-workato-default/_doc, JSON,
  body met _timestamp=now, message, log.level (info/error), service.name (R1-R4),
  team.name = project property TEAM_NAME. Titel van de stap: "POST Message".
- Twijfel je over iets? Vraag het mij in plaats van te gokken.
