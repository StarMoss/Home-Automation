# ADR-001: Centralized Smart Home Control Center

| | |
|---|---|
| **Status** | Proposed |
| **Date** | 2026-10-19 |
| **Decision** | Use Home Assistant (self-hosted, always-on, on the home LAN) as the integration layer; build any custom control center against its REST/WebSocket API |

---

## 1. Context

Devices in the home today:

| Device | Protocol / ecosystem | Vendor API? |
|---|---|---|
| Google Nest thermostat | Google cloud, Smart Device Management (SDM) API | Yes, official |
| VOCOlinc smart plugs | HomeKit Accessory Protocol (HAP), local | No public cloud API; HAP is the interface |
| Lenovo smart plugs and lights | Believed to be rebadged Tuya ("Powered by Tuya") | No public Lenovo API (per Lenovo support, as reported on forums); Tuya Cloud API instead |

**Goal:** manage all devices from one place, ideally with a custom control center and programmatic access.

**Core problem:** there is no universal discovery or auth mechanism across these ecosystems. Each has its own onboarding flow, credential model, and schema.

## 2. Decision Drivers

- One API surface for the custom UI, not three vendor integrations
- Prefer local control where possible (reliability, latency, privacy)
- Low ongoing maintenance; this should behave like an appliance
- Avoid writing and maintaining OAuth/pairing/session code per vendor
- The primary dev machine (MacBook) is not always on or at home

## 3. Options Considered

### Option A: Integrate each vendor API directly
- Nest: SDM REST + OAuth2 + Pub/Sub
- VOCOlinc: HAP pairing and encrypted sessions over the LAN
- Lenovo: Tuya Cloud IoT API
- **Pros:** full control, no extra hub dependency
- **Cons:** three auth flows, three schemas, three failure modes to own; HAP session handling is non-trivial

### Option B: Home Assistant as the integration layer (chosen)
- Existing, maintained integrations for all three ecosystems handle auth/pairing
- Normalizes devices into entities (`climate.*`, `switch.*`, `light.*`) with `state` + `attributes`
- Exposes a unified local REST API and WebSocket API
- **Pros:** one API and one token for the custom UI; large community; local-first where the device allows
- **Cons:** adds a always-on dependency; abstraction can hide vendor-specific detail

### Option C: Homebridge / OpenHAB
- Viable alternatives; not evaluated in depth. Homebridge is HomeKit-centric, which fits VOCOlinc but is a weaker fit as a general API for a custom dashboard.
- Revisit only if Home Assistant proves unsuitable.

### Option D: Apple Home (or another vendor app) as the hub
- Easy for HomeKit devices, but no open programmatic API for a custom control center, and Tuya/Nest coverage is incomplete without bridges.

## 4. Decision

Adopt **Option B**. Home Assistant becomes the single integration point. The custom control center is a client of Home Assistant, not of the individual vendors.

## 5. Integration Design

### 5.1 Per-device path

| Device | HA integration | Auth / discovery | Local or cloud |
|---|---|---|---|
| Nest thermostat | Google Nest (SDM) | Register Device Access project (one-time US$5), enable SDM API in Google Cloud, OAuth2. Devices are tied to the Google account, so there is no LAN discovery. HA stores and refreshes tokens. | Cloud |
| VOCOlinc plugs | HomeKit Controller | mDNS/Bonjour discovery (`_hap._tcp`), then HAP pairing with the 8-digit setup code. Plugs must be in HomeKit mode. | Local |
| Lenovo plugs/lights | Tuya or Smart Life | Link the app account (QR scan in Smart Life app) and sync devices from Tuya cloud | Cloud (local control only via custom firmware) |

### 5.2 Layering

```
Custom control center (UI)
        |  REST + WebSocket, single long-lived token
        v
   Home Assistant  <-- entity normalization (domain, state, attributes)
   |        |        |
 Nest SDM   HAP     Tuya cloud
 (OAuth2)  (LAN)    (account link)
```

### 5.3 Custom control center API usage
- **REST:** `GET /api/states`, `POST /api/services/<domain>/<service>`
- **WebSocket:** subscribe to state-change events for a live dashboard (preferred over polling)
- **Auth:** HA long-lived access token; keep it out of the browser if the UI is exposed beyond the LAN

## 6. Hosting Decision

The MacBook is rejected as host: it sleeps, travels, and leaves the LAN. A cloud VM is also rejected: HomeKit discovery and pairing need the same L2 network (mDNS), so HA must run at home.

| Option | Notes | Recommendation |
|---|---|---|
| Home Assistant Green (about US$100, price unverified) | Purpose-built appliance, minimal setup | **Default pick** |
| Raspberry Pi 4/5 (4GB+), HA OS, SSD preferred over SD | Cheap, best-documented path, low power | Equally good |
| Mini PC (N100-class) running HA in Docker | More headroom if a homelab grows; you also patch the host OS | If other self-hosting is planned |
| NAS with Docker | One less device if you already own one | Fine if available |

## 7. Operational Overhead (estimates)

- **Initial setup:** a few hours (hardware, pairing three integrations, first dashboard)
- **Steady state:** roughly 20-30 minutes per month (updates, checking integrations)
- **Updates:** monthly releases; quarterly is workable, read the changelog for breaking changes
- **Backups:** enable automatic HA backups and copy them off the device. Not yet configured, so treat this as a required task.
- **Failure mode if neglected:** stale integrations and missing updates, not an outage of paired devices

## 8. Consequences

**Positive**
- One API, one schema, one credential for the custom UI
- Local control for HomeKit devices
- Large community and integration ecosystem for future devices

**Negative / costs**
- New always-on device to run and back up
- HA becomes a single point of failure for the control center (devices still work in their own apps)
- Entity abstraction can drop vendor-specific detail (for example, Nest traits); raw attributes may be needed for exotic cases
- Cloud dependence remains for Nest and Tuya regardless of the hub

## 9. Risks and Open Questions

| Item | Risk / question | Confidence | Next step |
|---|---|---|---|
| Lenovo = Tuya | Inferred from forum reports; some users failed to onboard Lenovo plugs in Smart Life | **Moderate-low** | Confirm which app your devices use; test one device in Smart Life before committing |
| Tuya integration variant | Legacy Tuya integration depends on an IoT Core subscription that can lapse; the newer Smart Life (Beta) integration avoids the developer-project workflow | Moderate | Prefer Smart Life integration; check current docs before setup |
| Nest account type | SDM API is reported incompatible with Google Workspace accounts and Advanced Protection | Moderate-high | Confirm the Nest is on a personal Google account |
| Nest scope | SDM covers thermostats/cameras/doorbells, not every Nest product; camera streaming is limited | High | Thermostat control only here, so likely fine |
| VOCOlinc mode | Must be set up in HomeKit mode and on 2.4GHz Wi-Fi; may need an Apple home hub for remote HomeKit control (not needed for local HA control) | Moderate | Verify mode in the VOCOlinc app |
| Custom firmware | `tuya-convert`/Tasmota/ESPHome can give local Lenovo control but may void warranty and may not work on newer chips | Low | Only if cloud dependence becomes unacceptable |
| HA Green price and availability | From memory, not verified | Low | Check current price |

## 10. Implementation Plan

1. Procure and flash HA hardware; assign a DHCP reservation
2. Pair VOCOlinc plugs via HomeKit Controller (validate mDNS visibility on the LAN)
3. Link Nest via Device Access (register, create project, OAuth consent)
4. Verify one Lenovo device in Smart Life, then add the Tuya/Smart Life integration
5. Configure backups and off-device copies
6. Generate a long-lived token; prototype the custom UI against `/api/states` and the WebSocket event stream
7. Decide on remote access (VPN such as WireGuard or Tailscale rather than exposing HA publicly)

## 11. References

- Google Device Access: https://developers.google.com/nest/device-access
- SDM API overview: https://developers.google.com/nest/device-access/api
- Home Assistant Nest integration: https://www.home-assistant.io/integrations/nest/
- Tuya Smart Life integration for HA: https://github.com/tuya/tuya-smart-life
- Home Assistant REST/WebSocket API docs: https://developers.home-assistant.io/docs/api/rest/
