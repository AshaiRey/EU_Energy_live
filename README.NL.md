# EU Energy Live Prices — Home Assistant integratie


## Wat doet hij

1. Bij het toevoegen vraag je **e-mailadres** en **label** in — dit zijn
   invulbare velden in de configuratie-UI, precies zoals gevraagd.
2. De integratie roept daarmee automatisch éénmalig
   `POST /api/v1/keys` aan en bewaart het ontvangen `token`.
3. Elke dag op het **door jou tijdens installatie ingestelde tijdstip**
   (lokale HA-tijd, standaard 00:10) wordt
   `GET /api/v1/prices/today?zone=<zone>` aangeroepen met
   `Authorization: Bearer <token>`.
4. **Tokenbeheer:** we weten niet hoe lang een token geldig blijft, dus bij
   elke ophaalactie wordt eerst het bestaande token gebruikt. Krijgt de API
   een 401/403 terug, dan vraagt de integratie **automatisch een nieuw
   token** aan (met dezelfde email/label) en slaat dat op — zonder dat je
   iets hoeft te doen. Blijft het token wél gewoon geldig, dan wordt het
   gewoon hergebruikt.
5. De data wordt beschikbaar gemaakt als sensoren:
   - 5 referentie-sensoren: **Zone, Land, Valuta, Eenheid (bron), Datum**
   - 24 sensoren, één per uur van de dag (`Prijs 00:00` t/m `Prijs 23:00`),
     met de prijs in **EUR/kWh, afgerond op 4 decimalen** — ideaal om
     automatiseringen op te triggeren.

## Installatie

1. Kopieer de map `custom_components/euenergy_live` naar de
   `custom_components`-map van je Home Assistant config
   (of voeg deze repo toe als custom repository in HACS).
2. Herstart Home Assistant.
3. Ga naar **Instellingen → Apparaten en diensten → Integratie
   toevoegen → "EU Energy Live Prices"**.
4. Vul in:
   - **E-mailadres**
   - **Label** (standaard: "Home Assistant")
   - **Zone** (standaard: "NL")
   - **Dagelijks ophaaltijdstip** (standaard: 00:10) — dit tijdstip
     kies je hier eenmalig bij installatie.
5. Klaar — de integratie haalt direct de eerste keer data op, en daarna
   iedere dag op het door jou ingestelde tijdstip.

## Ophaaltijdstip achteraf wijzigen

Ga naar **Instellingen → Apparaten en diensten → EU Energy Live Prices →
Configureren** (het tandwiel-icoon) om het dagelijkse ophaaltijdstip op
elk moment aan te passen. De integratie herlaadt zichzelf automatisch en
gebruikt vanaf dat moment het nieuwe tijdstip.

## Handmatig verversen

Er is een knop-entiteit **"Refresh"** beschikbaar op het apparaat. Druk
hierop om de data direct opnieuw op te halen, zonder op het geplande
tijdstip te hoeven wachten — handig bij testen of als je zeker wilt zijn
van de laatste stand.

## Sensoren en entity-ID's

- `sensor.eu_energy_live_nl_zone`
- `sensor.eu_energy_live_nl_land`
- `sensor.eu_energy_live_nl_valuta`
- `sensor.eu_energy_live_nl_eenheid_bron`
- `sensor.eu_energy_live_nl_datum`
- `sensor.eu_energy_live_nl_prijs_0000` t/m `sensor.eu_energy_live_nl_prijs_2300`
- `sensor.eu_energy_live_nl_laatste_refresh` — tijdstip van de laatst
  geslaagde data-ophaling (automatisch of via de Refresh-knop, dat maakt
  voor deze sensor geen verschil)
- `button.eu_energy_live_nl_refresh` — knop om de data handmatig te
  verversen

Elke uursensor heeft ook attributen:
- `price_eur_mwh` — de originele prijs in EUR/MWh zoals de API die levert
- `utc_timestamp` — het originele UTC-tijdstip
- `local_time` — hetzelfde tijdstip in lokale HA-tijdzone

### Voorbeeld automatisering

```yaml
automation:
  - alias: "Was aanzetten bij lage energieprijs"
    trigger:
      - platform: time
        at: "14:00:00"
    condition:
      - condition: numeric_state
        entity_id: sensor.eu_energy_live_nl_prijs_1400
        below: 0.05
    action:
      - service: switch.turn_on
        target:
          entity_id: switch.wasmachine
```

## Let op: zomertijd/wintertijd

Bij de overgang naar zomer-/wintertijd heeft een dag 23 of 25 uur. De
integratie matcht elk API-uur op het **lokale uur van de dag**; op een
25-uursdag wordt bij het dubbele uur simpelweg het laatst gevonden item
aangehouden, op een 23-uursdag is die ene uursensor tijdelijk
`unavailable`. Voor de overige 23 sensoren verandert er niets.

## Attributie

De API-bron vereist attributie: "Data via euenergy.live (CC-BY-4.0).
Attribution required." — dit staat automatisch als `attribution` op elke
sensor.


*(**Disclaimer**: *
*While the coding was rather simple I used Claude.ai to check the code and asked for improvements. Using this code is at your own risk and I will not be responsible for any provided code nor for the presented data.)*
