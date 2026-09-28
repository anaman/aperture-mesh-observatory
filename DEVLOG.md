# Development log

Chronological record of the project. Newest entries at the top. Each entry: date, what was decided or
done, what was measured, what failed, what is next.

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
