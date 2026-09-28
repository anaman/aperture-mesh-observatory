# Project Proposal — Aperture Mesh Observatory

**A GPS-disciplined, multi-receiver coherent sensing platform for LoRa mesh networks**

- **Project name:** Aperture Mesh Observatory
- **Date opened:** 2026-09-28
- **Status:** Proposal (Phase 0)
- **Maintainer:** Charles Anaman ([@anaman](https://github.com/anaman))
- **License:** Software GPL-3.0-or-later · Documentation CC BY 4.0 *(proposed)*

---

## 1. Summary

The Aperture Mesh Observatory is an open hardware and software project that assembles a small number
of inexpensive software-defined radios into a **coherent radio observatory** for LoRa mesh networks,
beginning with **MeshCore**. It measures the physical layer that mesh protocols currently hide: node
location and bearing, true coverage, collisions, interference, and the absolute arrival time of
packets across separated receivers.

The project has a strict ordering. It is **passive sensing first** (nothing is transmitted except a
deliberately compliant calibration beacon), **open replication second** (every experiment documented
so others can rebuild it), and **spatially-aware Linux relays third** (using the resulting measurements
to improve how relays propagate and forward).

## 2. Motivation

LoRa mesh networks — MeshCore, Meshtastic, and Reticulum-over-LoRa — are spreading into off-grid and
community use. Their software is well understood, but their **physical layer is nearly invisible** to
operators. Concretely, an operator cannot currently answer:

- Is the position a node advertises where it actually is?
- A repeater claims coverage — is that coverage real, measured, and where are the holes?
- A link degrades. Is that path loss, multipath, a collision, or an interferer/jammer?
- Which physical direction should a directional relay antenna point?
- When several nodes transmit at once, which one actually got through, and when?

A single receiver cannot answer these. Answering them requires **multiple receivers whose measurements
can be combined coherently** — in time, frequency, and (for direction finding) carrier phase. That is
what this project builds.

## 3. Background and prior art (honest assessment)

### 3.1 RTL2832u coherent multichannel receiver — YO3IIU (2014)

An experiment write-up (personal blog, not peer-reviewed) demonstrating several RTL2832u dongles fed
from **one clock source** (a low-jitter clock distribution board), with residual **sample-time** offsets
measured by cross-correlating the channels and removed numerically. Demonstrated applications include
**bandwidth aggregation** over non-contiguous spectrum and frequency-hopping reception. Notable
practical findings: a self-powered USB hub is required (≈185 mA per dongle), USB 2.0 can carry roughly
ten 2 Msps 8-bit streams, and **clocking several dongles from a single dongle worked only up to three
receivers** — the fourth broke synchronisation, which is why a dedicated clock buffer is needed.

*Assessment:* foundational and reproducible, but qualitative; it solves sample-timing coherence and
assumes "no samples are lost" rather than proving it. It does **not** address carrier-phase ambiguity.

### 3.2 Phase-coherent multichannel SDR — Laakso, Rajamäki, Wichman & Koivunen (EUSIPCO 2020)

A 7-channel coherent RTL-SDR system (~€150 in parts) using a common clock plus a **switchable injected
reference-noise source** to calibrate each channel's phase, with measured drift below 1°/minute. It
demonstrates sparse-array (Minimum-Redundancy / Boundary array) direction-of-arrival estimation using
difference co-arrays on real captured data, and argues that sparse arrays reach full-array resolution
with fewer radio front ends.

*Assessment:* a strong engineering contribution (a cheap coherent platform) plus a real-data sanity
check of sparse-array theory — but the results are qualitative (no error metric or Cramér–Rao
comparison), measured in a reflective lecture hall with a single static source, ideal-array assumptions,
and a tuner-silicon dependency. The authors themselves note the co-array method is **multipath-sensitive**
and **not suitable for transmit beamforming**.

### 3.3 The enabling asset: a GPS-disciplined reference (Web-888)

The Web-888 network SDR includes a GPS/GNSS module with **PPS**, and the ADC clock is produced by a
software-governed **Si5351** synthesiser that a PID loop disciplines from PPS (the manufacturer's own
term: *"poor man's GPSDO"*). The same governed PLL can drive a **clock-out SMA**, and a reference
**clock-in** option also exists. This is a time- and frequency-reference source that can discipline the
whole receiver set. Manufacturer's caveat: enabling clock-out **degrades the Si5351 phase noise**, so it
is disabled by default.

### 3.4 The synthesis

- **YO3IIU** lacked absolute time → guessed alignment by correlation.
- **Laakso** had a common clock but still needed per-measurement phase calibration.
- **GPS discipline** replaces the guesswork with an absolute time/frequency base, but still does **not**
  fix the per-retune carrier-phase ambiguity — so Laakso-style reference injection remains necessary for
  direction finding.

Therefore the project is built on two distinct capability regimes that must not be conflated:

| Regime | Physical requirement | What it enables |
|---|---|---|
| **Long baseline** (receivers metres–kilometres apart) | time/frequency coherence only | TDOA positioning using absolute arrival time |
| **Short baseline** (receivers centimetres–metres apart) | carrier-phase coherence | bearing / direction of arrival |

## 4. Objectives

1. **Build a mesh observatory** that passively monitors LoRa mesh traffic and produces a live,
   ground-truthed picture of coverage, location (proximity and, later, bearing), collisions and
   interference.
2. **Build a community RF survey instrument** — a fixed and/or roving data-collection rig that maps
   sub-GHz spectrum occupancy, the LoRa mesh, 2.4 GHz Wi-Fi and Bluetooth LE over a neighbourhood,
   georeferenced by GNSS.
3. **Achieve time/frequency coherence** across receivers using the GPS-disciplined reference, and use it
   for long-baseline TDOA.
4. **Achieve carrier-phase coherence** using a reference-injection calibration network, and use it for
   short-baseline direction of arrival.
5. **Use the Heltec V4 MeshCore modem as a cooperative calibration beacon**, transmitting compliant known
   signals from known positions to calibrate the array.
6. **Document everything** — bench, software, experiments, failures — so the work is fully replicable.
7. **Apply the results to relaying**: feed spatial awareness back into Linux-based relay behaviour.

## 5. Approach — three planes

- **Reference plane.** GPS → PPS → Web-888 Si5351 → disciplined clock-out → (future) 28.8 MHz feed to
  the dongles. Supplies absolute time and stable frequency to the whole system.
- **Sensing plane.** The RTL-SDR for sub-GHz (LoRa band and spectrum sweeps), the Web-888 for HF/VHF, and
  a spare ESP32 board for 2.4 GHz Wi-Fi/BLE survey work. Capture, then process to RSSI, arrival time and
  (later) bearing.
- **Act / truth plane.** The spare MeshCore modem decodes real packets (ground truth), transmits the
  calibration beacon, and eventually acts as the radio interface of a spatially-aware relay.

Detailed architecture: [`ARCHITECTURE.md`](ARCHITECTURE.md). Hardware: [`HARDWARE.md`](HARDWARE.md).

## 6. Phased plan (summary)

**Phase 0 — Proposal & corpus (now).** Publish this proposal, the annotated source list, and the
repository scaffolding.

**Phase 1 — LoRa observatory core.** RTL-SDR capture plus Heltec modem ground truth; unified live view;
channel plan; interference, collision and duty-cycle analytics (passive only); synchronised dataset
capture.

**Phase 2 — Community RF survey.** Fixed and/or roving georeferenced mapping of sub-GHz spectrum, the LoRa
mesh, 2.4 GHz Wi-Fi and Bluetooth LE; heat-maps, occupancy history, mesh overlay. See
[`RF_SURVEY.md`](RF_SURVEY.md).

**Phase 3 — PHY demodulation & multi-protocol decode.** Integrate GNU Radio `gr-lora_sdr`; decode
Meshtastic, MeshCore and Reticulum framing.

**Phase 4 — Time/frequency coherence.** Distribute the Web-888 disciplined clock; establish absolute-time
timestamping; implement long-baseline TDOA between separated receivers (needs a second receiver).

**Phase 5 — Carrier-phase coherence.** Build the reference-injection phase-calibration network; run the
Laakso-style phase-alignment procedure; demonstrate short-baseline bearing, validated by the Heltec modem
as a known-position beacon.

**Phase 6 — Applications.** DOA-aided relaying; interference nulling; multi-packet reception research;
sparse-array geometry studies.

**Phase 7 — Out of scope, documented as future research.** Distributed and transmit-side coherent arrays.

Exit criteria and milestones are in [`ROADMAP.md`](ROADMAP.md).

## 7. Limitations and risks (stated up front)

- **RTL-SDR is receive-only.** No transmit beamforming is possible on this hardware, and the project will
  not pretend otherwise.
- **Phase noise versus frequency accuracy.** GPS discipline gives excellent long-term frequency/time but
  not necessarily the spectral purity needed as the *sole* phase reference. A dedicated GPS-disciplined
  oscillator may be required for tight direction finding — a future purchase decision, not an assumption.
- **Two elements is the theoretical floor** for an array: a bearing with front/back ambiguity and poor
  resolution. Resolution improves with more elements or a second baseline.
- **Multipath.** Both source works flag it, and real mesh environments are maximally multipath. Diversity
  combining *helps*; direction finding *suffers*.
- **Dynamic range.** 8-bit samples and per-board gain differences limit performance; this is inherited
  from the hardware lineage.
- **Demodulation cost.** Real-time LoRa demodulation on a small Linux host is the computational wall;
  direction finding on preambles is cheaper than full demodulation.
- **Clock distribution details are unverified.** Whether the Web-888 clock-out can be set to 28.8 MHz and
  used independently of the ADC clock must be confirmed against firmware before Phase 3 commits to it.

## 8. Non-goals

- No transmit beamforming on receive-only hardware.
- No jamming, interference generation, or denial-of-service research.
- No interception of encrypted payloads beyond what a normal network participant can decode; the aim is
  physical-layer measurement, not message content.
- No claims of precision beyond what is measured and reported.

## 9. Regulatory posture

Sensing is passive and generally permissible. Transmissions (the calibration beacon) must stay within
ISM EIRP and duty-cycle limits, and any direction-finding/relay work on amateur bands must follow
amateur rules (identification, no encryption, minimum necessary power). Details and citations in
[`REGULATORY.md`](REGULATORY.md).

## 10. Reproducibility and documentation

This repository is the project's memory. Development is logged chronologically in
[`DEVLOG.md`](../DEVLOG.md); released changes are tracked in [`CHANGELOG.md`](../CHANGELOG.md); all source
lives under [`src/`](../src/); and the rebuild procedure is in [`REPLICATION.md`](REPLICATION.md).
Every referenced source is dated and classified in [`REFERENCES.md`](REFERENCES.md), consistent with a
practice of verifying the age and provenance of sources before relying on them.

## 11. Open questions

**Resolved (2026-09-28):** one RTL-SDR dongle; one Web-888; one Heltec V4 MeshCore modem.

Still open:
1. Can the Web-888 clock-out be configured to 28.8 MHz, and is it independent of the 122.88 MHz ADC clock?
2. Is a spare ESP32 board (ESP32-C6 / ESP32-S3) available for Wi-Fi/BLE survey work, and what is the exact
   ESP32-S3 board model?
3. Is the RF survey primarily **fixed** (a parked installation) or **roving** (carried/driven), or both?
4. For a second receiver (needed for TDOA and for a bearing array), is inter-building or inter-site
   spacing intended?
5. Which 5 GHz Wi-Fi coverage, if any, is wanted — noting the current hardware cannot reach 5 GHz.

## 12. References

See [`REFERENCES.md`](REFERENCES.md) for the annotated, dated source list.
