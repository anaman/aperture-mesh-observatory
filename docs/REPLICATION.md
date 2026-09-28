# Replication guide

How to rebuild the Aperture Mesh Observatory on your own bench. This is written so a careful stranger can
follow it without asking questions.

## 0. What you are building

A receiver set that produces **coherent** measurements of a LoRa mesh: coverage, proximity, (later)
bearing, collisions and interference, timestamped against GPS. See [`PROPOSAL.md`](PROPOSAL.md) and
[`ARCHITECTURE.md`](ARCHITECTURE.md) for the why.

## 1. Minimum bench

- 2× RTL-SDR dongles (RTL2838 / R820T-class).
- 1× Web-888 network SDR with its GNSS module and a GNSS antenna (sky view).
- 1× MeshCore-capable modem (or any LoRa transceiver you can drive from Linux).
- 1× self-powered USB hub.
- 1–2 Linux hosts.

Phase 1 needs **no modification** to any radio. Later phases add a clock distribution network and a
reference-injection network.

## 2. Software prerequisites

- Linux with the rtl-sdr blog driver build and udev rules for the dongles.
- `rtl_tcp` to expose a dongle over the network.
- Python 3 for the capture/measure/serve pipeline.
- (Phase 2) GNU Radio 3.10 plus the `gr-lora_sdr` out-of-tree module.
- (Later) DSP and array-processing libraries of your choice.

## 3. Bring-up order

1. **Verify each receiver alone.** Confirm each dongle tunes and streams; confirm the Web-888 web
   interface and its GPS/PPS status.
2. **Establish ground truth.** Get the MeshCore modem decoding real traffic from Linux.
3. **Time-stamp everything.** Read the Web-888 GPS PPS counter and confirm your captures carry absolute
   time.
4. **Capture both dongles.** One on USB, one over `rtl_tcp`. Confirm stream continuity (no dropped
   samples).
5. **Channelise** to the channel plan (Meshtastic 906.875 MHz, MeshCore 910.525 MHz, Reticulum 911.0 MHz
   in the US).
6. **Correlate with the modem's decode log** to label what the SDRs see.
7. **Measure**: RSSI, coverage, collisions, interference, duty cycle.
8. **Persist**: write synchronised IQ + decode log for offline work.

## 4. Adding coherence (later phases)

### 4.1 Time/frequency
- Configure the Web-888 clock-out; confirm the target frequency for your dongles (28.8 MHz) and that it is
  independent of the ADC clock.
- Distribute through a proper clock buffer — a single dongle's clock output degraded past ~3 receivers in
  prior art.
- Feed each dongle's clock-in point; confirm the modification with a known signal before trusting it.

### 4.2 Carrier phase
- Build a switchable reference-injection network feeding every channel.
- With the reference on, measure and correct inter-channel phase.
- With the reference off, observe the environment.
- Characterise drift; recalibrate on a schedule and after every retune.
- Validate bearings against the cooperative calibration beacon at known positions.

## 5. What to record (mandatory)

For every experiment, write in [`DEVLOG.md`](../DEVLOG.md): date, hardware configuration, hypothesis,
method, raw result, interpretation, and what failed. Reproducibility depends on the failures being
written down as clearly as the successes.

## 6. What NOT to do

- Do not transmit except with the compliant calibration beacon.
- Do not operate on amateur bands without a licence, and never with encryption or obscured messages.
- Do not attempt direction finding on private communications; the goal is physical-layer measurement.

## 7. Known unknowns a replicator will hit

- The exact clock-injection point for your specific dongle board.
- Whether the Web-888 clock-out can be set to your dongle's clock frequency.
- Gain and phase mismatch between individual dongles (calibrate per board).
- Host DSP load when demodulating in real time.
