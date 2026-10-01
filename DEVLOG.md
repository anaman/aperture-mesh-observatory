# Development log

Chronological record of the project. Newest entries at the top. Each entry: date, what was decided or
done, what was measured, what failed, what is next.

---

## 2026-10-01 — Project principle set: listen-first, licence-gated transmit

**Decided (Charles)**
- The project must remain fully usable by an operator **without a ham licence** — they may **listen**
  (passive sensing works for everyone) but must **not transmit** on any licence-requiring frequency until
  they hold the licence. The operator bears responsibility for every transmission.

**Documented**
- [`REGULATORY.md`](docs/REGULATORY.md) rewritten: a listen-first principle, a **per-band transmit gate**
  table, and the implementation (a single declaration holding `transmit_enabled`, licence type and
  callsign; receive-only until declared; band-aware transmit paths).
- Guardrails added to [`FUTURE_SCOPE.md`](docs/FUTURE_SCOPE.md) and [`ROADMAP.md`](docs/ROADMAP.md); `README.md`
  safety section updated.

**Design note**
- Transmit is gated **per band**, not globally: ISM bands run within power/duty limits without a licence;
  amateur bands stay locked until a callsign is declared. The gate is a configuration declaration, not a
  hidden switch — it is logged and visible.

---

## 2026-10-01 — Future scope recorded: Winlink/B2F and a distributed node grid

**Requested by Charles (future work)**
- Integrate the platform with **Winlink B2F**.
- Use **detected ESP32 devices** as listening/transmitting tools: pick up and decode the three main LoRa
  data (MeshCore, Meshtastic, Reticulum) and, if the operator wishes, respond — with a visual of the
  direction the message came from.

**Researched**
- Read the *Open B2F* spec in the browser (the site blocks automated fetches). B2F = message structure +
  forwarding between Winlink RMS gateways and client programs; transports are Pactor/WINMOR/ARDOP/VARA/
  Robust Packet/AX.25/Telnet; messages compress as FBB B1; header Type includes **Position Report**;
  FCC §97.309 permits listening.

**Recorded (new document: [`docs/FUTURE_SCOPE.md`](docs/FUTURE_SCOPE.md))**
- Winlink integration = a **bridge** through an existing B2F client (PAT), not a reimplementation. LoRa is
  *not* a Winlink transport. **Hardware gap:** Winlink RF needs a transmit radio the bench lacks.
  **Legal:** Winlink RF is amateur radio — licence required, no business content, no encryption.
- Distributed grid is scoped to **our own / explicitly authorized** devices only; commandeering
  third-party devices is excluded (unauthorized access and transmission).
- Decode is passive and permitted; payloads only where we hold the key. "Respond" = transmit, allowed
  only as an authorized participant, to traffic addressed to us.
- **Direction visual** needs ≥2 coherent receivers (Phase 5) or a multi-node RSSI grid; one receiver
  cannot give a bearing.

**Next**
- Phase 1 bring-up unchanged; these remain future phases.

---

## 2026-10-01 — Roving (wardriving) confirmed as the primary survey mode

**Confirmed**
- The community RF survey is **mobile**: wardriving, not a parked installation. (Charles clarified
  "wardrobe" was a mis-transcription of "wardriving".)

**Changed**
- [`RF_SURVEY.md`](docs/RF_SURVEY.md): roving is now the primary mode; added the roving rig (hardware it
  carries), a roving procedure, and the tooling list. Fixed mode demoted to an optional later addition.
- [`ROADMAP.md`](docs/ROADMAP.md) Phase 2 restated as "roving (wardriving) first"; exit criteria note the
  roving rig and a mobile GNSS dependency.
- `README.md` and `PROPOSAL.md` updated to match.

**Note**
- Motion is what makes a single-receiver coverage map work: it supplies the spatial diversity one antenna
  cannot. Direction finding still needs ≥2 coherent receivers — unchanged.

---

## 2026-09-28 — Bench confirmed; community RF survey added

**Confirmed**
- Licenses accepted: code GPL-3.0, docs CC BY 4.0; repository public.
- Equipment finalised: **1× RTL-SDR**, **1× Web-888**, **1× Heltec V4** MeshCore modem. *(Not two RTL
  dongles — the original proposal's two-dongle assumption is corrected.)*

**Decided**
- Added the **community RF survey** as a first-class capability (now Phase 2): a fixed and/or roving,
  georeferenced data-collection rig mapping sub-GHz spectrum, the LoRa mesh, 2.4 GHz Wi-Fi and Bluetooth
  LE. New document: [`docs/RF_SURVEY.md`](docs/RF_SURVEY.md).
- Renumbered the roadmap to Phases 0–7 to insert the survey.
- Recorded the survey hardware reality honestly:
  - RTL-SDR tops out ≈1.766 GHz → **cannot reach 2.4 GHz** (no Wi-Fi/BLE).
  - **One receiver is not an array** — switched antennas + motion give coverage mapping, not direction
    finding; bearings need ≥2 coherent receivers.
  - The Heltec's ESP32-S3 offers 2.4 GHz Wi-Fi/BLE **only when not running MeshCore firmware** → use a
    spare ESP32 board for survey work.
  - **5 GHz Wi-Fi is a hard gap**; roving needs its own GNSS.
- Adopted "reuse before build" for the survey layer: `rtl_power`, Kismet, WiGLE-style logging, `gpsd`,
  `rtl_433`, DragonOS where adequate.

**Privacy / ethics**
- Wi-Fi BSSIDs and BLE MACs to be hashed at ingest; no household-identifying data published. Passive
  observation only — no association, no payload capture, no jamming.

**Next**
- Phase 1 bring-up: verify the RTL-SDR; confirm Heltec MeshCore decode from Linux; prove capture
  continuity; build the channel plan.

---

## 2026-09-28 — Project opened (Phase 0)

**Decided**
- Named the project **Aperture Mesh Observatory** and opened this repository.
- Scoped it as passive sensing first, open replication second, spatially-aware Linux relays third.
- Defined the core distinction the whole design rests on: **GPS discipline solves time/frequency;
  reference injection solves carrier phase.** Both are needed, for different capabilities.
- Settled the three-plane architecture (reference / sensing / act-truth).

**Reviewed (sources)**
- YO3IIU (2014) — RTL2832u coherent multichannel receiver: sample-time coherence by correlation,
  bandwidth aggregation; limited to ~3 receivers on a single clock without a distribution buffer.
- Laakso, Rajamäki, Wichman & Koivunen (EUSIPCO 2020) — phase-coherent multichannel SDR with injected
  reference calibration; sparse-array direction finding on real data; results qualitative and
  multipath-sensitive.
- Web-888 manufacturer design notes — GPS/PPS, Si5351 "poor man's GPSDO", clock-out/clock-in, phase-noise
  caveat, band limits (cannot see 902–928 MHz).

**Findings / open risks**
- The Web-888 cannot sense the LoRa band; it is the reference and HF/VHF monitor only.
- Two elements is the theoretical floor for an array: bearing with front/back ambiguity.
- Whether the Web-888 clock-out can be set to 28.8 MHz independently of the ADC clock is unverified.

**Next**
- Confirm bench inventory (one or two RTL dongles in play).
- Begin Phase 1 bring-up: verify each receiver, establish modem ground truth, confirm stream continuity.
