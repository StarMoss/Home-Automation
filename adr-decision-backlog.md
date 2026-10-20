# Smart Home Control Center: Decision Backlog (ADR Templates)

Companion to `smart-home-control-center-adr.md` (ADR-001, which decides on Home Assistant as the integration layer). Everything below is **still undecided**. Each template has options pre-filled where I could, a blank **Decision** section for you to complete, and a "leaning" note that is my suggestion only, not a decision.

Date created: 2026-10-19

## How to use this

1. Work the decisions roughly in the suggested order below.
2. For each: strike options you reject, fill in **Decision**, **Consequences**, and set **Status** to Accepted.
3. When a decision is made, move it to its own file (`adr-00N-<slug>.md`) if you want the conventional one-file-per-ADR layout.
4. Revisit triggers say when to reopen a decision.

## Index

| ADR | Decision | Status | Blocks | Suggested order |
|---|---|---|---|---|
| 002 | Home Assistant hosting hardware | Open | Everything else that needs a running HA | 1 |
| 003 | IoT network segmentation | Open | Camera placement, mDNS/HomeKit discovery, remote access | 2 |
| 005 | Backup and restore strategy | Open | Safe to add devices and credentials | 3 |
| 006 | Credentials and secrets management | Open | Nest, Tuya, HA token handling | 3 |
| 008 | Lenovo/Tuya integration path | Open (needs a verification test) | Lenovo plug and light setup | 4 |
| 007 | Camera hardware, recording, and storage | Open | Camera purchase | 4 |
| 004 | Remote access method | Open | Using the system away from home | 5 |
| 010 | Update and maintenance policy | Open | Long-term reliability | 5 |
| 011 | Configuration management (config as code) | Open | Reproducibility, documentation | 6 |
| 009 | Custom control center architecture | Open | The custom UI itself | 7 |

Note: IDs are stable labels, not an ordering. ADR-001 is the existing decision.

## Prerequisites to verify before deciding (not ADRs, just facts to check)

- [ ] Nest is on a personal Google account (SDM API is reported incompatible with Google Workspace and Advanced Protection)
- [ ] Lenovo devices: which app are they registered in (Lenovo Link / Lenovo Link Pro), and do they onboard in Smart Life?
- [ ] VOCOlinc plugs are in HomeKit mode on 2.4GHz
- [ ] What router/switch/AP gear you have, and whether it supports VLANs, multiple SSIDs, and mDNS reflection
- [ ] Which Reolink models you are considering, and whether they are PoE, plugged-in, or battery

---

## ADR-002: Home Assistant Hosting Hardware

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
What physical device runs Home Assistant 24/7 at home?

### Context
The MacBook is ruled out (sleeps, leaves the LAN). A cloud VM is ruled out (HomeKit discovery needs the same LAN). Adding cameras may increase CPU, storage, and I/O needs.

### Decision drivers
- [ ] Reliability and low maintenance (appliance-like)
- [ ] Headroom for camera streams/recording (number of cameras: ___)
- [ ] Boot/storage durability (SSD vs SD card)
- [ ] Other self-hosting plans (Pi-hole, media, etc.): yes / no
- [ ] Budget: ___
- [ ] Power draw and noise

### Options
| Option | Pros | Cons |
|---|---|---|
| Home Assistant Green | Purpose-built, minimal setup | Fixed hardware; less headroom (price unverified, about US$100 from memory) |
| Raspberry Pi 4/5 (4GB+), HA OS, SSD | Cheap, widely documented | Tighter with several cameras; you source parts |
| Mini PC (N100-class), HA OS or Docker | More headroom; room for other services | You patch the host OS if using Docker; more power than a Pi |
| NAS with Docker | No new device; storage available | Depends on NAS capability; HA OS features differ in containers |

### Leaning (not a decision)
HA Green or Pi for devices only; mini PC if you expect several cameras or other services. Confidence: moderate, since I haven't measured camera load.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
Camera count grows, HA becomes sluggish, or the host fails.

---

## ADR-003: IoT Network Segmentation

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
Do IoT devices (and cameras) live on the main network, a separate SSID/VLAN, or a mix? How does Home Assistant reach them?

### Context
HomeKit (VOCOlinc) discovery uses mDNS, which does not cross VLANs by default. Nest and Tuya are cloud-mediated. Reolink cameras are local-LAN devices and a sensitive device class. Lenovo/Tuya and VOCOlinc devices use 2.4GHz Wi-Fi for pairing.

### Decision drivers
- [ ] Security isolation of cameras and cheap IoT devices
- [ ] mDNS/HAP discovery must still work
- [ ] Your router/AP capabilities (VLANs, guest network, mDNS reflector): ___
- [ ] Complexity you are willing to maintain
- [ ] HA host placement: which network?

### Options
| Option | Pros | Cons |
|---|---|---|
| A. Single flat network | Simplest; discovery just works | No isolation; cameras and IoT share a network with laptops |
| B. Separate IoT SSID/VLAN, HA on the IoT VLAN | Discovery works inside the VLAN; isolates IoT from personal devices | Your laptop reaching HA requires a firewall rule |
| C. Separate IoT VLAN and cameras VLAN, HA with access to both | Strongest isolation | Needs mDNS reflection/firewall rules; most to maintain |
| D. Guest network for IoT | Easy on consumer routers | Often blocks device-to-device traffic, which can break discovery and HA |

### Leaning (not a decision)
Option B if your gear supports it. Confidence: moderate, depends heavily on your hardware, so verify that first.

### Decision
_TBD_

### Firewall rules to document once decided
- IoT to internet: ___
- HA to IoT/cameras: ___
- Laptop/phone to HA: ___
- Cameras to internet (block? Per HA docs, Reolink works with internet blocked): ___

### Consequences
_TBD_

### Revisit when
You change router/AP hardware, add many cameras, or discovery breaks.

---

## ADR-004: Remote Access Method

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
How do you reach Home Assistant and the custom control center when away from home?

### Context
HA must stay at home. You work from a laptop that moves around. Exposing HA directly to the internet increases risk.

### Decision drivers
- [ ] Security (no open inbound ports if possible)
- [ ] Ease of use from phone and laptop
- [ ] Reliability if your ISP uses CGNAT (unknown: ___)
- [ ] Cost
- [ ] Do other people (household, guests) need access?

### Options
| Option | Pros | Cons |
|---|---|---|
| VPN into home (WireGuard) | Self-controlled; no third-party relay | Requires port forwarding or a reachable address; setup effort |
| Mesh VPN (Tailscale or similar) | Works through CGNAT; easy | Third-party coordination service; check terms and cost |
| Home Assistant Cloud (Nabu Casa) | Simple, supports the project | Subscription; third-party relay; I have not verified current pricing |
| Reverse proxy with TLS + auth, port forwarded | Direct access | Largest attack surface; most upkeep |

### Leaning (not a decision)
VPN or mesh VPN, no public exposure. Confidence: moderate-high on the principle, low on specific product details since I have not researched current terms.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
ISP or router changes, or others need access.

---

## ADR-005: Backup and Restore Strategy

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
What gets backed up, how often, where copies go, how many are kept, and how restores are tested.

### Context
HA holds all device pairings and integration credentials. Losing it can mean re-pairing every HomeKit accessory, redoing the Nest OAuth flow, and relinking Tuya. HA's own backups on its own disk do not protect against hardware failure.

### Decision drivers
- [ ] Recovery time objective (how long can the house be "dumb"?): ___
- [ ] Recovery point objective (how much config loss is acceptable?): ___
- [ ] Backups contain secrets, so storage must be trusted or encrypted
- [ ] Effort you will actually keep up

### Questions to answer
- What is in scope: HA config/database, add-ons, camera recordings (see ADR-007), Reolink/other device configs?
- Frequency: ___ ; retention: ___
- Off-device destination(s): ___ (NAS, cloud storage, another machine)
- Encryption of backups: yes / no; where the key lives: ___
- Who/what verifies backups succeeded (alerting)? ___
- Restore test cadence: ___ (a backup never restored is unproven)

### Options for destination
| Option | Pros | Cons |
|---|---|---|
| Local NAS or another home machine | Fast, free | Same-house risk (fire, theft, power surge) |
| Cloud storage (encrypted) | Off-site | Ongoing cost; secrets leave the house, so encrypt |
| Both (3-2-1 style) | Resilient | More to maintain |

### Leaning (not a decision)
Automatic HA backups plus an encrypted off-site copy, and a documented restore test once. Confidence: high on the principle, no product recommendation made.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
You add devices, change hosting, or a restore test fails.

---

## ADR-006: Credentials and Secrets Management

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
Where credentials live, who can read them, how they are rotated, and how expiry is tracked.

### Context
Secrets in play (inventory to be completed):
- Google Cloud OAuth client ID/secret and Device Access project ID (Nest)
- Nest OAuth tokens (HA refreshes them)
- Tuya/Smart Life account (or IoT Core credentials, depending on integration)
- HomeKit pairing codes and keys (stored by HA)
- Reolink device and NVR/hub admin passwords
- Home Assistant admin account and long-lived access tokens (used by the custom UI)
- VPN keys (ADR-004)
- Router/AP admin credentials

### Decision drivers
- [ ] Separate accounts for IoT services vs your primary Google account? (A dedicated account limits blast radius.)
- [ ] MFA availability per service
- [ ] Who needs access if you are unavailable (bus factor): ___
- [ ] Backups contain secrets (see ADR-005)

### Questions to answer
- Password manager/vault for human-held secrets: ___
- How the custom UI stores its HA token (server-side env var vs browser): ___
- Token scope and rotation interval for HA long-lived tokens: ___
- Tracking expiry/renewal (for example Tuya subscription status if the legacy integration is used): ___
- Secrets must never be committed to Git (see ADR-011): confirm approach for `secrets.yaml`

### Options for storing machine-used secrets
| Option | Pros | Cons |
|---|---|---|
| HA `secrets.yaml` / UI-managed credentials | Native | On the HA host; included in backups |
| Password manager (human-retrieved) | Familiar | Not automatable |
| Secrets manager used from your platform work | Strong tooling | Overkill and a new dependency for a home setup |

### Leaning (not a decision)
Dedicated accounts where possible, password manager for human secrets, server-side token storage for the UI. Confidence: moderate.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
A token leaks, an account provider changes its auth flow, or you add a new integration.

---

## ADR-007: Camera Hardware, Recording, and Storage

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
Which cameras (and whether Reolink at all), how they are powered, and where recordings live.

### Context
Reolink has an official HA integration with local operation and no required cloud (per HA and Reolink docs). Ring was named as the thing being replaced or compared; I have not verified Ring's HA support. Battery-powered Reolink Wi-Fi cameras require a Home Hub/NVR to work with HA. Feature support varies by model.

### Decision drivers
- [ ] Number and locations of cameras: ___
- [ ] Indoor/outdoor; doorbell needed: ___
- [ ] Power: PoE, plugged-in, or battery (battery forces a hub/NVR)
- [ ] Does the property have Ethernet/PoE runs, or can it? ___
- [ ] Retention period for recordings: ___ days
- [ ] Privacy: local-only vs cloud
- [ ] Budget: ___

### Options: vendor
| Option | Pros | Cons |
|---|---|---|
| Reolink | Local operation; official HA integration | Model-specific feature differences; battery models need a hub |
| Ring | (not evaluated) | Cloud-based per my unverified understanding |
| Other local-first brand | (not evaluated) | Needs research |

### Options: recording and storage
| Option | Pros | Cons |
|---|---|---|
| microSD in each camera | Cheap, simple | Limited capacity; wear; retrieval per camera |
| Reolink NVR / Home Hub | Central recording; required for battery cameras | Extra hardware cost |
| Recording to a NAS or HA host disk | Flexible | More load on the host; storage sizing needed |

### Questions to answer
- Does HA need to record, or only show live view and receive events? ___
- Will recordings be included in backups (storage cost)? ___
- Notification handling for detection events: ___

### Leaning (not a decision)
Start with one PoE/plugged-in camera to test the HA integration and measure host load before buying more. Confidence: moderate-high.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
Camera count changes, a model lacks needed features, or storage fills up.

---

## ADR-008: Lenovo / Tuya Integration Path

**Status:** Open (blocked on a verification test)
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
How the Lenovo plugs and lights get into Home Assistant, or whether they are replaced.

### Context
Lenovo devices appear to be Tuya-based, but this is inferred from forum reports (moderate-low confidence). Some users reported failing to onboard Lenovo plugs in the Smart Life app. There is no public Lenovo API.

### Verification test (do this first)
- [ ] Which app are the devices currently in? ___
- [ ] Can one device be added to the Smart Life app? Result: ___
- [ ] Does it appear in HA through the Smart Life or Tuya integration? Result: ___

### Options
| Option | Pros | Cons |
|---|---|---|
| A. Smart Life (Beta) HA integration | Avoids the developer-project workflow | Beta; cloud-dependent; devices may need to be re-added in the Smart Life app |
| B. Legacy Tuya integration | Mature | Depends on an IoT Core subscription that can lapse and needs manual renewal |
| C. Reflash with open firmware (Tasmota/ESPHome via tuya-convert) | Fully local | May void warranty; may not work on newer chips; effort |
| D. Replace with local-first devices | Removes the problem | Cost; e-waste |
| E. Leave in the Lenovo app, outside HA | No work | Defeats the goal of central control |

### Leaning (not a decision)
Run the verification test, then choose A if it works. Confidence: low until the test is done.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
The integration breaks, the cloud service changes, or devices fail.

---

## ADR-009: Custom Control Center Architecture

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
What the custom UI is, where it runs, and how it talks to Home Assistant.

### Context
ADR-001 decides that the UI is a client of Home Assistant's REST/WebSocket API. Not decided: whether a custom UI is even needed beyond HA's built-in dashboards, plus its stack, hosting, and auth.

### First question
- What can the custom UI do that HA's built-in dashboards (including custom cards) cannot? ___
  If the answer is "not much", the right decision may be to skip the custom build or start with HA dashboards.

### Decision drivers
- [ ] Learning/portfolio goals vs getting a working dashboard quickly
- [ ] Which devices/controls it must show: ___
- [ ] Access from phone and laptop, inside and outside the home (ADR-004)
- [ ] Who uses it: just you / household

### Options
| Option | Pros | Cons |
|---|---|---|
| A. HA built-in dashboards | Zero build; maintained | Less custom control |
| B. Custom web app, backend proxy holds the HA token | Token stays server-side; flexible | You host and maintain a second service |
| C. Custom front end calling HA directly from the browser | Simplest | Token in the browser; riskier if exposed beyond the LAN |
| D. Hybrid: HA dashboards plus a small custom app for specific views | Incremental | Two surfaces to maintain |

### Technical questions
- Stack: ___ ; hosting location (HA host, mini PC, other): ___
- Realtime: WebSocket subscription for state changes (preferred over polling)
- Auth for users of the UI: ___
- Error handling when HA is down: ___

### Leaning (not a decision)
Start with A, move to B or D once you know what is missing. Confidence: moderate, and it depends on your goals.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
Dashboards can't meet a specific need, or HA's API changes.

---

## ADR-010: Update and Maintenance Policy

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
How and how often HA core, add-ons, integrations, and device firmware are updated, and how failures are noticed.

### Context
HA releases roughly monthly, and breaking changes are noted in release notes. Device firmware (VOCOlinc, Lenovo, Reolink, Nest) updates through separate apps.

### Questions to answer
- Cadence: monthly / quarterly / on demand: ___
- Pre-update steps (fresh backup, read changelog): ___
- Rollback plan: ___
- Auto-update anything? ___ (for example, add-ons only)
- Who tracks device firmware, and where (the device inventory)? ___
- Monitoring and alerting: how will you know HA is down or an integration is failing? ___ (for example, notification on unavailable entities, uptime check)
- Cameras: firmware update check cadence: ___

### Options
| Option | Pros | Cons |
|---|---|---|
| Update on release | Latest fixes | More chances of breakage |
| Quarterly batch | Predictable | Lag behind security fixes |
| Pin versions, update deliberately | Stability | Falls behind if neglected |

### Leaning (not a decision)
Quarterly batch with a pre-update backup, with a sooner patch for security issues. Confidence: moderate.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
An update causes an outage, or you add safety-critical automations.

---

## ADR-011: Configuration Management (Config as Code)

**Status:** Open
**Date:** 2026-10-19
**Owner:** TBD

### Decision needed
Whether and how Home Assistant configuration, dashboards, and automations are version-controlled and documented.

### Context
HA configuration can be edited in the UI or managed as YAML files. UI-managed config lives in HA's storage and is captured by backups, but is not diffable. Documentation (device inventory, runbooks) needs a home.

### Decision drivers
- [ ] Comfort with YAML-based management vs the UI
- [ ] Wanting reviewable history of changes
- [ ] Secrets must stay out of Git (see ADR-006)
- [ ] Where docs live (this ADR set, device inventory, runbooks): ___

### Options
| Option | Pros | Cons |
|---|---|---|
| UI-managed only, backups as the source of truth | Lowest effort | No change history; hard to review |
| YAML for automations/dashboards in a Git repo; integrations via UI | Reviewable history; fits your engineering habits | Some config remains UI-only; sync discipline needed |
| Fully YAML-managed where possible | Most reproducible | More work; HA is moving some config to the UI |

### Questions to answer
- Repo location (private): ___
- What goes in the repo: ADRs, device inventory, runbooks, HA YAML (without secrets)
- How config reaches the HA host (file sync, add-on, manual): ___

### Leaning (not a decision)
A private Git repo for docs and automations/dashboards, integrations through the UI. Confidence: moderate.

### Decision
_TBD_

### Consequences
_TBD_

### Revisit when
Config grows complex, or a change breaks something and you can't tell what changed.
