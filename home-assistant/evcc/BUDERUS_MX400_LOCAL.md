# Buderus MX400 Local API in Home Assistant and evcc

The custom integration `buderus_mx400_local` reads the MX400/K 40 RF Local API directly over the LAN. It does not use SingleKey, MyBuderus, or the PointT cloud API. The manufacturer currently documents this local API as read-only; the integration therefore creates sensors only.

## Home Assistant setup

1. Copy `custom_components/buderus_mx400_local` into Home Assistant's `custom_components` directory and restart Home Assistant.
2. In **Settings > Devices & services**, add **Buderus MX400 Local API**.
3. Enter the MX400 hostname or IP address, plus the `Login` and `Pass` printed on the gateway label. Do not include a port or URL scheme. Password hyphens are stripped automatically.
4. Immediately before submitting, press the gateway's WLAN and Wireless buttons together for one second. The gateway requires token creation within five minutes.

The gateway firmware must be version `15.00.01` or newer. The generated local access token is stored in the Home Assistant config entry; the gateway password is not stored. The token does not expire according to the manufacturer documentation. The API uses HTTPS, but the vendor currently documents a certificate-verification limitation; the integration follows the vendor's local API guidance and disables certificate verification. Keep the gateway and Home Assistant on a trusted LAN.

Entities are created only when the connected appliance exposes their resource. They include compressor and electric-heater power in W, their component sum, flow/return/outdoor/room/DHW temperatures, heat demand and cumulative kWh counters. The sum only includes compressor and electric-heater values; it is not guaranteed to include every pump or auxiliary load. The installation's firmware and configuration determine the exact available set. The configured polling interval defaults to 30 seconds.

## evcc meter

evcc already has a Home Assistant meter template, so a second evcc-specific software integration is not needed for reading the new sensors. For monitoring only, register the heat pump as an external meter. This keeps the reading visible without feeding it into the site control point. Do not add it as an auxiliary meter when the grid meter already measures the heat pump, since that can double-count the load. The template accepts a lifetime energy counter in kWh as an optional energy input.

Example values for an evcc Home Assistant meter:

```yaml
type: homeassistant
uri: http://<home-assistant-host>:8123
power: sensor.<mx400_total_electrical_power>
energy: sensor.<mx400_total_electricity>
```

Replace the placeholders with the actual entity IDs shown by Home Assistant. The evcc Home Assistant meter expects numeric power values (W/kW) and, optionally, a cumulative energy sensor (Wh/kWh). Do not use the MX400's cumulative energy counter as a substitute for instantaneous power. Compare the component sum with a dedicated electrical meter before relying on it for energy accounting.

For a YAML-configured evcc instance, add the meter name to `site.meters.ext` so evcc only monitors it:

```yaml
site:
	meters:
		ext:
			- buderus_hp
```

## evcc heating-device control

evcc supports a heating-device loadpoint, but its generic `heatpump` charger requires a `setmaxpower` control path. The MX400 Local API is read-only, so these sensors alone cannot make a controllable evcc heat-pump loadpoint. Add that only when a separate supported actuator exists, for example a correctly wired SG Ready input controlled by an evcc-supported relay. Do not switch the heat pump's mains supply for this purpose.

## Future Buderus Open Web API

The integration separates resource acquisition from Home Assistant sensor definitions. When Buderus publishes the Open Web API specification, a new provider can implement the same read-only resource contract without changing entity IDs or dashboard cards. Its authentication, permissions, and write capabilities must be assessed from the published specification rather than assumed from the MX400 Local API.

## References

- [Buderus/Bosch MX400 Local API documentation](https://github.com/bosch-home-comfort/api-docs)
- [Buderus Open Web API announcement](https://www.buderus.de/de/heiztechnik-verstehen/schnittstellen-buderus-108842#Open_Web_API)
- [evcc Home Assistant meter template](https://github.com/evcc-io/evcc/blob/master/templates/definition/meter/homeassistant.yaml)
- [evcc generic heat-pump charger example](https://github.com/evcc-io/evcc/blob/master/assets/js/components/Config/defaultYaml/heatpump.yaml)