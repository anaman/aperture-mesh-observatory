# Regulatory posture

Passive reception is generally permissible and unregulated in most jurisdictions. The only transmissions
the project makes are deliberate, bounded, and gated by the operator's licence status — see the project
principle below.

> This is engineering guidance, not legal advice. Verify against your own national and local rules.

---

## 1. Project principle — listen-first, licence-gated transmit

The project is built so that **an operator without any radio licence can use the full sensing capability**
— survey, mapping, spectrum awareness, mesh decode — because all of it is passive. Transmitting is a
separate, gated capability.

- **Default state is receive-only.** Nothing transmits until the operator explicitly enables it.
- **Listening is permitted** to the amateur and ISM bands this project uses, without a licence, in most
  jurisdictions. (A few countries restrict reception of certain services; the operator is responsible for
  local rules.)
- **Transmit is decided per band**, not globally:
  - **Unlicensed ISM bands** (e.g. 902–928 MHz and 2.4 GHz in the US) — transmitting is permitted within
    power and duty-cycle limits.
  - **Licence-requiring bands** (amateur HF/VHF/UHF, etc.) — **locked** unless the operator declares a
    valid licence and callsign.
- **The operator bears responsibility for every transmission.** The project documents what each band
  requires; it does not, and cannot, make the legal decision for the operator.

### 1.1 Implementation of the gate

- A single configuration holds the operator's declaration: `transmit_enabled`, the licence type, and the
  callsign.
- With no declaration, all transmit paths are disabled and the public record shows a receive-only node.
- Enabling transmit on a licence-requiring band requires the declaration **and** a matching callsign; the
  action is written to the log.
- Transmit paths are band-aware: ISM paths may run within limits without a licence; amateur paths stay
  locked without one.

---

## 2. Per-band transmit gate

| Band / service | Listen | Transmit requires |
|---|---|---|
| US ISM 902–928 MHz (MeshCore / Meshtastic / Reticulum LoRa) | ✅ no licence | Unlicensed, within power/duty limits |
| EU ISM 868 MHz | ✅ no licence | Unlicensed, within duty-cycle limits |
| 2.4 GHz ISM (Wi-Fi / BLE) | ✅ no licence | Unlicensed — but **do not associate with or transmit on networks you do not own** |
| HF / VHF / UHF amateur (Winlink transports, APRS, D-STAR, packet) | ✅ generally permitted | **Amateur licence + ID + no encryption + no business content** |
| Anything else | Verify first | Verify first |

---

## 3. Reception (the bulk of the project)

- Passive monitoring of ISM bands is generally unregulated.
- Direction finding and localisation of *energy* is passive; it does not require a licence.
- **Do not decode or act on content you are not entitled to.** The project measures the physical layer,
  not private message content.

---

## 4. Low-power ISM transmission (the calibration beacon)

- In the US, Part 15 rules govern 902–928 MHz: limits on conducted/EIRP power and, for frequency-hopping
  and digital modulation, band-specific requirements. The beacon stays well within these.
- Outside the US, equivalent ISM rules apply (e.g. 868 MHz in Europe, with duty-cycle limits).
- Keep beacon transmissions short, infrequent, and at minimum necessary power.

---

## 5. Amateur bands (only if licensed)

- Amateur operation requires a licence and: station identification, **no encryption or obscured messages**
  (e.g. US Part 97.113(a)(4)), and minimum necessary power.
- **No business or commercial content** — relevant to any trust/business context; ham bands are not a
  business data path.
- Third-party traffic rules apply (traffic must be authorized; international third-party traffic needs an
  agreement).
- The Web-888 HF/VHF reception and the RTL-SDR sub-GHz reception described here are purely passive; they
  do not transmit.

---

## 6. Transmit beamforming / distributed arrays (future, out of scope)

- Raising effective radiated power via beamforming can breach EIRP limits even when driver power is legal.
  Any future transmit-side work must re-check limits before transmitting.
- Distributed coherent transmission across nodes consumes spectrum and timing resources; it is future
  research, not a current capability.

---

## 7. Data and privacy

- SDR captures may incidentally include third-party transmissions. Keep captures for engineering purposes,
  avoid publishing raw third-party payloads, and publish derived measurements instead.
- **Hash personal identifiers** (Wi-Fi BSSIDs, BLE MACs) at ingest; never publish household-identifying
  data.
- Never publish keys, addresses or private infrastructure details in this repository.

---

## 8. Safety

- **No jamming, no interference generation, no denial-of-service experiments, ever.** Interference features
  in this project are **passive detection and reporting** only.
- When in doubt about whether a transmission is permitted, do not transmit.
