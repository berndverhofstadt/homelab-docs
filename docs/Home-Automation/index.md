---
authors:
    - Bernd Verhofstadt
weight: 40
---
# Home automation with openHAB

openHAB is the central controller for the home-automation setup. It combines
devices and integrations (including MQTT) and provides one place for rules,
dashboards and manual controls.

The current Compose design groups openHAB, the MQTT broker, and device bridges
on a separate Unraid host. That host has its own Docker networks and is
intended for LAN-only access; it is not connected to the public tunnel. Names
such as `openhab.internal.example.org` are illustrative. Verify DNS, tunnel
ingress and off-LAN reachability before relying on this exposure boundary.

## Overview

The design separates network access by purpose:

- Traefik and the openHAB interface use the host's `proxy` network.
- openHAB and its MQTT consumers also use `shared-services` to reach the
  broker.
- Device bridges have no public hostname; they communicate with MQTT and
  their respective devices on the LAN.
- openHAB's configuration and userdata use persistent storage and require
  backup coverage.

The two hosts do not share Docker networks. The secondary host's
`proxy`/`shared-services`/`traefik-support` networks are independent from
same-named networks on the primary host. See
[Docker Compose and network segmentation](../Networking/Docker-Network-Segmentation.md).

Before relocating an existing installation, copy and verify the full
configuration and userdata, preserve MQTT credentials and topic ACLs, and
confirm that the second host can reach the physical devices. Do not treat a
Compose file alone as a data migration.

## Structure: Things, Items, Rules and UI

### Things

"Things" represent physical or logical devices, such as energy meters, relays,
heat-pump controls and Zigbee bridges. Things can be configured through
MainUI and stored in JSONDB, or defined with files under `/openhab/conf`.

### Items

Items represent states, commands and values used by rules and the UI.
Examples include current grid import/export power and the two smart-grid
control relays.

### Rules

Rules implement the automation logic: *if X happens, do Y*. JavaScript rules
can evaluate power, schedules and other state to decide which smart-grid mode
should be active.

### UI (MainUI)

MainUI provides overview pages, manual controls and charts. Group widgets by
room or function to keep navigation manageable.

## Integrations and related pages

The setup uses device bindings and network integrations such as MQTT and HTTP.
The [Smart Grid Automation](Smart-Grid-Automation.md) page describes one
openHAB automation example. The [Networking overview](../Networking/index.md)
and [Traefik reverse proxy](../Networking/Reverse-Proxy.md) pages describe
network segmentation and routing.
