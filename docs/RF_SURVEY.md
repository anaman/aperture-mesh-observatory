# Community RF Survey

The **community RF survey** is the broad-coverage, **roving** layer of the project: a data-collection
system carried or driven through the neighbourhood that scans the radio environment, georeferences what
it hears, and builds a detailed map of the community's RF — sub-GHz spectrum, the LoRa mesh, Wi-Fi and
Bluetooth LE.

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

## 2. Primary mode: roving (wardriving)

The survey is **mobile**: the rig is carried or driven through the neighbourhood with live GNSS, logging
position-tagged measurements. This is the classic wardriving approach and what the project is built
around.

- Best for coverage heat-maps, finding where signals exist at all, and mapping the mesh's real footprint.
- **Motion supplies the spatial diversity a single receiver cannot.** Many positions, each with a
  measurement, combine into a map — which is exactly why one antenna is enough for the *coverage* map,
  even though it is not enough for direction finding.
- Passive throughout; nothing is transmitted except the compliant calibration beacon.

An **optional fixed mode** (the rig parked at a vantage point) can be added later for long-duration
occupancy history and intermittent-emitter detection. It is not required.

### 2.1 The roving rig

| Element | Roving role | Requirement |
|---|---|---|
| RTL-SDR | sub-GHz spectrum sweeps + LoRa band | USB, self-powered hub |
| ESP32 board (spare) | 2.4 GHz Wi-Fi + BLE scanning | ESP32-C6 or ESP32-S3 |
| Heltec V4 (MeshCore) | mesh identity / ground truth | serial to host |
| GNSS | position + time on every record | mobile GNSS (puck, phone, or module) |
| Host | capture, logging, live map | laptop or SBC |
| Power + mounting | run while moving | battery pack, mount |

Practical notes:
- The host must be battery-capable and preferably arm's-length (a laptop is easiest to start; an SBC in a
  bag is tidier).
- Timestamps must be consistent across sensors; use the host clock disciplined by GNSS, or read GNSS time
  per record.
- Mount the antennas clear of the body and the vehicle shell — a poor antenna position silently biases
  every measurement.

### 2.2 Roving procedure

1. Fix on GNSS before starting; confirm time and position are sound.
2. Start the logger (sub-GHz sweep, mesh decode, Wi-Fi/BLE scan) with a single shared timestamp source.
3. Drive or walk the area; vary the route so coverage is not dominated by one corridor.
4. Stop; export; rebuild the map. Repeat runs over time to see change.

### 2.3 Tooling

- Wi-Fi/BLE: **Kismet** with a suitable radio, or an ESP32 scanning firmware; `gpsd` supplies position.
- Sub-GHz: `rtl_power`-style sweeps; `rtl_433` for 433/868/915 MHz consumer devices.
- Data format: WiGLE-style logging is an established, exchangeable format for wardriving runs.

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
