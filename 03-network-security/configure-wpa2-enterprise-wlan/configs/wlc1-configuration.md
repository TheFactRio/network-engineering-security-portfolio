# WLC-1 Configuration

## Management Interface

| Setting | Value |
|---|---:|
| IP Address | `192.168.200.254` |
| Netmask | `255.255.255.0` |
| Gateway | `192.168.200.1` |
| Primary DHCP Server | `192.168.200.254` |

## Dynamic Interface

| Setting | Value |
|---|---:|
| Interface Name | `WLAN-5` |
| VLAN ID | `5` |
| Physical Port | `1` |
| IP Address | `192.168.5.254` |
| Netmask | `255.255.255.0` |
| Gateway | `192.168.5.1` |
| Primary DHCP Server | `192.168.5.1` |

## WLAN

| Setting | Value |
|---|---:|
| Profile Name | `Floor 2 Employees` |
| WLAN ID | `5` |
| SSID | `SSID-5` |
| Status | Enabled |
| Interface | `WLAN-5` |
| FlexConnect Local Switching | Enabled |
| FlexConnect Local Authentication | Enabled |

## Layer 2 Security

| Setting | Value |
|---|---:|
| Security Mode | `WPA+WPA2` |
| WPA2 Policy | Enabled |
| Encryption | `AES` |
| Authentication Key Management | `802.1X` |
| Authentication Server | `172.31.1.254:1812` |

## Internal DHCP Scope

| Setting | Value |
|---|---:|
| Scope Name | `Wired Management` |
| Pool Start | `192.168.200.240` |
| Pool End | `192.168.200.249` |
| Network | `192.168.200.0` |
| Netmask | `255.255.255.0` |
| Default Router | `192.168.200.1` |
| Status | Enabled |

## SNMP Trap Receiver

| Setting | Value |
|---|---:|
| Community | `WLAN_SNMP` |
| Receiver Address | `172.31.1.254` |
| Status | Enabled |
