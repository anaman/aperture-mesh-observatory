# Aperture Mesh Observatory

**A GPS-disciplined, multi-receiver coherent sensing platform for LoRa mesh networks
(MeshCore · Meshtastic · Reticulum).**

Status: **Phase 0 — Proposal & corpus** · Project opened 2026-09-28
Maintainer: Charles Anaman ([@anaman](https://github.com/anaman))

---

## What this is

The Aperture Mesh Observatory turns a small set of inexpensive software-defined radios into a
*coherent radio observatory* for LoRa mesh networks. It is a passive sensing and measurement
platform first, and a path to spatially-aware Linux relays second.

It exists to answer questions that mesh operators currently cannot:

- **Where is a node, physically?** And does its advertised GPS position match its actual RF?
- **Where are the coverage holes**, measured rather than modelled?
- **Who is colliding, and who is jamming?** Which channel is congested, and when?
- **Which direction is that repeater?** (direction of arrival)
- **When did that packet arrive**, to sub-sample precision across *separate* receivers? (long-baseline TDOA)

The long-term prize is a Linux-based relay that understands the *spatial* structure of the channel it
operates in — not just the bytes.

## Why it is possible now

Two prior works (and one enabling piece of hardware already on the bench) make this affordable:

1. **RTL2832u-based coherent multichannel reception** — [YO3IIU, 2014](docs/REFERENCES.md#yo3iiu-2014)
   showed that several €25 RTL-SDR dongles sharing one clock can be made *sample-time* coherent by
   cross-correlation, and aggregated into a wider logical receiver.
2. **Phase-coherent multichannel SDR with sparse arrays** — [Laakso et al., EUSIPCO 2020](docs/REFERENCES.md#laakso-2020)
   added the missing piece for direction finding: a switchable injected reference signal that removes
   the per-retune **carrier-phase** ambiguity, with measured drift below 1°/minute.
3. **A GPS-disciplined reference already on hand** — the Web-888 HF/VHF receiver provides GPS PPS and a
   software-governed Si5351 that can emit a GPSDO-grade reference clock. That converts independent
   receivers into one *time- and frequency-coherent* instrument.

The critical distinction this project is built around:

> **GPS discipline solves time and frequency. Reference injection solves carrier phase. They are not the same thing, and the project needs both.**

## Hardware (current bench)

| Element | Role | Constraint |
|---|---|---|
| 2× RTL-SDR (RTL2838) dongles | LoRa-band sensing (902–928 MHz) | **Receive-only**, ~2.4 MHz each, 8-bit |
| Web-888 network SDR | HF/VHF monitoring **and** GPS time/frequency reference | **Cannot see LoRa** (≈0–61 MHz HF + 118–150 MHz VHF) |
| Spare MeshCore modem | Ground-truth decode, transmit, **cooperative calibration beacon** | The only transmitter |
| Linux host(s) | Capture, DSP, orchestration, relay logic | — |
| GPS/GNSS antenna (Web-888) | Absolute time + disciplined clock | Sky view required |

The intended future upgrade is a common-clock feed to the dongles (28.8 MHz) sourced from the
Web-888, plus a small phase-calibration injection network. See [`docs/HARDWARE.md`](docs/HARDWARE.md).

## Architecture — three planes

```
                   ┌──────────────────────────── REFERENCE PLANE ───────────────────────────┐
                   │  GPS ──PPS──▶ Web-888 ──Si5351──▶ clock-out SMA ──▶ 28.8 MHz ──▶ dongles │
                   │                    (PID-disciplined "poor man's GPSDO")                │
                   └──────────────────────────────────────────────────────────────────────┘
                                                    │  absolute time, stable frequency
                   ┌──────────────────────────── SENSING PLANE ─────────────────────────────┐
                   │   RTL-SDR #0  (USB)        ┐                                            │
                   │   RTL-SDR #1  (rtl_tcp)    ├──▶  capture ──▶ DSP ──▶ RSSI/TDOA/DOA      │
                   │   Web-888     (HF/VHF)     ┘                                            │
                   └──────────────────────────────────────────────────────────────────────┘
                                                    │  events, bearings, timestamps
                   ┌────────────────────────── ACT / TRUTH PLANE ───────────────────────────┐
                   │   MeshCore modem ──▶ decoded ground truth + TX + calibration beacon     │
                   └──────────────────────────────────────────────────────────────────────┘
```

Full detail in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## Roadmap (summary)

| Phase | Goal | Needs |
|---|---|---|
| **0** | Proposal, corpus, repo scaffolding | *this commit* |
| **1** | Mesh observatory: dual-SDR + modem, coverage mapping, interference/duty-cycle analytics, dataset capture | no hardware mods |
| **2** | LoRa PHY demodulation + multi-protocol decode (Meshtastic / MeshCore / Reticulum) | gr-lora_sdr |
| **3** | Time/frequency coherence → absolute-time long-baseline TDOA | Web-888 clock distribution |
| **4** | Carrier-phase coherence → short-baseline bearing; cooperative calibration beacon | reference-injection mod |
| **5** | Applications: DOA-aided relaying, interference nulling, multi-packet reception, sparse-array studies | — |
| **6** | Distributed / transmit-side coherent arrays | *out of current scope; research* |

Full detail in [`docs/ROADMAP.md`](docs/ROADMAP.md).

## Non-goals and safety

- **No transmit beamforming on the RTL-SDRs** — they are receive-only; this is physics, not policy.
- **No jamming, interference, or denial-of-service work**, ever. The wireless interference features are
  *passive detection and reporting* only.
- **Regulatory compliance is mandatory.** Sensing is passive and generally unrestricted; any transmit
  (including the calibration beacon) stays within ISM/EIRP/duty-cycle limits. See
  [`docs/REGULATORY.md`](docs/REGULATORY.md).

## Reproducing this project

Everything needed to replicate the bench, the software chain, and the experiments is documented:
start at [`docs/REPLICATION.md`](docs/REPLICATION.md).

## Documentation index

- [`docs/PROPOSAL.md`](docs/PROPOSAL.md) — the full project proposal
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — system architecture and data flow
- [`docs/HARDWARE.md`](docs/HARDWARE.md) — inventory, clock distribution, calibration hardware
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — phases, milestones, exit criteria
- [`docs/REPLICATION.md`](docs/REPLICATION.md) — how to rebuild from scratch
- [`docs/REGULATORY.md`](docs/REGULATORY.md) — spectrum, power, duty-cycle notes
- [`docs/GLOSSARY.md`](docs/GLOSSARY.md) — terms used throughout
- [`docs/REFERENCES.md`](docs/REFERENCES.md) — annotated, dated source list
- [`DEVLOG.md`](DEVLOG.md) — chronological development log
- [`CHANGELOG.md`](CHANGELOG.md) — released changes

## License

Software: **GPL-3.0-or-later** (see [`LICENSE`](LICENSE)) — chosen to stay compatible with the
lineage of coherent-rtlsdr tooling. Documentation: **CC BY 4.0**. Hardware design files, when added,
will carry an appropriate open-hardware license (proposed CERN-OHL-S v2). *These are proposals open
to change before the first code release.*

## References

See the annotated, dated list in [`docs/REFERENCES.md`](docs/REFERENCES.md). Project-adjacent
lineage:

- YO3IIU, *RTL2832u based coherent multichannel receiver* (2014)
- M. Laakso, V. Rajamäki, R. Wichman, V. Koivunen, *Phase-coherent multichannel SDR — Sparse array
  beamforming* (EUSIPCO 2020); code: `mlaaks/coherent-rtlsdr`
- MeshCore (`meshcore-dev/MeshCore`), Meshtastic, Reticulum / RNode
- Web-888 / RX-888 design notes
