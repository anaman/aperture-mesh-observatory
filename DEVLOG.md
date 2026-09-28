# Development log

Chronological record of the project. Newest entries at the top. Each entry: date, what was decided or
done, what was measured, what failed, what is next.

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
