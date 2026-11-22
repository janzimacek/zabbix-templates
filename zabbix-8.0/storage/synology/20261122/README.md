# Synology NAS SNMP templates for Zabbix 8.0

Universal Synology NAS monitoring for **Zabbix 8.0**, split into a reusable monitoring Core and optional 2-, 4-, and 8-disk dashboard layouts.

## Templates

| File | Purpose |
|---|---|
| `templates/template_synology_nas_core.yaml` | Monitoring Core, no dashboard. Physical disks are discovered automatically with LLD. |
| `templates/template_synology_nas_dashboard_2_disk.yaml` | Final dashboard layout for 2 physical disks. |
| `templates/template_synology_nas_dashboard_4_disk.yaml` | Final dashboard layout for 4 physical disks. |
| `templates/template_synology_nas_dashboard_8_disk.yaml` | Final dashboard layout for 8 physical disks. |

There is intentionally no 1-disk version.

## What the Core monitors

- ICMP availability
- model and DSM version
- system and power status
- Fan 1 / Fan 2 status
- system temperature
- hardware uptime
- RAID / Volume 1 status
- Volume 1 free, used, total and utilization
- physical disks through Zabbix low-level discovery (LLD):
  - disk model
  - disk status
  - disk health
  - temperature
  - bad sectors
  - identify failures
- trigger prototypes for disk status, health, bad sectors, identify failures and high temperature
- dynamic disk-temperature graph prototype

## Dashboard features

The dashboard variants use the same visual design and include:

- NAS status
- model
- DSM version
- system temperature
- volume utilization
- RAID / Volume status
- uptime
- Fan 1 / Fan 2 status
- ping response time
- ICMP availability
- fixed per-drive Health tiles for the selected 2/4/8-disk layout
- ping response-time graph
- system-temperature graph
- volume-capacity graph
- dynamic disk-temperature graphs from Core LLD
- active Problems widget

Where supported by the Zabbix Item value widget, normal states are shown in green and warning/failure thresholds in orange/red.

## Requirements

- Zabbix **8.0**
- Synology DSM with SNMP enabled
- Synology enterprise MIB/OID support exposed by the NAS
- SNMP interface configured on the Zabbix host
- ICMP allowed from the Zabbix server/proxy for availability and response-time checks

The templates were developed around SNMPv2c. Use a dedicated **read-only** community and restrict allowed NMS/source addresses when possible.

## Installation

### 1. Import Core

In Zabbix:

`Data collection > Templates > Import`

Import:

`templates/template_synology_nas_core.yaml`

For the initial import enable **Create new** and **Update existing**. Leave **Delete missing** disabled unless an upgrade note explicitly says otherwise.

### 2. Import dashboard layouts

Import the required dashboard YAML, or all three if the same Zabbix installation monitors multiple Synology models.

### 3. Link the host

Link exactly **one dashboard template** to each NAS host:

- 2 installed disks -> `Synology NAS by SNMP - Dashboard 2 disks`
- 4 installed disks -> `Synology NAS by SNMP - Dashboard 4 disks`
- 8 installed disks -> `Synology NAS by SNMP - Dashboard 8 disks`

The dashboard template already links the Core template.

**Do not link Core directly to the host at the same time.** Zabbix will reject the duplicate parent-template inheritance.

### 4. Configure SNMP

Configure the host SNMP interface with the NAS IP/DNS name, UDP/161 and read-only credentials.

### 5. Run first discovery

Open the inherited `Synology disk discovery` rule and run **Execute now**. Then verify `Monitoring > Latest data`.

The number of LLD disk items should match the number of physical disks exposed by the Synology SNMP disk table.

## Default thresholds

| Setting | Default |
|---|---:|
| Disk temperature warning | 50 °C |
| System temperature warning | 60 °C |
| System temperature critical | 70 °C |
| Volume utilization warning | 80 % |
| Volume utilization critical | 90 % |

These defaults are starting points. Override the macros for specific drive classes, Synology models and environmental conditions.

## Dashboard design note

Zabbix LLD can automatically create item, trigger and graph instances, but it does not dynamically create arbitrary Item value dashboard tiles. For this reason:

- disk discovery and fault monitoring remain universal in Core;
- per-disk Health tiles are provided by separate 2/4/8-disk presentation templates;
- disk-temperature graphs remain dynamic through a Graph prototype widget.

This keeps monitoring logic common while allowing clean dashboards for typical Synology bay counts.

## Troubleshooting

### Only one disk is discovered

1. Run `Synology disk discovery > Execute now`.
2. Inspect LLD results and `Latest data`.
3. Verify both disks expose rows in the Synology disk SNMP table.
4. Check the `{$SYNOLOGY.DISK.MODEL.MATCHES}` macro. Default is `.+`.

### `parent template Core would be linked twice`

Core is already linked directly to the host. Use **Unlink** on Core, not **Unlink and clear**, then link only the selected dashboard template.

### Ping response time is unsupported

The final dashboard templates use a valid Zabbix SIMPLE `icmppingsec` check. Confirm `fping` is installed/usable by Zabbix and ICMP is allowed.

### Dashboard Health tile differs from LLD disk state

Dashboard Health tiles are presentation items for the expected disk indexes. Core LLD remains authoritative for monitoring and trigger generation. Check the SNMP disk table/indexes and make sure the correct 2/4/8-disk dashboard was selected.

## Upgrade safety

Before changing a production template:

1. Export the currently working template.
2. Import the new version with **Update existing** enabled.
3. Keep **Delete missing** disabled unless the release notes specifically require cleanup.
4. Verify `Latest data`, discovery, triggers and the dashboard before removing the old export/backup.

## Repository layout

```text
templates/
  template_synology_nas_core.yaml
  template_synology_nas_dashboard_2_disk.yaml
  template_synology_nas_dashboard_4_disk.yaml
  template_synology_nas_dashboard_8_disk.yaml
README.md
CHANGELOG.md
RELEASE_CHECKLIST.md
SHA256SUMS
LICENSE
.gitignore
```

## Versioning

Initial GitHub release: **v1.0.0**.

## License

MIT License. See `LICENSE`.
