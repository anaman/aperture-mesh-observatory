# Future scope

Capabilities the project intends to grow into. None are current; each is recorded with its real
constraints so nothing is promised that the hardware or the rules do not allow.

---

## 1. Winlink / B2F integration

### 1.1 What B2F is

*Open B2F — Winlink Message Structure and B2 Forwarding Protocol* (spec last revised February 2018)
defines how messages are transferred between **Winlink Radio Mail Server (RMS) gateway stations** and
Winlink-compatible **client programs**.

- **Transports:** Pactor 1–4, WINMOR, ARDOP, VARA, Robust Packet, AX.25 Packet, and Telnet over TCP/IP.
- **Message structure:** header + ASCII body + attachments (8-bit bytes, filename up to 255 characters).
- **Compression:** up to five messages combined and compressed as one file using FBB B1 (LZH/LZHUF);
  source archived at `ARSFI/Winlink-Compression`.
- **Address header fields:** Mid, Date, Type, From, To, Cc, Subject, Mbo, Body, File — where **Type**
  includes Bulletin, Private, Service, Inquiry, **Position Report**, Position Request, Option and System.
- **Monitoring:** FCC §97.309 "listen mode" permits on-air monitoring of documented codes.

### 1.2 How the observatory would integrate

Not by reimplementing B2F — by **producing messages for an existing client**.

- Reuse an open-source B2F client (**PAT**) or an existing client; the observatory becomes a message
  producer, not a protocol implementation.
- The **Position Report** message type is a natural fit: the RF survey can emit georeferenced telemetry
  directly into Winlink, giving off-grid situational reports when the internet is down.
- **LoRa is not a Winlink transport.** Integration is therefore a **gateway/bridge**: mesh traffic or
  survey output → B2F client → one of the supported transports (or Telnet/CMS when the internet is
  reachable).

### 1.3 The hardware gap

Winlink over RF needs a **transmit** radio with a suitable modem — VHF/UHF (VARA FM, AX.25 packet) or HF
(VARA HF, ARDOP, Pactor). **Everything on the current bench is receive-only on those bands:** the Web-888
receives HF/VHF only, the RTL-SDR receives only, and the Heltec transmits only on 915 MHz ISM LoRa.
Winlink operation requires a **new transceiver** — a genuine addition to the equipment list.

### 1.4 The legal line (important)

Winlink's RF services are **amateur radio**:

- A **licence** is required.
- **No business or commercial content.** (Relevant to the trust/business context: Winlink on ham bands is
  not a channel for commercial traffic.)
- **No encryption or obscured messages** beyond what the rules permit.
- **Third-party traffic rules** apply (traffic must be authorized; international third-party traffic needs
  an agreement).
- Station identification is required.

So: Winlink integration is an **off-grid situational-reporting and emergency-communications** feature, not
a business data path.

---

## 2. Distributed sensor and transmitter grid

### 2.1 Scope — who the nodes are

The grid uses only devices **we own or have explicit written authorization to control**. Commandeering
third-party devices is not a feature of this project: it would be unauthorized access and unauthorized
transmission, and the project does not do it. Recording this plainly so the boundary is never ambiguous.

### 2.2 Concept

Deploy our **own** ESP32/LoRa nodes across the area. Each node:

- **listens** on the relevant bands and reports RSSI + timestamp + its own position to the Linux core;
- if it carries a LoRa transceiver, may **transmit** on the ISM mesh (MeshCore / Meshtastic / Reticulum)
  **as an authorized participant**, within power and duty-cycle limits.

### 2.3 Hardware reality

- ESP32-C6 / ESP32-S3 boards have **no sub-GHz LoRa radio** — they are 2.4 GHz Wi-Fi / BLE / 802.15.4.
  An ESP32-S3 can host an external SX1262, but a bare C6/S3 **cannot hear 915 MHz**.
- Decoding LoRa therefore needs **LoRa-capable nodes** (Heltec-class, SX1262 + MCU). The C6/S3 boards
  cover the 2.4 GHz Wi-Fi/BLE layer.
- A useful grid is therefore **mixed**: LoRa nodes for the mesh bands, ESP32-C6/S3 nodes for 2.4 GHz.

---

## 3. Multi-protocol decode (MeshCore / Meshtastic / Reticulum)

Handled by Phase 3 (software LoRa PHY + protocol framing). Notes:

- Decoding is **passive** and permitted.
- Payload contents are only available where we hold the key or are a member of the network; without that,
  channel traffic is opaque but **physical-layer metadata** (identity, timing, signal strength, occupancy)
  is still measurable — and is often the more useful product.

---

## 4. Responding to a message

Replying is **transmitting**, so it follows the same rules as any transmission:

- only as an **authorized participant** on that network;
- only to traffic **addressed to us**, or as an accepted member of that network;
- within power and duty-cycle limits;
- never impersonating another station and never injecting into a third party's conversation.

When the operator chooses to respond, the platform sends through the appropriate radio (the Heltec modem
on the ISM mesh, or the future Winlink radio on ham bands).

---

## 5. Direction visualisation

"Show me which direction that message came from" has two honest routes:

| Route | Requirement | Accuracy |
|---|---|---|
| **Coherent bearing** | ≥2 phase-coherent receivers + calibration (Phase 5) | true bearing |
| **Multi-node RSSI localisation** | several **owned** receivers at known positions (no phase coherence needed) | coarse position/region |

With a **single** receiver there is **no** bearing — only "heard / not heard", which roving motion converts
into a coverage map but not into a direction. The direction visual is therefore a Phase 5/6 deliverable,
or a multi-node estimate.

---

## 6. Guardrails (non-negotiable)

- **Listen-first, licence-gated transmit.** The full sensing capability works without any licence because
  it is passive. Transmit is per-band and locked behind an explicit licence declaration; the operator
  bears responsibility for every transmission.
- Transmit only as an authorized participant, within power and duty-cycle limits.
- Only devices we own or are explicitly authorized to control.
- No jamming, no deauthentication, no interference generation.
- No payload capture beyond what we are entitled to receive.
- Hash personal identifiers (Wi-Fi BSSIDs, BLE MACs); publish aggregates, never household-identifying data.
- Winlink: amateur-only, no business content, no encryption or obscuring.
