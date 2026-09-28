# Glossary

Terms are spelled out in full here and, on first use in prose, in the project's documents.

- **Aperture** — the effective extent of an antenna array. A physical array has a real aperture; multiple
  cooperating receivers can synthesise a larger (synthetic) aperture, improving angular resolution.
- **Aperture Mesh Observatory** — this project: a GPS-disciplined, multi-receiver coherent sensing platform
  for LoRa mesh networks.
- **Carrier phase** — the phase of the received radio carrier at a receiver. Comparing carrier phase across
  array elements gives a bearing; it requires phase-coherent receivers.
- **Coherence** — the property that measurements from separate receivers can be combined meaningfully. This
  project distinguishes time/frequency coherence from carrier-phase coherence.
- **Co-array (difference co-array)** — the set of pairwise element spacings of an array. Sparse geometries
  with rich co-arrays can match the resolution of a full array with fewer elements.
- **Direction of arrival** — the angle from which a signal arrives at an array.
- **Duty cycle** — the fraction of time a transmitter occupies the channel; regulated in many bands.
- **GPSDO** — GPS-disciplined oscillator: an oscillator whose frequency is steered by GPS timing signals.
- **Injected reference** — a deliberately added reference signal fed into every receiver channel so their
  relative phases can be measured and corrected (the Laakso method).
- **IQ** — in-phase / quadrature samples; the complex baseband representation of a radio signal.
- **LoRa** — a chirp-spread-spectrum radio modulation used by MeshCore, Meshtastic and others.
- **MeshCore** — a lightweight hybrid routing/flooding mesh protocol for LoRa packet radios.
- **Multipath** — a signal arriving by several paths; a major error source for bearing, and a source of
  diversity gain for reception.
- **MUSIC** — a subspace method for direction-of-arrival estimation.
- **PPS** — pulse-per-second; the timing output of a GPS receiver.
- **rtl_tcp** — a small server that streams samples from an RTL-SDR dongle over the network, so a receiver
  can be physically remote from the host.
- **Short baseline / long baseline** — small receiver spacing (enabling bearing via carrier phase) versus
  large spacing (enabling position via arrival-time difference).
- **Si5351** — a programmable clock synthesiser; in the Web-888 it produces the ADC clock and can drive the
  clock-out output.
- **TDOA** — time difference of arrival; localising a transmitter from the arrival-time differences at
  separated receivers.
- **Web-888** — a single-board, networked HF/VHF SDR used here as the GPS time/frequency reference and
  HF/VHF monitor. It cannot receive the 902–928 MHz LoRa band.
