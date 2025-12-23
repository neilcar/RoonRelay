# Roon Relay - Configuration Examples

## Example 1: Single Site with 1 Relay + 1 Roadwarrior VPN

### Network Diagram

```
                                    INTERNET
                                        │
                                        │
                              ┌─────────┴─────────┐
                              │    OPNsense       │
                              │   172.16.0.1      │
                              │                   │
                              │  WG Roadwarrior   │
                              │  10.10.99.1/24    │
                              └─────────┬─────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    │           LAN 172.16.0.0/24           │
                    │                   │                   │
           ┌────────┴────────┐ ┌────────┴────────┐ ┌────────┴────────┐
           │  Roon Server    │ │   VM Relay      │ │  Other clients  │
           │  172.16.0.106   │ │  172.16.0.108   │ │  172.16.0.x     │
           └─────────────────┘ └─────────────────┘ └─────────────────┘


                              ┌─────────────────┐
                              │ VPN RW Client   │
                              │ (smartphone)    │
                              │ 10.10.99.3      │
                              │                 │
                              │ AllowedIPs:     │
                              │ 10.10.99.0/24   │
                              │ 172.16.0.0/24   │
                              │ 255.255.255.255 │
                              └─────────────────┘
```

### Components

| Component | IP | Role |
|-----------|-----|------|
| OPNsense | 172.16.0.1 / 10.10.99.1 | Firewall + WG server |
| Roon Server | 172.16.0.106 | Roon Server |
| VM Relay | 172.16.0.108 | Broadcast relay |
| RW Client | 10.10.99.3 | Smartphone via VPN |

### OPNsense Configuration

**NAT Port Forward (Firewall → NAT → Port Forward):**
- Interface: WireGuard RW
- Protocol: UDP
- Source: 10.10.99.0/24
- Destination: 255.255.255.255
- Destination port: 9003
- Redirect target IP: 172.16.0.108
- Redirect target port: 9003

### appsettings.json

```json
{
  "SiteName": "MainRelay",
  "TunnelPort": 9004,
  "RemoteRelayIp": "",
  "LocalInterfaces": [
    {
      "LocalIp": "172.16.0.108",
      "BroadcastAddress": "172.16.0.255",
      "SubnetMask": "255.255.255.0"
    }
  ],
  "UnicastTargets": []
}
```

---

## Example 2: Site-to-Site

### Network Diagram

```
        SITE A                                                    SITE B
        ══════                                                    ══════

      INTERNET                                                  INTERNET
          │                                                         │
          │                                                         │
┌─────────┴─────────┐                                   ┌───────────┴─────────┐
│    OPNsense A     │                                   │     OPNsense B      │
│   172.16.0.1      │                                   │    192.168.30.1     │
│                   │         WireGuard S2S             │                     │
│  WG S2S           │◄─────────────────────────────────►│  WG S2S             │
│  10.10.90.1/30    │         encrypted tunnel          │  10.10.90.2/30      │
│                   │                                   │                     │
│  WG RW            │                                   │  WG RW              │
│  10.10.99.1/24    │                                   │  10.10.98.1/24      │
└─────────┬─────────┘                                   └───────────┬─────────┘
          │                                                         │
          │ LAN 172.16.0.0/24                                       │ LAN 192.168.30.0/24
          │                                                         │
    ┌─────┴─────┐                                             ┌─────┴─────┐
    │           │                                             │           │
┌───┴───┐   ┌───┴───┐                                     ┌───┴───┐   ┌───┴───┐
│ Roon  │   │Relay A│                                     │Relay B│   │Client │
│Server │   │  VM   │◄ ─ ─ ─ ─ tunnel 9004 ─ ─ ─ ─ ─ ─ ─ ►│  VM   │   │  LAN  │
│.106   │   │.108   │                                     │.40    │   │.30    │
└───────┘   └───────┘                                     └───────┘   └───────┘


┌─────────────────┐                                       ┌─────────────────┐
│ VPN RW Client A │                                       │ VPN RW Client B │
│ 10.10.99.3      │                                       │ 10.10.98.3      │
│                 │                                       │                 │
│ AllowedIPs:     │                                       │ AllowedIPs:     │
│ 10.10.99.0/24   │                                       │ 10.10.98.0/24   │
│ 172.16.0.0/24   │                                       │ 192.168.30.0/24 │
│ 255.255.255.255 │                                       │ 172.16.0.0/24   │
└─────────────────┘                                       │ 255.255.255.255 │
                                                          └─────────────────┘
```

### Components

| Component | IP | Site | Role |
|-----------|-----|------|------|
| OPNsense A | 172.16.0.1 / 10.10.90.1 / 10.10.99.1 | A | Firewall |
| Roon Server | 172.16.0.106 | A | Roon Server |
| VM Relay A | 172.16.0.108 | A | Relay |
| OPNsense B | 192.168.30.1 / 10.10.90.2 / 10.10.98.1 | B | Firewall |
| VM Relay B | 192.168.30.40 | B | Relay |
| LAN Client B | 192.168.30.30 | B | Roon endpoint |
| RW Client A | 10.10.99.3 | A | VPN client |
| RW Client B | 10.10.98.3 | B | VPN client |

### WireGuard S2S Configuration

**Site A - Peer:**
```
Allowed IPs: 10.10.90.2/32, 192.168.30.0/24, 10.10.98.0/24
Endpoint: <PUBLIC_IP_B>:51820
```

**Site B - Peer:**
```
Allowed IPs: 10.10.90.1/32, 172.16.0.0/24, 10.10.99.0/24
Endpoint: <PUBLIC_IP_A>:51820
```

### appsettings.json - Relay A (Site A)

```json
{
  "SiteName": "RelayA",
  "TunnelPort": 9004,
  "RemoteRelayIp": "192.168.30.40",
  "LocalInterfaces": [
    {
      "LocalIp": "172.16.0.108",
      "BroadcastAddress": "172.16.0.255",
      "SubnetMask": "255.255.255.0"
    }
  ],
  "UnicastTargets": []
}
```

### appsettings.json - Relay B (Site B)

```json
{
  "SiteName": "RelayB",
  "TunnelPort": 9004,
  "RemoteRelayIp": "172.16.0.108",
  "LocalInterfaces": [
    {
      "LocalIp": "192.168.30.40",
      "BroadcastAddress": "192.168.30.255",
      "SubnetMask": "255.255.255.0"
    }
  ],
  "UnicastTargets": []
}
```

---

## Example 3: Complex Multi-VLAN Configuration

### Network Diagram

```
                    SITE A                                                          SITE B
                    ══════                                                          ══════

                  INTERNET                                                        INTERNET
                      │                                                               │
                      │                                                               │
          ┌───────────┴───────────┐                                     ┌─────────────┴───────────┐
          │      OPNsense A       │                                     │       OPNsense B        │
          │                       │                                     │                         │
          │  LAN:    172.16.0.1   │         WireGuard S2S               │  LAN:    192.168.30.1   │
          │  VLAN100:192.168.100.1│◄───────────────────────────────────►│  VLAN100:192.168.99.1   │
          │  WG_S2S: 10.10.90.1   │         10.10.90.0/30               │  WG_S2S: 10.10.90.2     │
          │  WG_RW:  10.10.99.1   │                                     │  WG_RW:  10.10.98.1     │
          └───────────┬───────────┘                                     └─────────────┬───────────┘
                      │                                                               │
      ┌───────────────┼───────────────┐                           ┌───────────────────┼───────────────┐
      │               │               │                           │                   │               │
      │ LAN           │ VLAN 100      │                           │ LAN               │ VLAN 100      │
      │ 172.16.0.0/24 │192.168.100/24 │                           │ 192.168.30.0/24   │192.168.99/24  │
      │               │               │                           │                   │               │
┌─────┴─────┐   ┌─────┴─────┐   ┌─────┴─────┐               ┌─────┴─────┐       ┌─────┴─────┐   ┌─────┴─────┐
│   Roon    │   │ Relay A   │   │  Client   │               │ Relay B   │       │  Client   │   │  Client   │
│  Server   │   │    VM     │   │ VLAN100   │               │    VM     │       │   LAN     │   │ VLAN100   │
│           │   │           │   │           │               │           │       │           │   │           │
│172.16.0   │   │172.16.0   │   │192.168    │               │192.168.30 │       │192.168.30 │   │192.168.99 │
│  .106     │   │  .108     │   │  .100.5   │               │   .40     │       │   .30     │   │   .50     │
│           │   │192.168    │   │ (EndP2)   │               │192.168.99 │       │ (EndP3)   │   │           │
│           │   │ .100.100  │   │           │               │  .100     │       │           │   │           │
└───────────┘   └───────────┘   └───────────┘               └───────────┘       └───────────┘   └───────────┘


        ┌─────────────────┐                                         ┌─────────────────┐
        │ VPN RW Client A │                                         │ VPN RW Client B │
        │    (EndP1)      │                                         │    (EndP5)      │
        │   10.10.99.5    │                                         │   10.10.98.5    │
        │                 │                                         │                 │
        │ AllowedIPs:     │                                         │ AllowedIPs:     │
        │ 10.10.99.0/24   │                                         │ 10.10.98.0/24   │
        │ 172.16.0.0/24   │                                         │ 192.168.30.0/24 │
        │ 192.168.100/24  │                                         │ 192.168.99.0/24 │
        │ 255.255.255.255 │                                         │ 172.16.0.0/24   │
        └─────────────────┘                                         │ 255.255.255.255 │
                                                                    └─────────────────┘


        ┌─────────────────┐
        │ LAN Client A    │
        │    (EndP4)      │
        │  172.16.0.16    │
        │                 │
        │ (no VPN needed) │
        └─────────────────┘
```

### Components

| Component | IP | Site | Subnet | Role |
|-----------|-----|------|--------|------|
| OPNsense A | 172.16.0.1 | A | LAN | Firewall |
| OPNsense A | 192.168.100.1 | A | VLAN100 | Firewall |
| OPNsense A | 10.10.90.1 | A | WG_S2S | Tunnel endpoint |
| OPNsense A | 10.10.99.1 | A | WG_RW | VPN server |
| Roon Server | 172.16.0.106 | A | LAN | Roon Server |
| VM Relay A | 172.16.0.108, 192.168.100.100 | A | LAN + VLAN100 | Multi-homed relay |
| OPNsense B | 192.168.30.1 | B | LAN | Firewall |
| OPNsense B | 192.168.99.1 | B | VLAN100 | Firewall |
| OPNsense B | 10.10.90.2 | B | WG_S2S | Tunnel endpoint |
| OPNsense B | 10.10.98.1 | B | WG_RW | VPN server |
| VM Relay B | 192.168.30.40, 192.168.99.100 | B | LAN + VLAN100 | Multi-homed relay |

### Endpoints Summary

| Endpoint | IP | Location | Works | Notes |
|----------|-----|----------|-------|-------|
| EndP1 | 10.10.99.5 | WG_RW Site A | ✓ | NAT 255.255.255.255 → Relay A |
| EndP2 | 192.168.100.5 | VLAN100 Site A | ✓ | Relay A has interface on VLAN100 |
| EndP3 | 192.168.30.30 | LAN Site B | ✓ | Via S2S tunnel + Relay B |
| EndP4 | 172.16.0.16 | LAN Site A | ✓ | Same broadcast domain as server |
| EndP5 | 10.10.98.5 | WG_RW Site B | ✓ | NAT → Relay B → tunnel → Relay A |

### WireGuard S2S Configuration

**Site A - Peer (to Site B):**
```
Allowed IPs: 10.10.90.2/32, 192.168.30.0/24, 192.168.99.0/24, 10.10.98.0/24
Endpoint: <PUBLIC_IP_B>:51820
```

**Site B - Peer (to Site A):**
```
Allowed IPs: 10.10.90.1/32, 172.16.0.0/24, 192.168.100.0/24, 10.10.99.0/24
Endpoint: <PUBLIC_IP_A>:51820
```

### NAT Port Forward

**Site A (for WG_RW):**
- Interface: WireGuard RW
- Source: 10.10.99.0/24
- Destination: 255.255.255.255:9003
- Redirect: 172.16.0.108:9003

**Site B (for WG_RW):**
- Interface: WireGuard RW
- Source: 10.10.98.0/24
- Destination: 255.255.255.255:9003
- Redirect: 192.168.30.40:9003

### appsettings.json - Relay A (Site A)

```json
{
  "SiteName": "RelayA",
  "TunnelPort": 9004,
  "RemoteRelayIp": "192.168.30.40",
  "LocalInterfaces": [
    {
      "LocalIp": "172.16.0.108",
      "BroadcastAddress": "172.16.0.255",
      "SubnetMask": "255.255.255.0"
    },
    {
      "LocalIp": "192.168.100.100",
      "BroadcastAddress": "192.168.100.255",
      "SubnetMask": "255.255.255.0"
    }
  ],
  "UnicastTargets": []
}
```

### appsettings.json - Relay B (Site B)

```json
{
  "SiteName": "RelayB",
  "TunnelPort": 9004,
  "RemoteRelayIp": "172.16.0.108",
  "LocalInterfaces": [
    {
      "LocalIp": "192.168.30.40",
      "BroadcastAddress": "192.168.30.255",
      "SubnetMask": "255.255.255.0"
    },
    {
      "LocalIp": "192.168.99.100",
      "BroadcastAddress": "192.168.99.255",
      "SubnetMask": "255.255.255.0"
    }
  ],
  "UnicastTargets": []
}
```

### Firewall Rules - Site A (WG_S2S)

| # | Proto | Source | Destination | Port | Description |
|---|-------|--------|-------------|------|-------------|
| 1 | UDP | 192.168.30.40 | 172.16.0.108 | 9004 | Relay tunnel |
| 2 | UDP | 192.168.30.0/24 | 172.16.0.0/24 | 9003 | Discovery from LAN B |
| 3 | UDP | 192.168.99.0/24 | 172.16.0.0/24 | 9003 | Discovery from VLAN100 B |
| 4 | UDP | 10.10.98.0/24 | 172.16.0.0/24 | 9003 | Discovery from RW B |
| 5 | TCP | 192.168.30.0/24 | 172.16.0.0/24 | 9100-9200 | TCP from LAN B |
| 6 | TCP | 192.168.30.0/24 | 172.16.0.0/24 | 9330-9332 | TCP API from LAN B |
| 7 | TCP | 192.168.99.0/24 | 172.16.0.0/24 | 9100-9200 | TCP from VLAN100 B |
| 8 | TCP | 192.168.99.0/24 | 172.16.0.0/24 | 9330-9332 | TCP API from VLAN100 B |
| 9 | TCP | 10.10.98.0/24 | 172.16.0.0/24 | 9100-9200 | TCP from RW B |
| 10 | TCP | 10.10.98.0/24 | 172.16.0.0/24 | 9330-9332 | TCP API from RW B |
| 11 | * | * | * | * | BLOCK all other |

### Firewall Rules - Site B (WG_S2S)

| # | Proto | Source | Destination | Port | Description |
|---|-------|--------|-------------|------|-------------|
| 1 | UDP | 172.16.0.108 | 192.168.30.40 | 9004 | Relay tunnel |
| 2 | UDP | 172.16.0.0/24 | 192.168.30.0/24 | 9003 | Discovery from LAN A |
| 3 | UDP | 172.16.0.0/24 | 192.168.99.0/24 | 9003 | Discovery to VLAN100 B |
| 4 | TCP | 172.16.0.0/24 | 192.168.30.0/24 | 9100-9200 | TCP to LAN B |
| 5 | TCP | 172.16.0.0/24 | 192.168.30.0/24 | 9330-9332 | TCP API to LAN B |
| 6 | TCP | 172.16.0.0/24 | 192.168.99.0/24 | 9100-9200 | TCP to VLAN100 B |
| 7 | TCP | 172.16.0.0/24 | 192.168.99.0/24 | 9330-9332 | TCP API to VLAN100 B |
| 8 | TCP | 172.16.0.0/24 | 10.10.98.0/24 | 9100-9200 | TCP to RW B |
| 9 | TCP | 172.16.0.0/24 | 10.10.98.0/24 | 9330-9332 | TCP API to RW B |
| 10 | * | * | * | * | BLOCK all other |

---

## Roon Ports Summary

| Port | Protocol | Purpose |
|------|----------|---------|
| 9003 | UDP | Discovery broadcast/multicast |
| 9004 | UDP | Tunnel between relays |
| 9100-9200 | TCP | Control and audio streaming |
| 9330-9332 | TCP | HTTP/HTTPS API |
