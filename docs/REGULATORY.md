# Regulatory posture

This project is passive sensing first. Passive reception is generally permissible and unregulated in most
jurisdictions. The only transmissions the project makes are a deliberately compliant calibration beacon
from a licensed-for-purpose radio.

> This is engineering guidance, not legal advice. Verify against your own national and local rules.

## 1. Reception (the bulk of the project)

- Passive monitoring of ISM bands (US: 902–928 MHz, Part 15) is generally unregulated.
- Direction finding and localisation of *energy* is passive; it does not require a licence.
- **Do not decode or act on content you are not entitled to.** The project measures the physical layer,
  not private message content.

## 2. Low-power ISM transmission (the calibration beacon)

- In the US, Part 15 rules govern 902–928 MHz: limits on conducted/EIRP power and, for frequency-hopping
  and digital modulation, band-specific requirements. The beacon must stay within these.
- Outside the US, equivalent ISM rules apply (e.g. 868 MHz in Europe, with duty-cycle limits).
- Keep beacon transmissions short, infrequent and at minimum necessary power.

## 3. Amateur radio bands (only if used)

- Some related community mesh activity sits on amateur bands (e.g. AREDN at 2.4 GHz/900 MHz, APRS, D-STAR).
- Amateur operation requires a licence and: station identification, **no encryption or obscured
  messages** (e.g. US Part 97.113(a)(4)), and minimum necessary power.
- The Web-888 HF/VHF reception described here is purely passive listening; it does not transmit.

## 4. Transmit beamforming / distributed arrays (Phase 6, out of scope)

- Raising effective radiated power via beamforming can breach EIRP limits even if the driver power is
  legal. Any future transmit-side work must re-check limits before transmitting.
- Distributed coherent transmission across nodes consumes spectrum and timing resources; it is future
  research, not a current capability.

## 5. Data and privacy

- SDR captures may incidentally include third-party transmissions. Keep captures for engineering purposes,
  avoid publishing raw third-party payloads, and prefer publishing derived measurements.
- Never publish keys, addresses or private infrastructure details in this repository.

## 6. Safety

- No jamming, no interference generation, no denial-of-service experiments, ever. Interference features in
  this project are **passive detection and reporting** only.
