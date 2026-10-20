# Smart Home Device Inventory

## Note: This is a demo template. DO NOT commit the real template to a public repo.

Companion to `smart-home-control-center-adr.md` (ADR-001) and `adr-decision-backlog.md`.

Last reviewed: ____-__-__

## How to use this

- One row per physical device. Add a row when you buy something, and update it when you move, reset, or replace it.
- Rows are prefilled with the devices we've discussed. Replace the `___` blanks. Rows marked *example* are placeholders, so duplicate or delete them.
- **Never put passwords, tokens, API keys, or HomeKit setup codes in this file.** Record *where* a secret lives (for example, "password manager: entry name"), not the secret itself. Keep this file out of any public repo.
- Cells marked **(verify)** are things I could not confirm from here.

## 1. Device Inventory

| ID | Device | Make / model | Qty | Room / location | Protocol | Control path | Registered in (app/account) | HA integration | HA entity IDs | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| D-001 | Thermostat | Google Nest, model ___ | 1 | ___ | Google cloud (SDM API) | Cloud | Google account: ___ (personal? **verify**) | Google Nest | `climate.___` | Planned / Active |
| D-002 | Smart plug (*example*) | VOCOlinc, model ___ | ___ | ___ | HomeKit (HAP) over Wi-Fi 2.4GHz | Local | VOCOlinc app, HomeKit mode | HomeKit Controller | `switch.___` | Planned / Active |
| D-003 | Smart plug (*example*) | Lenovo, model ___ | ___ | ___ | Wi-Fi (believed Tuya, **verify**) | Cloud | Lenovo app: ___ / Smart Life: ___ | Smart Life or Tuya | `switch.___` | Planned / Active |
| D-004 | Smart light (*example*) | Lenovo, model ___ | ___ | ___ | Wi-Fi (believed Tuya, **verify**) | Cloud | Lenovo app: ___ / Smart Life: ___ | Smart Life or Tuya | `light.___` | Planned / Active |
| D-005 | Camera (*example*) | Reolink, model ___ | ___ | ___ | LAN (PoE / plugged-in / battery) | Local | Reolink app/account: ___ | Reolink | `camera.___` | Proposed |
| D-006 | NVR / Home Hub (*if used*) | Reolink, model ___ | 1 | ___ | LAN | Local | ___ | Reolink | ___ | Proposed |
| D-007 | HA host | ___ | 1 | ___ | LAN | n/a | n/a | n/a | n/a | Planned |

**Column notes**

- **ID:** stable label (D-001, D-002, ...), so other docs can reference a device without depending on its name.
- **Protocol:** Wi-Fi/Tuya, HomeKit (HAP), Google SDM, LAN/RTSP-style local, Zigbee/Z-Wave/Matter, etc.
- **Control path:** *Local* (works with the internet down) or *Cloud* (depends on a vendor service). For cloud devices, note which service in the dependency table (section 4).
- **HA entity IDs:** fill in after pairing. The custom UI should reference these.
- **Status:** Proposed, Planned, Active, Needs attention, Retired.

## 2. Network and Identity

| ID | Hostname | MAC address | IP (DHCP reservation) | Network / VLAN | Wi-Fi band | Signal / AP | Internet access allowed? |
|---|---|---|---|---|---|---|---|
| D-001 | ___ | ___ | ___ | ___ | 2.4 / 5 / n/a | ___ | Needed (cloud) |
| D-002 | ___ | ___ | ___ | ___ | 2.4 (required for pairing) | ___ | ___ |
| D-003 | ___ | ___ | ___ | ___ | 2.4 (required for pairing) | ___ | Needed (cloud) |
| D-005 | ___ | ___ | ___ | ___ | ___ | ___ | Blockable? (see ADR-003) |
| D-007 | ___ | ___ | ___ | ___ | ___ | ___ | ___ |

Decisions that fill in this table: ADR-003 (segmentation) and ADR-002 (HA host).

## 3. Firmware and Maintenance

| ID | Current firmware | Update method (which app) | Last updated | Next check | Known issues |
|---|---|---|---|---|---|
| D-001 | ___ | Nest / Google Home app | ____-__-__ | ____-__-__ | ___ |
| D-002 | ___ | VOCOlinc app | ____-__-__ | ____-__-__ | ___ |
| D-003 | ___ | Lenovo / Smart Life app | ____-__-__ | ____-__-__ | ___ |
| D-005 | ___ | Reolink app/client | ____-__-__ | ____-__-__ | ___ |
| D-007 | HA OS ___ / Core ___ | HA UI | ____-__-__ | ____-__-__ | ___ |

Cadence decided in ADR-010.

## 4. Service and Account Dependencies

| Service | Used by (IDs) | Account email/ID | Auth type | Secret location (not the secret) | Renewal / expiry to watch | MFA on? |
|---|---|---|---|---|---|---|
| Google Device Access (Nest) | D-001 | ___ | OAuth2 + one-time US$5 registration | ___ | OAuth consent status | ___ |
| Google Cloud project (Nest) | D-001 | ___ | OAuth client ID/secret | ___ | ___ | ___ |
| VOCOlinc app/cloud | D-002 | ___ | App account | ___ | n/a | ___ |
| Lenovo app | D-003, D-004 | ___ | App account | ___ | n/a | ___ |
| Smart Life / Tuya | D-003, D-004 | ___ | App account link (or IoT Core credentials if using the legacy integration) | ___ | Legacy integration: IoT Core subscription can lapse | ___ |
| Reolink | D-005, D-006 | ___ | Device admin password | ___ | n/a | n/a |
| Home Assistant | all | ___ | Admin account; long-lived tokens for custom UI | ___ | Token rotation interval: ___ | ___ |

Decisions that fill in this table: ADR-006 (credentials).

## 5. Pairing and Reset Notes

Record anything you will want if you have to repair a device.

| ID | How it was paired | Setup code location (not the code) | Reset procedure | Gotchas |
|---|---|---|---|---|
| D-002 | VOCOlinc app, HomeKit mode, phone on 2.4GHz | ___ (for example, on the device label or in a password manager) | ___ | Reset or network changes can drop the pairing; re-pair in HA |
| D-003 | ___ | ___ | ___ | Verify it onboards in Smart Life (ADR-008) |
| D-005 | ___ | ___ | ___ | Battery models require the Reolink Home Hub/NVR |

## 6. Acceptance Checklist (per device)

Copy this block for each device and mark pass/fail with the date.

```
Device ID: D-___        Date tested: ____-__-__

[ ] Discovered/added in Home Assistant
[ ] Entities present and named consistently
[ ] On/off or primary control works from HA
[ ] State change made in the vendor app shows up in HA
[ ] Survives HA restart
[ ] Survives router/AP reboot
[ ] Behavior with internet down: works / degrades / fails  (expected per control path)
[ ] Entity IDs recorded in section 1
[ ] Firmware version recorded in section 3
[ ] Included in a tested backup restore (ADR-005)

Notes:
```

## 7. Change Log

| Date | Device ID | Change | Reason |
|---|---|---|---|
| ____-__-__ | ___ | ___ | ___ |
