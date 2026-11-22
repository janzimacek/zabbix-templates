# Release checklist

## Completed

- [x] Zabbix export version 8.0
- [x] Core separated from dashboard presentation
- [x] Universal physical-disk LLD
- [x] 2-disk dashboard finalized
- [x] 4-disk dashboard finalized
- [x] 8-disk dashboard finalized
- [x] No 1-disk dashboard by design
- [x] YAML parse validation
- [x] UUIDv4-format validation
- [x] SHA-256 checksums generated

## Recommended before publishing v1.0.0

- [ ] Perform clean import into a non-production Zabbix 8.0 instance
- [ ] Verify clean installation of Core + each dashboard variant
- [ ] Re-test disk discovery on representative 2-, 4- and 8-disk Synology systems
- [ ] Verify dashboard after a Zabbix service restart
- [ ] Test at least one disk-warning trigger safely
- [ ] Add anonymized dashboard screenshots under `docs/images/` if desired
- [ ] Set the preferred copyright holder in `LICENSE`
- [ ] Create GitHub release/tag `v1.0.0`
