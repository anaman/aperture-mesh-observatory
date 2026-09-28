# Community RF Survey

The **community RF survey** is the broad-coverage layer of the project: a data-collection system that
scans the radio neighbourhood, georeferences what it hears, and builds a detailed map of the community's
RF environment — sub-GHz spectrum, the LoRa mesh, Wi-Fi and Bluetooth LE.

It complements the high-precision coherent core (bearing and TDOA). The survey answers *what is out
there, and roughly where*; the coherent core answers *exactly where, and from which direction*.

## 1. Goal

Produce a detailed, georeferenced map of the local RF environment:

- **Spectrum occupancy** by band across ≈24–1766 MHz (sub-GHz) and HF/VHF.
- **LoRa mesh overlay** — which MeshCore/Meshtastic/Reticulum nodes are heard, with what signal strength,
  and whether their advertised position matches their RF footprint.
- **Wi-Fi** — 2.4 GHz networks observed passively (SSID/BSSID, channel, signal).
- **Bluetooth LE** — BLE advertisements observed passively.
- **Signal-strength heat-maps** to reveal coverage, holes and hot spots.

## 2. Two modes

The same instrument supports both; a survey can be run fixed, roving, or both.

- **Fixed ("the wardrobe").** The rig parked at a vantage point (a home installation) running
  continuously. Best for long-duration occupancy history, duty-cycle measurement, and detecting
  intermittent or distant emitters.
- **Roving.** The rig carried or driven through the neighbourhood with live GNSS, recording position-
  tagged measurements. Best for coverage heat-maps and finding where signals exist at all.

> Both are passive. Nothing is transmitted except the compliant calibration beacon.

## 3. Sensing layers and which hardware supplies each

| Layer | Source | Notes |
|---|---|---|
| Sub-GHz spectrum sweep (24–1766 MHz) | RTL-SDR with a wideband power sweep | e.g. `rtl_power`-style FFT scan |
| LoRa mesh ident/position | Heltec V4 (MeshCore) + RTL-SDR | decoded identity vs measured RSSI |
| HF / VHF spectrum | Web-888 | 0–61 MHz + 118–150 MHz |
| Wi-Fi 2.4 GHz | ESP32-C6 / ESP32-S3 | passive beacon/probe observation |
| Bluetooth LE | ESP32-C6 / ESP32-S3 | passive advertisement observation |
| 802.15.4 (Thread/Zigbee) | ESP32-C6 | optional |
| Position / time | GNSS (fixed base + roving) | georeferencing + absolute time |
| **5 GHz Wi-Fi** | *none yet* | gap — needs additional hardware |

## 4. Data model

Every observation is one row of a georeferenced record:

```
timestamp (absolute, UTC)   position (lat, lon, alt)   band / channel
layer (spectrum|mesh|wifi|ble)   identifier (hashed where personal)   RSSI / SNR   confidence
```

Rules:
- Identifiers that can point at a person or household (Wi-Fi BSSIDs, BLE MACs) are **hashed on ingest**;
  raw addresses are not published.
- Every row carries its position, time and the configuration that produced it.
- Captures are append-only and versioned so maps can be rebuilt.

## 5. The "omni antenna array" — what is and is not possible

An *array of omnidirectional antennas* only delivers array processing if the receivers behind it are
**coherent** (see [`ARCHITECTURE.md`](ARCHITECTURE.md)). With the current hardware:

- **One RTL-SDR** → you can connect a set of omnidirectional antennas through an **RF switch** and select
  between them. This gives *switched diversity*, not coherent combining: useful for characterising
  multipath and for picking the strongest element, but **not** for direction finding.
- **Roving with one antenna** → the *motion* provides the spatial diversity; position-tagged RSSI across
  many locations builds the coverage map. This is the classic and effective wardriving approach.
- **True direction finding / beamforming** requires **two or more phase-coherent receivers** with a shared
  clock and phase calibration. That is the project's Phase 4/5 path and a future hardware increment — not
  something one dongle can do.

So the practical plan is: **use motion + position-tagged RSSI for the map now**, and keep the coherent
array as the precision upgrade later.

## 6. Reuse before building (preflight)

The project prefers existing, maintained tools over custom code where they are adequate:

- **Sub-GHz spectrum:** `rtl_power` / `rtl_fft`-style sweeps from an RTL-SDR.
- **Wi-Fi / BLE / 802.15.4 survey:** **Kismet** (with suitable radios) or the ESP32 scanning firmware
  approach; **WiGLE**-style logging as an established data format for wardriving.
- **Sub-GHz device decoding:** `rtl_433` for 433/868/915 MHz consumer devices, where relevant.
- **Position:** `gpsd` as the standard GNSS daemon.
- **All-in-one environments:** distributions such as DragonOS bundle many of these.
- **Mesh mapping:** existing Meshtastic/MeshCore map tooling for node overlays.

Custom code is written only for what is genuinely new here: the unified georeferenced data model, the
LoRa mesh overlay, and the fusion with the coherent core.

## 7. Outputs

- Interactive RF heat-maps (signal strength / occupancy over geography).
- Per-band occupancy and duty-cycle charts over time.
- A mesh overlay: node identity, measured RSSI, and advertised position.
- A Wi-Fi/BLE observation layer (identifiers hashed).

## 8. Privacy, ethics and legal

Mapping a *neighbourhood* means observing other people's signals. The project treats this deliberately:

- **Passive only.** Listen; never associate with networks, never transmit on them, never capture payloads.
- **Hash personal identifiers** (Wi-Fi BSSIDs, BLE MACs) at ingest; do not publish raw addresses.
- **Do not publish anything that identifies a household or person** — publish aggregates and heat-maps,
  not device lists.
- **BLE is a tracking risk.** Advertisements can reveal device presence and movement; scan passively and
  aggregate.
- Follow local law on radio reception and on personal data; in Europe this touches GDPR-style obligations.
- **No jamming, no deauthing, no interference** — the interference features are detection only.

Details in [`REGULATORY.md`](REGULATORY.md).
