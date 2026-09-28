# Hardware

The bench described here is deliberately modest: commodity receivers, one GPS-disciplined reference, and
one protocol-aware modem. The point is that coherence is a *technique*, not expensive equipment.

> **Privacy note:** this document describes hardware and its capabilities only. No network addresses,
> hostnames, keys or private infrastructure details are published here.

## 1. Current bench *(confirmed 2026-09-28)*

### 1.1 RTL-SDR dongle (×1)

- Chipset: RTL2838 (RTL2832u demodulator + R820T-class tuner).
- Coverage: **≈24–1766 MHz**. This reaches the 902–928 MHz LoRa band, the 433/868 MHz device bands, and
  the 2 m/70 cm amateur bands — but **not 2.4 GHz**, so it cannot see Wi-Fi or Bluetooth LE.
- Bandwidth: ≈2.4 MHz instantaneous; **8-bit** samples.
- Native clock: **28.8 MHz**.
- **Receive-only.**
- Bus current: ≈185 mA tuned (R820T) — use a self-powered USB hub.
- **One receiver cannot form an antenna array.** See [`RF_SURVEY.md`](RF_SURVEY.md) §5 for what an
  "omni antenna array" can and cannot do with a single channel.

### 1.2 Web-888 network SDR (×1)

- Origin: RX-888 lineage; single-board, plug-and-play, web interface (Alpine Linux, read-only root).
- HF: ≈0–61 MHz. VHF: second Nyquist-zone channel with a 118–150 MHz band-pass filter and +20 dB LNA.
- ADC: LTC2208, up to 130 Msps; 16-bit.
- Front end: LNA + step attenuator + low-pass filter, plus a 0.5 ppm TCXO.
- Digital: Xilinx Zynq XC7Z010 (FPGA + dual-core ARM); 13 DDC channels at 12 kHz plus two spectrum
  channels; 1 GbE.
- **GNSS module** (BDS/GPS/GLONASS/GALILEO) with **PPS**.
- **Si5351** clock synthesiser governs the ADC clock and can drive the **clock-out SMA**; a reference
  **clock-in** option also exists. Clock-out is disabled by default because it degrades phase noise.
- **Cannot receive the 902–928 MHz LoRa band, nor 2.4 GHz.** It is the reference, the HF/VHF monitor, and
  (ideally) a fixed installation — not a LoRa or Wi-Fi sensor.

### 1.3 Heltec V4 — MeshCore modem (×1)

- MCU: ESP32-S3 (dual-core, Wi-Fi + BLE 5) with a Semtech **SX1262** sub-GHz LoRa transceiver.
- Roles:
  - **MeshCore modem** — the only transmitter and the only protocol-aware radio; ground-truth decode,
    calibration beacon, later the radio front end of a spatially-aware relay.
  - **2.4 GHz Wi-Fi + BLE radio** — the ESP32-S3 can scan 2.4 GHz Wi-Fi and BLE.
- **Constraint:** while the unit is flashed with MeshCore firmware, its Wi-Fi/BLE is not a general-purpose
  scanner. To use the ESP32 for Wi-Fi/BLE surveying, either run custom firmware (losing the MeshCore role
  on that unit) or use a **separate** ESP32 board. Use the separate board.

### 1.4 Supporting boards (2.4 GHz survey sensors)

- **Waveshare ESP32-C6-LCD-1.47** — 2.4 GHz Wi-Fi 6, BLE 5, 802.15.4 (Thread/Zigbee) plus a small display.
  Ideal dedicated Wi-Fi/BLE/802.15.4 survey sensor and status display.
- **ESP32-S3 board** (exact model TBC) — 2.4 GHz Wi-Fi + BLE; alternative survey sensor.
- Both are **2.4 GHz only** — neither reaches 5 GHz Wi-Fi.

### 1.5 Positioning

- The Web-888 GNSS is at the **fixed** installation; it is the time/frequency reference and a survey
  anchor.
- A **roving** survey needs its own position source: a USB GNSS puck, a phone, or a GNSS module on the
  survey MCU.

### 1.6 Compute

- Linux host(s) run capture, DSP, fusion and the live view; one host may host the RTL-SDR and the modem,
  with the Web-888 and any additional sensors reachable over the network.

## 2. What the confirmed suite can and cannot sense

| Band / service | Sensor | Status |
|---|---|---|
| HF 0–61 MHz | Web-888 | ✅ |
| VHF 118–150 MHz | Web-888 | ✅ |
| Sub-GHz ≈24–1766 MHz (LoRa 902–928, 433, 868, 2 m, 70 cm) | RTL-SDR | ✅ |
| LoRa mesh (MeshCore/Meshtastic/Reticulum) decode | Heltec modem (+ SDR) | ✅ |
| Wi-Fi 2.4 GHz | ESP32-C6 / ESP32-S3 | ✅ (needs a spare board) |
| Bluetooth LE | ESP32-C6 / ESP32-S3 | ✅ (needs a spare board) |
| **Wi-Fi 5 GHz** | — | ❌ gap |
| **Sub-GHz spectrum heat-mapping** | RTL-SDR (`rtl_power`) | ✅ |
| **Directions of arrival** | needs ≥2 coherent receivers | ❌ with one RTL-SDR |

## 3. Clock distribution (planned)

```
Web-888 (GPS/PPS-disciplined Si5351)
        │  clock-out SMA
        ▼
  clock buffer / divider  ──▶ 28.8 MHz ──▶ RTL-SDR clock-in
```

- The dongle wants **28.8 MHz**; whether the Web-888 clock-out can supply it independently of the
  122.88 MHz ADC clock must be confirmed against firmware.
- Prior art: a single dongle's clock output was only good for ~3 receivers before the fourth broke
  synchronisation — use a proper buffer if the array ever grows.

## 4. Phase calibration (planned, Phase 5)

Following the Laakso method: a **switchable injected reference signal** fed into every coherent channel so
relative phases can be measured and corrected. With only one receiver today, this is dormant until the
array grows. The Heltec modem doubling as a **cooperative calibration beacon** gives a known signal from a
known position for validation.

## 5. Bill-of-materials sketch

| Item | Qty | Purpose |
|---|---|---|
| RTL-SDR dongle (RTL2838) | 1 | Sub-GHz sensing, spectrum sweeps, LoRa band |
| Web-888 network SDR | 1 | GPS reference + HF/VHF monitoring (fixed) |
| GNSS antenna | 1 | Reference timing |
| Heltec V4 (MeshCore) | 1 | Truth, TX, calibration beacon |
| ESP32-C6 / ESP32-S3 board | 1+ | 2.4 GHz Wi-Fi / BLE / 802.15.4 survey |
| Mobile GNSS (puck/phone/module) | 1 | Roving survey position |
| Self-powered USB hub | 1 | Power the dongle |
| Portable power + enclosure | 1 | Roving survey rig |
| Linux host | 1 | Capture, DSP, fusion |
| Clock buffer (planned) | 1 | Distribute 28.8 MHz (if the array grows) |
| Reference injection network (planned) | 1 | Phase calibration (if the array grows) |

Prices are deliberately omitted; the project describes capability and lets builders source locally.

## 6. Verified versus assumed

- **Verified:** RTL-SDR band limit (no 2.4 GHz); Web-888 band limits, GPS/PPS, Si5351 governance and
  clock-out/clock-in availability; Heltec V4 = ESP32-S3 + SX1262.
- **Assumed, to confirm:** whether the Web-888 clock-out can be set to 28.8 MHz independently of the ADC
  clock; whether a spare ESP32 board is available for Wi-Fi/BLE survey work; the exact ESP32-S3 board
  model.
