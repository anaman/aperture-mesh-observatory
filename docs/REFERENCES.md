# References

Annotated and dated. Each entry notes what it is, when it is from, and how much weight to give it —
sources are verified for age and provenance before being relied upon.

---

## Primary works this project builds on

### — RTL2832u coherent multichannel receiver (@yo3iiu, 2014)
- **Type:** personal technical blog / experiment write-up (not peer-reviewed)
- **URL:** https://yo3iiu.ro/blog/?p=1450
- **Date:** circa mid-2014 (article image paths dated 2014/06)
- **What it shows:** several RTL2832u dongles fed from a single clock source; residual **sample-time**
  offsets found by cross-correlation and removed numerically; bandwidth aggregation over non-contiguous
  spectrum; frequency-hopping reception.
- **Useful engineering facts:** ≈185 mA per dongle; USB 2.0 carries ≈10 streams of 2 Msps 8-bit;
  clocking several dongles from one dongle worked only to ~3 receivers.
- **Assessment:** foundational and reproducible, but qualitative; solves sample-timing coherence only and
  assumes "no samples lost." Does not address carrier phase.
- **Its own references:** `ltl.tkk.fi/~plahteen/else/works/rtl/sticks.pdf` (Aalto/TKK),
  `superkuh.com/rtlsdr.html`, a SARA-list Google Groups thread.
- **Provenance note:** Ref 1 is from the Aalto/TKK group — the same lineage as the 2020 paper below.

### — Phase-coherent multichannel SDR — Sparse array beamforming (Laakso et al., 2020)
- **Authors:** M. Laakso, V. Rajamäki, R. Wichman, V. Koivunen
- **Venue:** EUSIPCO 2020 (conference paper, ~5 pages)
- **Code / artifacts:** https://github.com/mlaaks/coherent-rtlsdr (GPL-3.0)
- **What it shows:** a ~7-channel coherent RTL-SDR receiver (~€150 in parts) using a common clock plus a
  **switchable injected reference noise source** for per-channel **carrier-phase** calibration; drift
  below 1°/min; sparse-array (Minimum-Redundancy / Boundary) direction finding via difference co-arrays on
  real data.
- **Assessment:** strong engineering (a cheap coherent platform) and a real-data sanity check of
  sparse-array theory. Results are *qualitative* (no RMSE or Cramér–Rao comparison), measured in a
  reflective lecture hall with one static source; ideal-array assumptions; tuner-silicon dependency. The
  authors note the co-array method is multipath-sensitive and unsuitable for transmit beamforming.
- **Related work by the same group:** M. Laakso, *"Multichannel coherent receiver on the RTL-SDR"* (M.Sc.
  thesis); Laakso et al., *"Near-field localization using machine learning: an empirical study"* (2021),
  https://research.aalto.fi/files/67797446/Laakso_empirical_near_field_localization.pdf

### — Underlying algorithms (context)
- **MUSIC** (Schmidt, 1986) — subspace direction-of-arrival estimation.
- **Difference co-array / covariance matrix augmentation** (Pillai et al., 1985 and later) — sparse-array
  processing; the paper notes the augmented covariance is not guaranteed positive-definite, which
  complicates source-number estimation.

---

## Target network

### — MeshCore
- **Repo:** https://github.com/meshcore-dev/MeshCore
- **Description:** a lightweight, hybrid routing/flooding mesh protocol for packet radios (LoRa). Node
  roles include companion, repeater and room server.
- **Design notes / philosophy:** Ripple Radios (@ripplebiz) posts, e.g.
  https://buymeacoffee.com/ripplebiz/meshcore-philosophy (Mar 2025) and
  https://buymeacoffee.com/ripplebiz/some-big-news (Jan 2025).
- **Architecture overview:** https://mesh101.com/mesh-tech/architecture/
- **Community wiki:** https://forum.letsmesh.net/t/meshcore-key-concepts/30 (Aug 2025);
  MeshCore GitHub wiki.

### — Related mesh protocols (multi-protocol decode targets)
- **Meshtastic** — https://meshtastic.org/docs/ (device list, `meshtasticd`, protocol)
- **Reticulum / RNode** — https://github.com/markqvist/Reticulum

---

## Enabling hardware

### — Web-888 / RX-888 design notes (manufacturer)
- **URL:** https://www.rx-888.com/web/design/ (read 2026-09-28)
- **Key facts used:** 0–60 MHz HF plus a second-Nyquist VHF channel (118–150 MHz); LTC2208 ADC up to
  130 Msps; Xilinx Zynq XC7Z010; Si5351 clock synthesiser governing the ADC clock; **GNSS module with
  PPS**; a PID-disciplined "poor man's GPSDO" driving the Si5351; **clock-out SMA** and **clock-in**
  option; clock-out degrades phase noise and is disabled by default; GPS PPS counter readable.
- **Downloads / firmware:** https://www.rx-888.com/web/downloads.html
- **Assessment:** manufacturer documentation — authoritative for capabilities, but the clock-out
  frequency/independence claims still need confirmation against firmware before Phase 3 relies on them.

### — GPS-disciplined references (context for a possible Phase 4 purchase)
- GNSS-disciplined oscillators with one or more RF outputs (e.g. Leo Bodnar LBE-1425 class) are the
  fallback if the Web-888 clock-out proves too noisy for tight direction finding. Recorded as an option,
  not a recommendation.

---

## Project-adjacent (prior work in this workspace)

These are the local artifacts the bench grew out of; they are summarised here because they informed the
proposal. Their contents are not republished.
- `sdr-decode` suite — receive + decode of LoRa broadcasts over the 902–928 MHz band from a USB SDR and
  a network SDR, merged into one live view.
- Device inventory (2026-09-05) — hardware capability mapping, including the Web-888 band limits.
- Community mesh options research (2026-08-30) — the broader Wisconsin-region mesh landscape (AREDN,
  Meshtastic/MSPMesh, WIDigitalNetwork), which frames the community use case.

---

## Verification practice

For every claim taken from an external source, the project records: the URL, the publication or fetch
date, whether it is peer-reviewed, and an explicit assessment of its limits. Sources whose dates do not
match the events they describe, or whose provenance cannot be established, are not relied upon.
