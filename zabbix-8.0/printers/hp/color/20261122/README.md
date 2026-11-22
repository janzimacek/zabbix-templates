# HP Color LaserJet Pro MFP 4302dw by SNMP for Zabbix 8.0

Zabbix 8.0 SNMP template for the HP Color LaserJet Pro MFP 4302dw.

This model-specific template uses Printer-MIB indexes verified on a real HP Color LaserJet Pro MFP 4302dw. It is based on the previously validated HP printer template family, but toner monitoring is static rather than LLD because the tested printer exposes localized Czech supply descriptions as Hex-STRING values.

## Features

- ICMP availability
- Ping response time
- SNMP system name, description and uptime
- Printed pages total
- Dashboard-friendly printed-page value with space-separated thousands
- Black, Cyan, Magenta and Yellow toner monitoring
- For each toner:
  - maximum capacity
  - current level
  - remaining percentage
  - dedicated graph
- Warning below 20%
- High below 10%
- Warning trigger depends on High trigger, preventing duplicate simultaneous alarms
- All toner triggers tagged `component=toner`
- Additional toner color tags: `color=black|cyan|magenta|yellow`
- Host dashboard included

## Verified Printer-MIB indexes

| Toner | Cartridge | Max OID suffix | Level OID suffix |
|---|---|---|---|
| Black | HP W2200A | `.8.1.1` | `.9.1.1` |
| Cyan | HP W2201A | `.8.1.2` | `.9.1.2` |
| Magenta | HP W2203A | `.8.1.3` | `.9.1.3` |
| Yellow | HP W2202A | `.8.1.4` | `.9.1.4` |

The tested printer returned maximum capacity `100` for all four cartridges.

## Requirements

- Zabbix 8.0
- HP Color LaserJet Pro MFP 4302dw
- SNMP enabled on the printer
- SNMP interface configured on the Zabbix host
- ICMP permitted if availability/response-time monitoring is required

Use a dedicated read-only SNMP community and restrict access to the Zabbix server/proxy where possible.

## Installation

1. Open **Data collection > Templates > Import**.
2. Import `templates/template_hp_color_laserjet_pro_mfp_4302dw_snmp.yaml`.
3. Enable **Create new** and **Update existing**.
4. For the initial import leave **Delete missing** disabled.
5. Link `HP Color LaserJet Pro MFP 4302dw by SNMP` to the printer host.
6. Configure the host SNMP interface.
7. Verify **Monitoring > Latest data**, Triggers and the host dashboard.

## Toner thresholds and dependencies

Default macros:

| Macro | Default |
|---|---:|
| `{$PRINTER.TONER.WARN}` | `20` |
| `{$PRINTER.TONER.CRIT}` | `10` |

Behavior for every toner:

- 25%: no problem
- 15%: Warning only
- 8%: High only

The Warning trigger depends on the High trigger for the same toner.

Every toner trigger is tagged:

```text
component=toner
```

and also with the corresponding color tag.

## Central toner dashboard

A central Zabbix Problems widget can use:

```text
Tag name: component
Operator: Equals
Value: toner
```

This collects toner problems from this template together with other printer templates using the same tag convention.

## Dashboard

The included dashboard contains:

- ICMP availability
- Ping response time
- System name
- Uptime
- exact printed-page count
- toner graphs in a 2 x 2 layout:
  - Black | Cyan
  - Magenta | Yellow
- ping response graph
- printed pages graph
- Printer Problems widget

Toner graphs use half-height dashboard widgets compared with the original four-across design.

## Printed page formatting

The numeric `printer.pages.total` item remains authoritative for history and graphs. A dependent text item formats the dashboard value with a space as the thousands separator, e.g. `10 328`.

## SNMP verification

If toner values do not match, validate the actual device before editing the template:

```bash
snmpwalk -v2c -c COMMUNITY PRINTER_IP 1.3.6.1.2.1.43.11.1.1.6
snmpwalk -v2c -c COMMUNITY PRINTER_IP 1.3.6.1.2.1.43.11.1.1.8
snmpwalk -v2c -c COMMUNITY PRINTER_IP 1.3.6.1.2.1.43.11.1.1.9
```

This release is deliberately model-specific. Do not assume these indexes are identical on other HP Color LaserJet models without verifying Printer-MIB output first.

## Release

GitHub package based on template revision **v5**.
