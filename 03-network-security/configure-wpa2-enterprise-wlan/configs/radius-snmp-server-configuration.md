# RADIUS/SNMP Server Configuration

## Server Network Settings

| Setting | Value |
|---|---:|
| Device | `RADIUS/SNMP Server` |
| IP Address | `172.31.1.254/24` |
| Default Gateway | `172.31.1.1` |

## RADIUS Service

The server provides centralized 802.1X authentication for the `SSID-5` WLAN.

| Setting | Value |
|---|---:|
| RADIUS Client | `WLC-1` |
| WLC Address | `192.168.200.254` |
| Authentication Port | `1812/UDP` |
| Shared Secret | `Cisco123` |
| User Account | `user1` |
| User Password | `User1Pass` |

The credentials and shared secret are training-only values supplied with the Packet Tracer activity.

## SNMP Service

WLC-1 sends SNMP traps to this server at `172.31.1.254` using community string `WLAN_SNMP`.
