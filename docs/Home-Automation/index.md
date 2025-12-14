---
authors:
	- Bernd Verhofstadt
weight: 40
---
# Home Automation: OpenHab

OpenHab is the central brain of my home automation setup. It ties together devices from different vendors (Shelly, Atlantic, Zigbee, MQTT, …) and exposes a single place where all the logic, dashboards and integrations live.

In this section I describe how I run OpenHab in my homelab on Unraid, how I structure things (things, items, rules and UI), and how it connects to other components described in this documentation (for example the Smart Grid integration of the Atlantic Explorer v5 heat pump boiler).

## Overview of the setup

At a high level, the OpenHab setup looks like this:

- **Platform**: Unraid server, running OpenHab as a Docker container
- **Network**: OpenHab is on the same Docker / LAN networks as other services (Shelly devices, MQTT broker, energy meter, …)
- **Storage**: configuration and userdata are mapped to persistent paths on the Unraid array
- **Access**: the main UI is available via the browser, optionally fronted by Traefik (see networking section)

OpenHab is responsible for:

- Collecting state from devices (power usage, temperatures, contact states, …)
- Exposing manual controls via the MainUI (switches, setpoints, charts)
- Running automation rules (for example: control of the Atlantic heat pump boiler smart‑grid contacts)

## Running OpenHab on Unraid (Docker)

OpenHab runs as a Docker container managed by Unraid. The exact UI steps depend on your Unraid template, but conceptually the container has:

- A **config volume** mapped to the OpenHab `conf` directory (things, items, rules, transformations, …)
- A **userdata volume** for persistence (logs, mapdb/rrd4j/influxdb connections, etc.)
- One or more **network attachments** so it can communicate with your devices (typically the main bridge / br0 LAN)

Example volume mappings you typically see:

| Host path                               | Container path  | Purpose                |
|-----------------------------------------|-----------------|------------------------|
| `/mnt/user/appdata/openhab/conf`       | `/openhab/conf` | Configuration files    |
| `/mnt/user/appdata/openhab/userdata`   | `/openhab/userdata` | Runtime data and logs |

Once the container is started, the OpenHab UI is available on the configured HTTP port, for example:

- `http://openhab.local:8080` or `http://<unraid-ip>:8080`

If you are using Traefik as reverse proxy, you can expose OpenHab via a friendly hostname and HTTPS (see the Traefik page in the Networking section).

## Structure: Things, Items, Rules and UI

I try to keep a clear structure inside OpenHab so that it remains maintainable over time.

### Things

"Things" represent the physical or logical devices in the system:

- Shelly devices (for example a Shelly Pro 2 controlling the heat pump boiler smart‑grid inputs)
- Energy meters (grid import / export)
- Heat pump boiler (via Modbus or other bindings, if available)
- Zigbee bridges, etc.

Most of my Things are configured via the MainUI and stored in the JSONDB, but the approach also works with file‑based `.things` definitions in `/openhab/conf/things`.

### Items

Items represent states, commands and values that rules and the UI use. Examples:

- `energyhome_Vermogen` – current grid import/export power (W)
- `HPB_SG_1`, `HPB_SG_2` – the two smart‑grid control relays on the Shelly Pro 2
- `HPB_Mode` – a string item representing the current logical mode of the heat pump boiler (Normal, Comfort, Boost, Off)

Items are the main building blocks you will see referenced in rules and in the UI.

### Rules

Rules implement the actual automation logic: *if X happens, do Y*. For more advanced logic I use JavaScript (JS Scripting) rules.

One example is the **Smart Grid Ready** rule for the Atlantic Explorer v5 heat pump boiler, which runs every minute and decides which SG mode should be active based on power injection and time of day. The full rule and explanation are documented on the dedicated Smart Grid page.

### UI (MainUI)

OpenHab’s MainUI is used for:

- Overview pages (current power usage, boiler status, temperatures)
- Manual control widgets (boost buttons, mode selectors)
- Charts and historical data

I group widgets per room or per function (energy, heating, hot water, …) to keep navigation clean.

## Integrations used

The most relevant bindings / integrations in this homelab are:

- **Shelly binding** – for direct control and monitoring of Shelly devices. Used for the Shelly Pro 2 driving the SG contacts of the Atlantic heat pump boiler.
- **Network / MQTT / HTTP** – depending on the device, some integrations are done via MQTT topics or HTTP APIs.
- **Energy metering** – to get live import/export values that drive automations such as the Smart Grid rule.

These integrations provide the raw data; OpenHab rules turn that into behaviour.

## Relation to other pages

OpenHab appears in several other parts of this documentation:

- The **Smart Grid Automation** page describes how OpenHab controls the Atlantic Explorer v5 heat pump boiler via a Shelly Pro 2 and SG Ready contacts, including the full JS rule.
- Networking and reverse proxy pages show how services like OpenHab can be exposed securely via Traefik.

This page is meant as the high‑level entry point for the home automation stack; detailed configurations for specific use‑cases are documented in their respective sub‑pages.

