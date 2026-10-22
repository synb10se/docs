# evcc Price Optimization

[Deutsche Version](EVCC_PREISOPTIMIERUNG.md)

## Requirements

- The evcc planner tariff must be configured to use electricity prices.
- The `sensor.synboxen_forecast_planner` entity (planner tariffs via the Tariff API) and the other Tariff API forecasts (solar generation, feed-in, and electricity prices) must be enabled in Home Assistant. The integration retrieves enabled tariff forecasts at startup.
- The smart price limits for SynBox-01, SynBox-02, and the home battery must be available in evcc.
- The `select.synboxen_battery_mode` entity must be enabled so Home Assistant can renew the temporary charge lock.

## Operation

The **evcc Price Optimization Enabled** switch (`input_boolean.evcc_smart_price_optimizer`) enables or pauses the automation. It is listed with the Home Assistant helpers. A newly created switch starts off; Home Assistant restores its previous state after restarts.

When enabled, the automation calculates price limits and a battery target daily at 22:00:

- Wallboxes: the cutoff for the four cheapest hours in total. The same value is applied to SynBox-01 and SynBox-02.
- Home battery: the target state of charge accounts for the previous night's average consumption, the solar forecast from 08:00 to 22:00, and a minimum 20% reserve for the 22:00–08:00 period. This plans enough energy for overnight use while leaving room for expected PV generation. The current battery state of charge and capacity are included. Required grid-charging time is converted into the cheapest planner intervals using an assumed maximum charging power of 10.8 kW.

Price thresholds are converted from the EPEX forecast to the configured gross price using the configured surcharge and VAT: `(EPEX price + EUR 0.18/kWh) × 1.19`. The result is then rounded up to evcc's 0.005 EUR/kWh step size. In Smart mode, evcc charges from the grid when the planner price does not exceed the relevant limit. Once the battery target is reached, Home Assistant sets evcc's temporary **lock charging** mode every 30 seconds; the mode expires automatically after 60 seconds. The lock is not renewed while the battery is below its target and is cleared at 08:00.

At 08:00, Home Assistant stores the average house power from the previous ten hours as the consumption estimate for the next night. On the first run, it uses the 24-hour average instead. The assumed charging power is a measured peak; if actual charging power is lower, evcc may need more low-price intervals. If a forecast is unavailable, has too few intervals, or produces invalid price limits, existing price limits are left unchanged and the battery target is set to 0. Changing the switch takes effect at the next 22:00 calculation.

## Flow

```mermaid
flowchart TD
    A[22:00: optimizer enabled] --> B[Read planner and solar forecasts]
    B --> C[Read saved night use; first run uses 24 h mean]
    C --> D[Compute target: at least 20 percent plus night use]
    D --> E[Account for forecast PV headroom from 08:00 to 22:00]
    E --> F[Estimate grid charging time at 10.8 kW]
    F --> G[Set cutoff from cheapest planner intervals]
    G --> H{Battery SOC reached target?}
    H -- No --> I[Allow Smart grid charging]
    H -- Yes --> J[Set holdcharge; renew every 30 seconds]
    I --> H
    J --> H
    K[08:00] --> L[Store previous 10 h average consumption]
    L --> M[Clear holdcharge]
```

## Configuration

The helpers are defined in `configuration.yaml`; the automations are in `automations.yaml`. Entity IDs used by the configuration:

- `sensor.synboxen_forecast_planner`
- `number.synboxen_synbox_01_smart_cost_limit`
- `number.synboxen_synbox_02_smart_cost_limit`
- `number.synboxen_battery_grid_charge_limit`
- `sensor.synboxen_tariff_api_solar`
- `sensor.solarnet_leistung_verbrauch`
- `sensor.synboxen_battery_soc`
- `sensor.synboxen_battery_capacity`
- `select.synboxen_battery_mode`
- `input_number.evcc_night_consumption_kwh`
- `input_number.evcc_battery_target_soc`
