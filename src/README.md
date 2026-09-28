# Source

Software for the Aperture Mesh Observatory lives here. It is organised so that each part can be tested
alone and reused, and so a reader can follow the data from antenna to result.

| Directory | Responsibility |
|---|---|
| `capture/` | Acquire timestamped IQ from the RTL-SDR dongles (USB and `rtl_tcp`) and the Web-888. |
| `channelise/` | Down-convert, filter and select channels per the channel plan. |
| `measure/` | Estimate RSSI/SNR, detect packets, estimate arrival time and (later) phase difference. |
| `demod/` | LoRa PHY demodulation (`gr-lora_sdr` integration) and protocol framing. |
| `truth/` | Drive the MeshCore modem: decode log, calibration-beacon control. |
| `fuse/` | Combine receivers: deduplication, TDOA, bearing, coverage fusion. |
| `serve/` | Unified live view, logs, dataset writer. |
| `relay/` | Spatially-aware relay logic (later phases). |

Nothing here yet — Phase 0 is documentation. The first code (Phase 1) will appear under `capture/`,
`measure/` and `serve/`.

Conventions:
- Small, single-purpose modules; a module that cannot be tested alone is too big.
- Every measurement writes its configuration and timestamp alongside its result.
- No secrets, addresses or private infrastructure details in source or config committed here.
