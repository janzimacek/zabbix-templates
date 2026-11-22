# Zabbix Template for UniFi UDM Pro

A community template for monitoring **Ubiquiti UniFi UDM Pro** with **Zabbix 8.0**.

The template combines three monitoring methods:

- the official local **UniFi Network API** for controller, gateway, client and adopted-device data;
- **SNMPv2 / IF-MIB** for individual WAN link status and traffic counters;
- **ICMP** for independent UDM Pro availability monitoring.

The template includes items, dependent items, low-level discovery, triggers, graphs, value maps and a host dashboard. All environment-specific values are configured as **host-level macros**. The exported template contains no API key, SNMP community, IP address, site ID, device ID, ISP name or WAN capacity.

## Features

### UDM Pro health

- ICMP availability
- UniFi API availability watchdog
- UDM Pro state, model, name, firmware, MAC address and API-reported IP address
- CPU utilization
- Memory utilization
- Uptime and restart detection
- Load averages for 1, 5 and 15 minutes
- Aggregate uplink receive and transmit rates reported by the UniFi API

### UniFi clients

- Total connected clients
- Wired clients
- Wireless clients
- VPN clients
- Client-count history and graph

### Adopted UniFi devices

Low-level discovery discovers adopted UniFi devices and creates items for:

- device state;
- IP address;
- firmware version;
- firmware-update availability.

Trigger prototypes report:

- a device that is not online;
- an available firmware update.

### Dual-WAN monitoring

Two WAN interfaces are monitored through standard IF-MIB 64-bit counters:

- physical link state;
- download rate in bit/s;
- upload rate in bit/s;
- download utilization as a percentage of configured capacity;
- upload utilization as a percentage of configured capacity.

The ISP labels, SNMP interface indices and nominal line capacities are configured independently on each host.

### Included dashboard

The host dashboard **UDM Pro - Network overview** includes:

- UDM Pro availability;
- WAN1 and WAN2 status;
- connected-client count;
- WAN traffic graphs;
- WAN utilization graph;
- CPU and memory gauges;
- load-average graph;
- connected-clients graph;
- active UDM and UniFi problems.

Dashboard widget titles use the generic labels `WAN1`, `WAN2`, `WAN1 traffic` and `WAN2 traffic`. Zabbix 8.0 does not expand user macros in widget titles. ISP macros are expanded in item names, graph legends, Latest data and problem names. Users may manually rename dashboard widgets after import.

## Tested environment

- Zabbix Server and Frontend: **8.0**
- UniFi Network: **10.6.x**
- UniFi OS / UDM Pro firmware: **5.1.x**
- Device: **UniFi Dream Machine Pro**
- SNMP: **SNMPv2**

Other releases may work, but API response fields and available endpoints can change between UniFi Network versions.

## Requirements

Before installation, make sure that:

1. The Zabbix server or proxy can reach the UDM Pro over:
   - HTTPS TCP/443;
   - SNMP UDP/161;
   - ICMP.
2. The UDM Pro has a local UniFi Network API key.
3. SNMP is enabled on the UDM Pro.
4. The Zabbix server has ICMP pinger support available.
5. The WAN interfaces and their SNMP `ifIndex` values are known.

## Installation

### 1. Download and import the template

Download:

```text
template_unifi_udm_pro_http_snmp.yaml
```

In Zabbix, open:

```text
Data collection → Templates → Import
```

Select the YAML file, review the import summary and click **Import**.

The imported template is named:

```text
UniFi UDM Pro by HTTP and SNMP
```

### 2. Create the UDM Pro host

Open:

```text
Data collection → Hosts → Create host
```

Example values:

```text
Host name: UDM-PRO
Visible name: UniFi UDM Pro
Host group: Network devices
```

Link the template:

```text
UniFi UDM Pro by HTTP and SNMP
```

A Zabbix agent interface is not needed. UDM Pro does not normally run a Zabbix agent on TCP/10050.

### 3. Add the SNMP interface

In the host configuration, add an interface of type **SNMP**:

```text
IP address: UDM Pro management address
Port: 161
Connect to: IP
SNMP version: SNMPv2
SNMP community: {$SNMP_COMMUNITY}
```

The community value itself will be defined as a host macro of type **Secret text**.

## Create a UniFi Network API key

In the UniFi Network application, open the local **Integrations** section and create a dedicated API key for Zabbix.

Recommended practices:

- use a dedicated key only for monitoring;
- grant read-only access when the UniFi version provides permission selection;
- do not reuse a personal or administrative token;
- store the key in Zabbix as a **Secret text** host macro;
- never commit the key to GitHub or include it directly in a YAML export.

The template authenticates with this HTTP header:

```text
X-API-KEY: {$UNIFI.API.KEY}
```

Bearer authentication is not used.

## Obtain the UniFi Site ID

Run the following command on the Zabbix server or another trusted host. `read -s` prevents the key from being displayed on screen.

```bash
read -s APIKEY
curl -sk \
  -H "X-API-KEY: $APIKEY" \
  "https://UDM_ADDRESS/proxy/network/integration/v1/sites"
unset APIKEY
```

Example response:

```json
{
  "offset": 0,
  "limit": 25,
  "count": 1,
  "totalCount": 1,
  "data": [
    {
      "id": "SITE_UUID",
      "internalReference": "default",
      "name": "Default"
    }
  ]
}
```

Use the value from `data[0].id` as:

```text
{$UNIFI.SITE.ID}
```

## Obtain the UDM Pro Device ID

Using the Site ID obtained above:

```bash
read -s APIKEY
curl -sk \
  -H "X-API-KEY: $APIKEY" \
  "https://UDM_ADDRESS/proxy/network/integration/v1/sites/SITE_UUID/devices"
unset APIKEY
```

Locate the object with:

```json
"model": "UDM Pro"
```

Use that object's `id` value as:

```text
{$UNIFI.DEVICE.ID}
```

Do not use the MAC address or the site UUID in place of the device UUID.

## Identify WAN SNMP interface indices

The template uses IF-MIB interface indices. These indices are not universal and can change with model, firmware and port assignment.

List interface names:

```bash
snmpwalk -v2c -c COMMUNITY UDM_ADDRESS \
  1.3.6.1.2.1.31.1.1.1.1
```

Check operational states:

```bash
snmpwalk -v2c -c COMMUNITY UDM_ADDRESS \
  1.3.6.1.2.1.2.2.1.8
```

Check negotiated speeds in Mbit/s:

```bash
snmpwalk -v2c -c COMMUNITY UDM_ADDRESS \
  1.3.6.1.2.1.31.1.1.1.15
```

For selected candidate indices:

```bash
for i in 3 4 5; do
  echo "===== INDEX $i ====="
  snmpget -v2c -c COMMUNITY UDM_ADDRESS \
    1.3.6.1.2.1.31.1.1.1.1.$i \
    1.3.6.1.2.1.2.2.1.7.$i \
    1.3.6.1.2.1.2.2.1.8.$i \
    1.3.6.1.2.1.31.1.1.1.15.$i
done
```

Relevant values:

```text
ifAdminStatus 1 = administratively enabled
ifOperStatus  1 = link up
ifOperStatus  2 = link down
```

To confirm the active WAN by traffic counters:

```bash
watch -n 2 '
for i in 3 4 5; do
  echo -n "index $i: "
  snmpget -v2c -c COMMUNITY -Ovq UDM_ADDRESS \
    1.3.6.1.2.1.31.1.1.1.6.$i \
    1.3.6.1.2.1.31.1.1.1.10.$i
done
'
```

Generate traffic through one WAN and observe which 64-bit counters increase.

## Required host macros

The template intentionally defines no macro values. Create all macros directly on the UDM Pro host:

```text
Data collection → Hosts → UDM-PRO → Macros
```

| Macro | Type | Example | Description |
|---|---|---:|---|
| `{$UNIFI.HOST}` | Text | `192.0.2.1` | UDM Pro IP address or DNS name, without `https://` |
| `{$UNIFI.API.KEY}` | Secret text | hidden | Local UniFi Network API key |
| `{$UNIFI.SITE.ID}` | Text | UUID | Site UUID returned by the sites endpoint |
| `{$UNIFI.DEVICE.ID}` | Text | UUID | UDM Pro UUID returned by the devices endpoint |
| `{$SNMP_COMMUNITY}` | Secret text | hidden | SNMPv2 community configured on UDM Pro |
| `{$WAN1.ISP}` | Text | `ISP 1` | Friendly ISP name used in item names, graph legends and triggers |
| `{$WAN1.IFINDEX}` | Text | `3` | SNMP `ifIndex` for WAN1 |
| `{$WAN1.DOWNLOAD.BPS}` | Text | `30000000` | Nominal WAN1 download rate in bit/s |
| `{$WAN1.UPLOAD.BPS}` | Text | `30000000` | Nominal WAN1 upload rate in bit/s |
| `{$WAN2.ISP}` | Text | `ISP 2` | Friendly ISP name used in item names, graph legends and triggers |
| `{$WAN2.IFINDEX}` | Text | `4` | SNMP `ifIndex` for WAN2 |
| `{$WAN2.DOWNLOAD.BPS}` | Text | `250000000` | Nominal WAN2 download rate in bit/s |
| `{$WAN2.UPLOAD.BPS}` | Text | `50000000` | Nominal WAN2 upload rate in bit/s |

Capacity macros use **bits per second**, not Mbit/s. Examples:

```text
30 Mbit/s  = 30000000
50 Mbit/s  = 50000000
250 Mbit/s = 250000000
1 Gbit/s   = 1000000000
```

## Verify the installation

After saving the host, open:

```text
Monitoring → Latest data
```

Filter by the UDM Pro host and verify these items first:

```text
UDM Pro: ICMP availability
UniFi Sites JSON
UniFi Devices JSON
UDM Pro Statistics JSON
UniFi Clients JSON
```

Expected results:

- ICMP availability is `Up (1)`;
- JSON master items return valid JSON;
- CPU, memory, uptime and load averages receive values;
- connected-client counts are populated;
- device discovery creates items for the UDM Pro and adopted access points or switches;
- WAN status items show `Up (1)`;
- WAN traffic items receive values after at least two SNMP polls.

WAN traffic preprocessing uses:

1. `Change per second` to convert the cumulative octet counter to octets per second;
2. `Custom multiplier = 8` to convert octets per second to bits per second.

The first traffic value may be empty because the rate calculation needs two consecutive counter samples.

## Items and retention

Raw API master items retain history for only **1 day**:

```text
UniFi Clients JSON
UniFi Devices JSON
UniFi Sites JSON
UDM Pro Statistics JSON
```

This limits long-term storage of raw client names, IP addresses and MAC addresses. Numeric and derived monitoring items use longer history periods for graphs and trends.

## Triggers

The template includes triggers for:

- UDM Pro unreachable by ICMP;
- UniFi API returning no device data;
- high CPU utilization;
- high memory utilization;
- UDM Pro restart;
- WAN1 physical-link failure;
- WAN2 physical-link failure;
- both WAN physical links down;
- sustained high WAN download utilization;
- sustained high WAN upload utilization;
- discovered UniFi device not online;
- firmware update available for a discovered device.

WAN link triggers use four consecutive non-up values. With a 30-second polling interval, this represents approximately two minutes of persistent link failure.

The `Both WAN physical links are down` trigger depends on the ICMP availability trigger. Individual WAN triggers depend on the combined dual-WAN trigger to reduce duplicate alerts.

## Graphs

Included graphs:

- `UDM Pro: WAN1 traffic`
- `UDM Pro: WAN2 traffic`
- `UDM Pro: WAN utilization`
- `UDM Pro: Load average`
- `UniFi: Connected clients`

WAN traffic graphs use automatically calculated Y-axis maximums. The utilization graph uses a fixed 0 to 100% scale.

## Dashboard

Open the dashboard from:

```text
Monitoring → Hosts → UDM-PRO → Dashboards
```

Select:

```text
UDM Pro - Network overview
```

The titles `WAN1`, `WAN2`, `WAN1 traffic` and `WAN2 traffic` are deliberately static. Zabbix 8.0 does not expand user macros in dashboard-widget titles.

The host macros `{$WAN1.ISP}` and `{$WAN2.ISP}` are still expanded in:

- item names;
- graph legends;
- trigger and problem names;
- Latest data.

Users who need ISP names in widget titles can manually rename the four dashboard widgets after importing the template.

## Security recommendations

- Never store a real API key or SNMP community directly in the template YAML.
- Configure `{$UNIFI.API.KEY}` and `{$SNMP_COMMUNITY}` as host-level **Secret text** macros.
- Use a dedicated monitoring API key.
- Restrict HTTPS and SNMP access to the Zabbix server or Zabbix proxy with firewall rules.
- Rotate any key that was displayed in a terminal recording, screenshot, chat or committed Git history.
- Do not publish production exports containing host-level macros.
- Review raw JSON history because client responses contain device names, IP addresses and MAC addresses.
- Replace the UDM Pro self-signed certificate with a trusted certificate when possible.

## Known limitations

- SNMP `ifOperStatus=up` confirms the Ethernet or SFP link, not end-to-end ISP connectivity.
- Separate Internet tests for each WAN require policy routing or another mechanism that forces probes through a specific ISP.
- WAN SNMP indices are environment-specific.
- The UniFi API can change between UniFi Network releases.
- The client query uses `limit=200`; environments with more than 200 concurrent clients may require a larger API limit or pagination.
- The UDM Pro API-reported IP can be an internal platform address rather than the management IP configured in `{$UNIFI.HOST}`.
- TLS certificate verification is disabled in the original HTTP-agent design for compatibility with the default UDM Pro certificate. Review this setting for production environments.

## Troubleshooting

### HTTP items show “Could not resolve host”

Check that `{$UNIFI.HOST}` contains only an IP address or DNS name:

```text
Correct:   192.168.1.1
Incorrect: https://192.168.1.1
```

### API returns HTTP 401

Verify that:

- the header is `X-API-KEY`;
- `{$UNIFI.API.KEY}` contains a valid local UniFi Network API key;
- Bearer authentication is not being used;
- the key has not been deleted or rotated.

### API returns HTTP 404

Verify:

- `{$UNIFI.SITE.ID}`;
- `{$UNIFI.DEVICE.ID}`;
- the local API paths available in **UniFi Network → Integrations → API Documentation**.

### SNMP items are unsupported

Check:

- the host has an SNMP interface;
- UDP/161 is reachable;
- the community is configured as `{$SNMP_COMMUNITY}` on the SNMP interface;
- `{$WAN1.IFINDEX}` and `{$WAN2.IFINDEX}` are correct;
- SNMP is enabled on UDM Pro.

Test from the Zabbix server:

```bash
snmpget -v2c -c COMMUNITY UDM_ADDRESS \
  1.3.6.1.2.1.2.2.1.8.IFINDEX
```

### WAN traffic graphs are empty

- Wait for at least two polling cycles.
- Confirm that high-capacity octet counters increase.
- Verify that the graph Y-axis maximum is **Calculated**, not fixed to `100`.
- Confirm the items use the unit `bps`.

### Utilization percentage looks different from the current traffic rate

Calculated utilization items and SNMP traffic items may be evaluated at slightly different times. Compare source item timestamps. Matching both intervals to 30 seconds reduces the visible difference.

### Dashboard displays WAN1 and WAN2 instead of ISP names

This is intentional. Zabbix does not expand user macros in widget titles. ISP names are displayed in graph legends, item names and problems. Rename the widgets manually if necessary.

## Versioning

The first public release is recommended as:

```text
v1.0.0
```

Suggested semantic versioning:

- `v1.0.1`: bug fix;
- `v1.1.0`: backward-compatible metrics or dashboard additions;
- `v2.0.0`: incompatible macro, key or structure changes.

## Contributing

Bug reports and pull requests are welcome. When reporting an issue, include:

- Zabbix version;
- UniFi Network version;
- UDM Pro firmware version;
- affected item key or trigger;
- exact Zabbix error text;
- sanitized API response structure when relevant.

Remove all API keys, SNMP communities, public IP addresses, private hostnames, MAC addresses and client details before posting logs or screenshots.

## License

This project is distributed under the **MIT License**. See `LICENSE`.
