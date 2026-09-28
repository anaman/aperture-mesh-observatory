# Hardware

The bench described here is deliberately modest: commodity receivers, one GPS-disciplined reference, and
one protocol-aware modem. The point is that coherence is a *technique*, not expensive equipment.

> **Privacy note:** this document describes hardware and its capabilities only. No network addresses,
> hostnames, keys or private infrastructure details are published here.

## 1. Current bench

### 1.1 RTL-SDR dongles (×2)

- Chipset: RTL2838 (RTL2832u demodulator + R820T-class tuner).
- Native clock: **28.8 MHz**.
- Coverage: usable through the 902–928 MHz US ISM band.
- Bandwidth: ≈2.4 MHz instantaneous; **8-bit** samples.
- **Receive-only.**
- Bus current: ≈185 mA each when tuned (R820T), so use a self-powered USB hub.
- One dongle is used directly over USB; the other is exposed over the network via `rtl_tcp`, so it can
  physically sit somewhere else (a different room, a different building, a remote site).

### 1.2 Web-888 network SDR

- Origin: RX-888 lineage; single-board, plug-and-play, web interface (Alpine Linux, read-only root).
- HF: ≈0–61 MHz. VHF: second Nyquist-zone channel with a 118–150 MHz band-pass filter and +20 dB LNA.
- ADC: LTC2208, up to 130 Msps; 16-bit.
- Front end: LNA + step attenuator + low-pass filter, plus a 0.5 ppm TCXO.
- Digital: Xilinx Zynq XC7Z010 (FPGA + dual-core ARM); 13 DDC channels at 12 kHz plus two spectrum
  channels; 1 GbE.
- **GNSS module** (BDS/GPS/GLONASS/GALILEO) with **PPS**.
- **Si5351** clock synthesiser governs the ADC clock and can drive the **clock-out SMA**; a reference
  **clock-in** option also exists. Clock-out is disabled by default because it degrades phase noise.
- **Cannot receive the 902–928 MHz LoRa band.** It is the reference and the HF/VHF monitor, not a LoRa
  sensor.

### 1.3 Spare MeshCore modem

- The only transmitter and the only protocol-aware radio.
- Roles: ground-truth decode, calibration beacon (compliant, low duty cycle), later the radio front end
  of a spatially-aware relay.
- Drivable from the Linux host (serial/BLE/USB depending on the specific device).

### 1.4 Linux host(s)

- Run capture, DSP, fusion, and the live view; one host may also host the modem and one dongle, with the
  other dongle and (optionally) further receivers reachable over the network.

## 2. Clock distribution (planned)

The reference plane must deliver one stable clock to all dongles.

```
Web-888 (GPS/PPS-disciplined Si5351)
        │  clock-out SMA
        ▼
  clock buffer / divider  ──▶ 28.8 MHz ──▶ RTL-SDR #0 clock-in
                          ──▶ 28.8 MHz ──▶ RTL-SDR #1 clock-in
```

Notes and unknowns:

- The dongles want **28.8 MHz**. Since the Si5351 is programmable, a second output *may* be settable to
  28.8 MHz; this must be confirmed against firmware before committing.
- Prior art used a dedicated low-jitter clock distribution board; a single dongle's clock output was only
  good for about three receivers before the fourth broke synchronisation. Use a proper buffer.
- A GPS-disciplined oscillator with low phase noise may be added later if direction-finding demands it.

## 3. Phase calibration (planned, Phase 4)

Following the Laakso method: a **switchable injected reference signal** is fed into every channel. With
the reference on, the relative channel phases are measured and corrected; with it off, the array observes
the real environment. Implementation options range from a small noise/reference generator with a
switchable splitter to a purpose-built injection board.

The **MeshCore modem doubling as a cooperative calibration beacon** provides an independent check: a
known signal from a known position, transmitted on command.

## 4. Antennas and siting

- Short-baseline direction finding needs elements spaced on the order of a fraction of a wavelength at
  915 MHz (≈33 cm per wavelength); a compact array or a small ground plane works.
- Long-baseline TDOA needs separated sites with GPS-disciplined timing and a clear sky view for the
  reference receiver.
- Diversity reception benefits from *spatially decorrelated* antennas, even coarsely separated.

## 5. Bill-of-materials sketch

| Item | Qty | Purpose |
|---|---|---|
| RTL-SDR dongle (RTL2838) | 2 | LoRa-band sensing |
| Web-888 network SDR | 1 | GPS reference + HF/VHF monitoring |
| GNSS antenna | 1 | Reference timing |
| MeshCore modem | 1 | Truth, TX, calibration beacon |
| Self-powered USB hub | 1 | Power the dongles |
| Linux host(s) | 1–2 | Capture, DSP, fusion |
| Clock buffer / divider (planned) | 1 | Distribute 28.8 MHz |
| Reference injection network (planned) | 1 | Phase calibration |

Prices are deliberately omitted; the project prefers to describe capability and let builders source
locally.

## 6. Verified versus assumed

- **Verified** (from the manufacturer's design notes and the existing bench): GPS/PPS presence, Si5351
  governance, clock-out/clock-in availability, Web-888 band limits, dongle composition and reachability.
- **Assumed, to confirm:** that the Web-888 clock-out can be set to 28.8 MHz independently of the ADC
  clock, and the exact dongle clock-injection point for the specific boards in use.
