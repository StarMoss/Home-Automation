# Smart Home Control Center: Design Docs

A documentation-first example of planning a centralized smart home control center. It uses Home Assistant as the integration layer for devices from several vendors, with an optional custom dashboard built on top.

This repository is a **public demonstration**. It contains design documents and templates only, with no code and no real device data. A separate private repository holds the actual deployment details.

## The problem

Consumer IoT devices rarely share a discovery or authentication model. The example household in these docs has:

| Device family | Ecosystem | How it is reached |
|---|---|---|
| Google Nest thermostat | Google cloud | Official Smart Device Management (SDM) API, OAuth2 |
| VOCOlinc smart plugs | HomeKit Accessory Protocol | Local LAN, mDNS discovery, HAP pairing |
| Lenovo smart plugs and lights | Believed to be Tuya-based | Tuya/Smart Life cloud (unverified, see below) |
| Reolink cameras (proposed) | Local LAN | Official Home Assistant integration |

Writing and maintaining a separate integration for each of these is a lot of work. The approach documented here puts Home Assistant in the middle. It handles each vendor's onboarding and normalizes every device into one entity model. A custom control center then talks to a single REST/WebSocket API.

```
Custom control center (UI)
        |  REST + WebSocket
        v
   Home Assistant
   |        |        |        |
 Nest SDM   HAP     Tuya     Reolink
 (OAuth2)  (LAN)    cloud    (LAN)
```

## What is in this repo

| File | Purpose |
|---|---|
| [`smart-home-control-center-adr.md`](smart-home-control-center-adr.md) | **ADR-001** (status: Proposed). Context, options considered, the decision to use Home Assistant, per-device integration paths, hosting options, risks, and an implementation plan. |
| [`adr-decision-backlog.md`](adr-decision-backlog.md) | **ADR-002 to ADR-011** as unfilled templates: hosting hardware, network segmentation, remote access, backups, secrets, camera storage, the Lenovo/Tuya path, custom UI architecture, update policy, and config management. Each lists options, drivers, a blank Decision section, and a revisit trigger. |
| [`device-inventory.md`](device-inventory.md) | A template for tracking devices, network identity, firmware, account dependencies, pairing notes, and per-device acceptance tests. |

Suggested reading order: ADR-001, then the backlog index, then the inventory.

## Status

Planning only. Nothing here has been built or deployed from this repo.

- ADR-001 is **Proposed**, not accepted.
- ADR-002 to ADR-011 are **Open** templates. The "leaning" notes in them are suggestions, not decisions.
- The device inventory is a template with placeholder values.

## How to read the confidence labels

The docs mark how sure they are, because several claims came from vendor docs and community forum posts, not hands-on testing. In particular:

- **Lenovo devices being Tuya-based** is inferred from forum reports (moderate-low confidence) and needs a verification test (see ADR-008).
- **Ring's Home Assistant support** is not verified.
- **Hardware prices** (such as Home Assistant Green) were not checked.
- Vendor integrations change. Check each linked doc before acting on it.

This project is not affiliated with Home Assistant, Google, VOCOlinc, Lenovo, Tuya, Reolink, or Ring. Product names are used to describe compatibility only.

## Public demo vs. private project

This repo is intentionally generic. The real deployment lives in a private repo and should follow these rules:

**Do not commit, in either repo:**
- Passwords, API keys, OAuth client secrets, long-lived tokens, VPN keys
- HomeKit setup codes or pairing keys
- Home Assistant `secrets.yaml`, backups, or `.storage/` contents (they contain credentials)

**Keep out of the public repo (fine in the private one):**
- Real MAC addresses, IP ranges, VLAN layout, SSIDs
- Real account emails and project IDs
- Firewall rules, camera locations, and your home layout
- Filled-in versions of `device-inventory.md`

The device inventory is the file most likely to leak details by accident. Keep the public copy as an empty template and fill in the real one only in the private repo.

## Using these docs for your own project

1. Copy the repo (or just the three files) into a **private** repository.
2. Work through the backlog in the order suggested in its index, filling in **Decision** and **Consequences** and setting each ADR's status to Accepted.
3. Move each finished ADR into its own file if you prefer one file per decision.
4. Fill in the device inventory as you pair devices, recording where secrets live but never the secrets themselves.

## Key links

- Google Device Access (Nest): https://developers.google.com/nest/device-access
- Home Assistant Nest integration: https://www.home-assistant.io/integrations/nest/
- Home Assistant Reolink integration: https://home-assistant.io/integrations/reolink
- Tuya Smart Life integration for Home Assistant: https://github.com/tuya/tuya-smart-life
- Home Assistant REST API: https://developers.home-assistant.io/docs/api/rest/
