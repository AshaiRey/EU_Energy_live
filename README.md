# EU Energy Live Prices — Home Assistant integration


## What it does

1. When adding the integration you enter your **email address** and
   **label** — these are configurable fields in the setup UI.
2. The integration automatically calls `POST /api/v1/keys` once with
   these details and stores the returned `token`.
3. Every day at the **time you set during installation** (local HA time, default 00:10), it calls
   `GET /api/v1/prices/today?zone=<zone>` with `Authorization: Bearer <token>`.
4. **Token management:** since it's unknown how long a token stays
   valid, every fetch first reuses the stored token. If the API returns
   a 401/403, the integration **automatically requests a new token**
   (using the same email/label) and saves it — no action needed from
   the user. If the token is still valid, it's simply reused.
5. The following data is exposed as sensors:
   - 5 reference sensors: **Zone, Country, Currency, Unit (source),
     Date**
   - 24 sensors, one per hour of the day (`Price 00:00` through
     `Price 23:00`), with the price in **EUR/kWh, rounded to 4
     decimals** — ideal for triggering automations.

## Installation

1. Copy the `custom_components/euenergy_live` folder into the
   `custom_components` folder of your Home Assistant config
2. Restart Home Assistant.
3. Go to **Settings → Devices & services → Add integration →
   "EU Energy Live Prices"**.
4. Fill in:
   - **Email address**
   - **Label** (default: "Home Assistant")
   - **Zone** (default: "NL")
   - **Daily fetch time** (default: 00:10) — chosen once here during
     installation.
5. Done — the integration fetches data immediately the first time, and
   then every day at the time you configured.

## Changing the fetch time later

Go to **Settings → Devices & services → EU Energy Live Prices →
Configure** (the gear icon) to change the daily fetch time at any
time. The integration automatically reloads and uses the new time from
that point on.

## Manual refresh

A button entity named **"Refresh"** is available. Press
it to fetch the data again immediately, without waiting for the
scheduled time — handy for testing or when you want to be sure you
have the latest figures.

## Sensors and entity IDs

- `sensor.eu_energy_live_nl_zone`
- `sensor.eu_energy_live_nl_land` (Country)
- `sensor.eu_energy_live_nl_valuta` (Currency)
- `sensor.eu_energy_live_nl_eenheid_bron` (Unit, source)
- `sensor.eu_energy_live_nl_datum` (Date)
- `sensor.eu_energy_live_nl_prijs_0000` through
  `sensor.eu_energy_live_nl_prijs_2300` (Price 00:00–23:00)
- `sensor.eu_energy_live_nl_laatste_refresh` (Last refresh) — the
  timestamp of the most recent successful fetch (automatic or via the
  Refresh button, it makes no difference for this sensor)
- `button.eu_energy_live_nl_refresh` — button to manually refresh the
  data

Note: entity IDs are derived from the (Dutch) entity names defined in
the code, so they keep the `_nl` / Dutch wording shown above regardless
of your Home Assistant language setting; only the *display* names in
the UI follow your chosen language via the translation files.

Each hourly sensor also has these attributes:
- `price_eur_mwh` — the original price in EUR/MWh as delivered by the API
- `utc_timestamp` — the original UTC timestamp
- `local_time` — the same timestamp in the local HA timezone

### Example automation

```yaml
automation:
  - alias: "Turn on washing machine at low energy price"
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
          entity_id: switch.washing_machine
```

## Note: daylight saving time (DST)

On the day of a DST transition, a day has 23 or 25 hours. The
integration matches each API hour to the **local hour of the day**; on
a 25-hour day, whichever entry is found last for the duplicated hour
is kept, and on a 23-hour day that one hourly sensor is temporarily
`unavailable`. The other 23 sensors are unaffected.

## Attribution

The API source requires attribution: "Data via euenergy.live (CC-BY-4.0). Attribution required." 
This is automatically set as the `attribution` property on every sensor.
