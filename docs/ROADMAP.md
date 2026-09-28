# Roadmap

Phases are ordered so that each one is useful on its own, and each one de-risks the next. Nothing later
is attempted until the earlier coherence claim is demonstrated and documented.

## Phase 0 — Proposal and corpus *(current)*

**Deliverables**
- This repository: proposal, architecture, hardware, roadmap, references, devlog.
- An annotated, dated source list with honest assessments.

**Exit criteria:** repo published; scope, non-goals and regulatory posture stated.

---

## Phase 1 — Mesh observatory *(no hardware modification required)*

**Goal:** a passive, ground-truthed picture of the mesh's physical layer using the two dongles plus the
modem.

**Capabilities**
- Unified capture from both dongles plus modem decode, timestamped.
- Two-site RSSI proximity/localisation keyed to decoded node identities.
- Coverage and hole mapping.
- Collision and interference detection (energy the modem does not decode).
- Duty-cycle measurement per channel.
- Synchronised dataset writer (IQ + decode log) for later research.

**Exit criteria:** a live view that shows, for real traffic, which node is near which receiver, which
channels are busy, and where energy appears without a decodable packet.

**Risks:** capture continuity; timestamp alignment between streams.

---

## Phase 2 — PHY demodulation and multi-protocol decode

**Goal:** decode LoRa PHY in software so one physical modem can support several monitored protocols.

**Work**
- Build GNU Radio 3.10 and integrate `gr-lora_sdr`.
- Channel plan for Meshtastic (906.875 MHz), MeshCore (910.525 MHz), Reticulum (911.0 MHz).
- Protocol framing: Meshtastic (protobuf, default key), MeshCore framing, Reticulum framing.

**Exit criteria:** concurrent decoding of at least two protocols from the SDRs, cross-checked against the
modem's own decode log.

**Risks:** real-time DSP load; sensitivity versus a native transceiver.

---

## Phase 3 — Time/frequency coherence → long-baseline TDOA

**Goal:** absolute-time timestamping across separated receivers for passive localisation.

**Work**
- Confirm the Web-888 clock-out can be set to 28.8 MHz; distribute it to the dongles.
- Read the GPS PPS counter for software timestamping.
- Implement arrival-time estimation and TDOA geometry.
- Validate against the cooperative calibration beacon at known positions.

**Exit criteria:** measured position error versus known beacon positions, reported honestly (including
failures).

**Risks:** clock distribution feasibility; phase noise; multipath bias.

---

## Phase 4 — Carrier-phase coherence → short-baseline bearing

**Goal:** direction of arrival from a compact coherent array.

**Work**
- Build the switchable reference-injection calibration network.
- Implement the Laakso-style phase alignment; characterise drift and recalibration cadence.
- Demonstrate bearing estimation on real LoRa preambles; test sparse geometries when more than two
  elements are available.
- Use the MeshCore modem as a known-position calibration beacon.

**Exit criteria:** a bearing estimate for a real transmitter, with the front/back ambiguity characterised
and a stated accuracy.

**Risks:** two elements is the theoretical floor; multipath dominates in cluttered environments.

---

## Phase 5 — Applications

**Goal:** turn measurements into relay improvements.

- Direction-of-arrival-guided relay decisions and directional antenna steering (transmit through the
  modem).
- Interference nulling / spatial filtering to hear a weak node in the presence of a strong one.
- Multi-packet reception research (separating concurrent transmitters).
- Sparse-array geometry studies to reduce per-relay cost.

**Exit criteria:** at least one demonstrated improvement to relay reliability, measured and documented.

---

## Phase 6 — Documented future research (out of current scope)

- Distributed coherent arrays across nodes ("cell-free" style) — needs cross-node sub-symbol time and
  carrier-phase synchronisation.
- Transmit-side coherent arrays — needs phase-coherent transmit silicon the current hardware lacks.

These are recorded so the project's boundaries are explicit, not to promise them.

---

## Cross-cutting requirements (every phase)

- **Document the failures**, not only the successes.
- **Log every experiment** in [`DEVLOG.md`](../DEVLOG.md) with date, hypothesis, method, result.
- **Verify sources** for age and provenance before relying on them.
- **Stay compliant** with spectrum rules (see [`REGULATORY.md`](REGULATORY.md)).
