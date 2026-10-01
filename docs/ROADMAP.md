# Roadmap

Phases are ordered so that each one is useful on its own, and each one de-risks the next. Nothing later is
attempted until the earlier claim is demonstrated and documented. Phases are logical groupings, not a
strict schedule.

## Phase 0 — Proposal and corpus *(complete, 2026-09-28)*

- Repository published: proposal, architecture, hardware, roadmap, references, devlog.

**Exit criteria:** ✅ repo published; scope, non-goals and regulatory posture stated.

---

## Phase 1 — Receiver bring-up and LoRa observatory core

**Goal:** a passive, ground-truthed picture of the LoRa mesh's physical layer using the RTL-SDR plus the
Heltec modem.

**Capabilities**
- Verify each receiver; confirm the Heltec modem decodes MeshCore from Linux (ground truth).
- Timestamped capture; confirm stream continuity.
- Channelise to the channel plan (MeshCore 910.525 MHz, Meshtastic 906.875 MHz, Reticulum 911.0 MHz US).
- Correlate modem decode log with SDR observations; detect energy the modem does not decode.
- Duty-cycle and congestion measurement per channel.

**Exit criteria:** a live view showing, for real traffic, which nodes are heard, which channels are busy,
and where energy appears without a decodable packet.

**Risks:** capture continuity; sensitivity versus a native transceiver.

---

## Phase 2 — Community RF survey *(new, 2026-09-28)*

**Goal:** a detailed, georeferenced map of the local RF environment, **roving (wardriving) first**.

**Capabilities**
- Roving rig: RTL-SDR sub-GHz spectrum occupancy sweeps (24–1766 MHz) with GNSS position per record.
- LoRa mesh overlay: decoded identity versus measured RSSI versus advertised position.
- Wi-Fi 2.4 GHz and Bluetooth LE observations from a spare ESP32 board (identifiers hashed).
- HF/VHF occupancy from the Web-888 (fixed reference/anchor).
- Position-tagged signal-strength heat-maps; overlay of all layers on one map.

**Exit criteria:** a georeferenced map, built from measured data, showing coverage, holes and hot spots,
with personal identifiers hashed and not published.

**Risks:** privacy/legal handling of third-party signals; the 5 GHz gap; consistency of roving position.

See [`RF_SURVEY.md`](RF_SURVEY.md).

---

## Phase 3 — PHY demodulation and multi-protocol decode

**Goal:** decode LoRa PHY in software so the observatory is not limited to the modem's protocol view.

**Work**
- Build GNU Radio 3.10 and integrate `gr-lora_sdr`.
- Protocol framing: Meshtastic (protobuf, default key), MeshCore framing, Reticulum framing.

**Exit criteria:** concurrent decoding of at least two protocols from the SDR, cross-checked against the
modem's own decode log.

**Risks:** real-time DSP load; sensitivity versus a native transceiver.

---

## Phase 4 — Time/frequency coherence → long-baseline TDOA

**Goal:** absolute-time timestamping across separated receivers for passive localisation.

**Work**
- Confirm the Web-888 clock-out can supply the receiver clock; distribute it.
- Read the GPS PPS counter for software timestamping.
- Implement arrival-time estimation and TDOA geometry with a second receiver at a separate site.

> **Note:** with a single RTL-SDR this phase requires at least one additional receiver to become a
> *second site*. Until then it can be developed against synthetic and replayed data.

**Exit criteria:** measured position error versus known beacon positions, reported honestly.

**Risks:** clock distribution feasibility; phase noise; multipath bias; needing a second receiver.

---

## Phase 5 — Carrier-phase coherence → short-baseline bearing

**Goal:** direction of arrival from a compact coherent array.

**Work**
- Grow to two or more coherent receivers; build the switchable reference-injection calibration network.
- Implement the Laakso-style phase alignment; characterise drift and recalibration cadence.
- Demonstrate bearing estimation on real LoRa preambles.
- Validate against the Heltec modem as a known-position calibration beacon.

**Exit criteria:** a bearing estimate for a real transmitter, with the front/back ambiguity characterised
and a stated accuracy.

**Risks:** two elements is the theoretical floor; multipath dominates in cluttered environments.

---

## Phase 6 — Applications

**Goal:** turn measurements into relay improvements.

- Direction-of-arrival-guided relay decisions and directional antenna steering (transmit through the
  modem).
- Interference nulling / spatial filtering to hear a weak node in the presence of a strong one.
- Multi-packet reception research (separating concurrent transmitters).
- Sparse-array geometry studies to reduce per-relay cost.

**Exit criteria:** at least one demonstrated improvement to relay reliability, measured and documented.

---

## Phase 7 — Documented future research (out of current scope)

- Distributed coherent arrays across nodes ("cell-free" style) — needs cross-node sub-symbol time and
  carrier-phase synchronisation.
- Transmit-side coherent arrays — needs phase-coherent transmit silicon the current hardware lacks.

Recorded so the project's boundaries are explicit, not to promise them.

---

## Cross-cutting requirements (every phase)

- **Document the failures**, not only the successes.
- **Log every experiment** in [`DEVLOG.md`](../DEVLOG.md) with date, hypothesis, method, result.
- **Verify sources** for age and provenance before relying on them.
- **Hash personal identifiers** and never publish household-identifying data.
- **Stay compliant** with spectrum and privacy rules (see [`REGULATORY.md`](REGULATORY.md)).
