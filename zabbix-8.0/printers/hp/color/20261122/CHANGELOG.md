# Changelog

## v1.0.0 - 2026-11-21

- Initial GitHub release for HP Color LaserJet Pro MFP 4302dw.
- Based on the validated HP printer template family.
- Replaced unreliable localized supplies LLD with verified static B/C/M/Y Printer-MIB indexes.
- Added Black, Cyan, Magenta and Yellow toner graphs.
- Added Warning <20% and High <10% toner triggers.
- Added Warning-to-High trigger dependencies for every toner.
- Added `component=toner` and per-color tags.
- Added dashboard with 2 x 2 half-height toner graph layout.
- Retained exact printed-page formatting, ping, uptime and Problems widget.
