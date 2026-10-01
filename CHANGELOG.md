# Changelog

All notable changes to this project are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/); the project aims for semantic-ish versioning of
deliverables per phase.

## [Unreleased]

### Changed — 2026-10-01 (listen-first principle)
- `docs/REGULATORY.md` rewritten: listen-first, licence-gated transmit; per-band transmit gate table;
  implementation of the gate.
- Guardrails added to `docs/FUTURE_SCOPE.md` and `docs/ROADMAP.md`; `README.md` safety section updated.

### Added — 2026-10-01 (future scope)
- `docs/FUTURE_SCOPE.md` — Winlink/B2F integration, distributed (owned) node grid, multi-protocol decode,
  the "respond" rules, and direction visualisation.
- `docs/ROADMAP.md` Phase 7 expanded to include the Winlink integration and the node grid.

### Changed — 2026-10-01 (roving confirmed)
- `docs/RF_SURVEY.md`: roving (wardriving) is the primary mode; added roving rig, procedure and tooling.
- `docs/ROADMAP.md`: Phase 2 restated as "roving (wardriving) first".
- `README.md`, `docs/PROPOSAL.md`: updated to match.

### Added — 2026-09-28 (Phase 0)
- Initial repository: `README.md`.
- `docs/PROPOSAL.md` — full project proposal.
- `docs/ARCHITECTURE.md` — reference / sensing / act-truth planes and data flow.
- `docs/HARDWARE.md` — bench inventory, planned clock distribution and phase calibration.
- `docs/ROADMAP.md` — Phases 0–6 with exit criteria.
- `docs/REPLICATION.md` — rebuild guide.
- `docs/REGULATORY.md` — spectrum, power and privacy posture.
- `docs/GLOSSARY.md` — terms.
- `docs/REFERENCES.md` — annotated, dated source list.
- `DEVLOG.md` — development log.
- `LICENSE` (GPL-3.0), `.gitignore`, `src/README.md`.

### Changed / Added — 2026-09-28 (bench confirmed, RF survey)
- Confirmed bench: 1× RTL-SDR, 1× Web-888, 1× Heltec V4 (corrects the earlier two-dongle assumption).
- `docs/RF_SURVEY.md` — georeferenced community RF survey subsystem (sub-GHz + Wi-Fi/BLE mapping).
- `docs/ROADMAP.md` — inserted Phase 2 (RF survey); phased plan renumbered to 0–7.
- `docs/HARDWARE.md` — rewritten for the confirmed bench; added capability matrix and honest gaps
  (no 2.4 GHz on the RTL-SDR, no 5 GHz, one receiver is not an array, Wi-Fi/BLE needs a spare ESP32).
- `README.md` — confirmed hardware table, survey capability, renumbered roadmap, doc index.
- `docs/PROPOSAL.md` — objectives, approach, phased plan and open questions updated.
- `DEVLOG.md` — decision entry for the confirmed bench and the added survey.
