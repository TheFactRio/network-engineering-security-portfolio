# AAA Local and RADIUS Authentication

## 📌Overview

This lab demonstrates how to secure administrative access to Cisco routers in stages: local passwords on the console, aux and vty lines, local user accounts, AAA with the local database, and finally centralized authentication against a RADIUS server.

The lab is based on a Cisco NetAcad AAA/RADIUS exercise and was adapted to GNS3(2.2.61):

- Routers: Cisco IOSv (IOS 15.9)
- Switches: Cisco IOSvL2 (IOS 15.2)
- RADIUS server: FreeRADIUS 3.0 on Ubuntu Server(24.04.5 LTS), which replaces the Windows WinRadius host from the original lab
- Client PC: VPCS

R2 acts as an ISP router. It also provides NAT so the Ubuntu server can reach the internet to install packages.

## 🎯Objectives

- Configure basic device settings, static routing and password protection
- Configure local authentication for console, aux and vty lines
- Configure AAA local authentication with a method list for Telnet
- Observe AAA authentication with `debug aaa authentication`
- Install and configure FreeRADIUS on Ubuntu Server
- Configure R1 to authenticate users against the RADIUS server
- Verify successful and failed RADIUS logins on the router, in the FreeRADIUS debug output and in Wireshark

## Topology

![Topology](topology.png)

## 📋Addressing Table

| Device | Interface | IP Address | Subnet Mask | Connected To / Purpose |
|---|---|---|---|---|
| R1 | G0/0 | 10.1.1.1 | 255.255.255.252 | R2 G0/1 |
| R1 | G0/1 | 192.168.1.1 | 255.255.255.0 | R1 LAN (SW1) |
| R2 | G0/0 | DHCP (192.168.42.159 during the lab) | 255.255.255.0 | GNS3 NAT node (upstream), NAT outside  |
| R2 | G0/1 | 10.1.1.2 | 255.255.255.252 | R1 G0/0, NAT inside |
| R2 | G0/2 | 10.2.2.2 | 255.255.255.252 | R3 G0/0, NAT inside |
| R3 | G0/0 | 10.2.2.1 | 255.255.255.252 | R2 G0/2 |
| R3 | G0/1 | 192.168.3.1 | 255.255.255.0 | R3 LAN (SW2) |
| Ubuntu Server (ubu-srv01) | ens3 | 192.168.1.3 | 255.255.255.0 | RADIUS server, gateway 192.168.1.1 |
| PC-1 (VPCS) | eth0 | 192.168.3.3 | 255.255.255.0 | R3 LAN host, gateway 192.168.3.1 |

RADIUS server: 192.168.1.3, UDP 1812 (authentication) and 1813 (accounting).

## ⚙️Configuration Summary

### R1

- Hostname, interface addressing, default route to R2 (`ip route 0.0.0.0 0.0.0.0 10.1.1.2`)
- `security passwords min-length 10`, `enable secret` with type 9 (scrypt)
- Console, aux and vty passwords, `exec-timeout 5 0`, `service password-encryption`, MOTD banner
- Local fallback account `LocalAdmin` (privilege 15, type 9 secret)
- `aaa new-model`
- RADIUS server `CCNAS` at 192.168.1.3 with `auth-port 1812 acct-port 1813`
- `aaa authentication login default group radius local`
- `aaa authentication login TELNET_LINES group radius local`, applied to `line vty 0 4` with `transport input telnet`

The previous local lab account `user01` and `login local` were removed from R1 before the RADIUS part to keep the configuration clean.

### R2 (ISP)

- Interface addressing, DHCP on G0/0
- Default route to the upstream gateway and static routes to 192.168.1.0/24 and 192.168.3.0/24
- NAT overload on G0/0 using standard ACL 1 (both LANs and both transit links)

### R3

- Same basic security as R1
- Local users `user01` and `Admin01` (type 9 secrets)
- `aaa new-model`
- `aaa authentication login default local-case none`
- `aaa authentication login TELNET_LINES local`, applied to `line vty 0 4`

### Ubuntu Server

- Static IP in netplan, DNS 8.8.8.8 and 1.1.1.1
- FreeRADIUS installed (`freeradius`, `freeradius-utils`, `freeradius-ldap`)
- `clients.conf`: client `R1` (192.168.1.1, `nastype = cisco`)
- `mods-config/files/authorize`: user `RadUser`

All passwords and shared secrets in this repository are lab-only credentials.

## ✅Verification

### Basic connectivity

R1, R3 and R2 interfaces are up with the addressing from the table:

![R1 interfaces](screenshots/01-r1-interfaces.png)
![R3 interfaces](screenshots/02-r3-interfaces.png)

R2 learned its upstream address by DHCP and has a default route plus static routes to both LANs:

![R2 interfaces and routes](screenshots/03-r2-interfaces-routes.png)

R1 reaches the internet and R2 translates inside addresses with NAT:

![R2 NAT and internet](screenshots/04-r2-nat-internet.png)

End-to-end connectivity between the two LANs:

![R1 ping R3](screenshots/05-r1-ping-r3.png)
![PC-1 ping Ubuntu](screenshots/06-pc1-ping-ubuntu.png)

### Basic device security

Minimum password length, scrypt enable secret, service password-encryption and the MOTD banner on R1. Line passwords are stored as type 7 because of `service password-encryption`:

![R1 basic security](screenshots/07-r1-basic-security.png)

### Local users and local AAA (R3)

Local users are stored as type 9 hashes and cannot be read in the running configuration:

![R3 local users](screenshots/08-r3-local-users.png)

Console login with the default list `local-case none`. A valid local user is authenticated from the local database:

![R3 console Admin01](screenshots/09-r3-console-admin01.png)

An unknown user also gets in, because the `none` method allows access when the username is not found in the local database. This is the weakness of using `none` as a fallback:

![R3 console baduser](screenshots/10-r3-console-baduser.png)

Telnet uses the separate list `TELNET_LINES` with no `none` fallback. Admin01 logs in, while an unknown user is rejected:

![R3 Telnet Admin01](screenshots/11-r3-telnet-admin01.png)
![R3 Telnet baduser](screenshots/12-r3-telnet-baduser-fail.png)

AAA debug output shows the method list `TELNET_LINES`, then PASS for the correct enable secret and FAIL for a wrong one:

![R3 debug aaa authentication](screenshots/13-r3-debug-aaa.png)

### FreeRADIUS server

Static addressing, default route and DNS on the Ubuntu server:

![Ubuntu network](screenshots/14-ubuntu-network.png)

FreeRADIUS client and user definitions:

![FreeRADIUS configuration](screenshots/15-ubuntu-freeradius-config.png)

Configuration syntax check passes, the service is running and listens on UDP 1812 and 1813:

![FreeRADIUS service](screenshots/16-ubuntu-freeradius-service.png)

Local test with `radtest` returns Access-Accept:

![radtest](screenshots/17-ubuntu-radtest-local.png)

### R1 AAA and RADIUS configuration

![R1 AAA RADIUS configuration](screenshots/18-r1-aaa-radius-config.png)

R1 reaches the RADIUS server:

![R1 ping RADIUS](screenshots/19-r1-ping-radius.png)

### RADIUS login tests on R1

Each attempt is shown from three points of view: the R1 login session, the FreeRADIUS debug output (`freeradius -X`) and a Wireshark capture filtered with `radius`. The R1 sessions were opened with PuTTY using the Telnet protocol.

**Attempt 1: valid user (RadUser / RadUserpass).** R1 sends an Access-Request and FreeRADIUS answers with Access-Accept. The user logs in.

![Attempt 1 R1](screenshots/20-attempt1-valid-r1.png)
![Attempt 1 FreeRADIUS](screenshots/21-attempt1-valid-freeradius.png)
![Attempt 1 Wireshark](screenshots/22-attempt1-valid-wireshark.png)

**Attempt 2: nonexistent user (NoExistingUser / 13435).** FreeRADIUS does not know the user and returns Access-Reject. R1 prints `% Authentication failed`. The `none` method from the original lab is not used here, so an unknown user is not let in.

![Attempt 2 R1](screenshots/23-attempt2-nouser-r1.png)
![Attempt 2 FreeRADIUS](screenshots/24-attempt2-nouser-freeradius.png)
![Attempt 2 Wireshark](screenshots/25-attempt2-nouser-wireshark.png)

**Attempt 3: valid user, wrong password (RadUser / badpass).** FreeRADIUS returns Access-Reject and R1 denies access. A reject from the server does not trigger the local fallback.

![Attempt 3 R1](screenshots/26-attempt3-badpass-r1.png)
![Attempt 3 FreeRADIUS](screenshots/27-attempt3-badpass-freeradius.png)
![Attempt 3 Wireshark](screenshots/28-attempt3-badpass-wireshark.png)

In the Wireshark captures the User-Password attribute is encrypted with the shared secret. Only the username and the Access-Request/Accept/Reject flow are visible.

### RADIUS-down fallback

FreeRADIUS was stopped on the Ubuntu server:

![FreeRADIUS stopped](screenshots/29-ubuntu-freeradius-stopped.png)

With no reply from the server, R1 falls back to the local database. The local account `LocalAdmin` logs in after a delay, and `show aaa servers` and `show users` show the session:

![LocalAdmin fallback](screenshots/30-r1-localadmin-fallback.png)

A RADIUS-only user (RadUser) cannot log in while the server is down, because it has no local account.

![RADIUS down fallback](screenshots/31-r1-radius-down-fallback.png)

## 🛠️Troubleshooting Notes

### RADIUS ports

Cisco IOS defaults to UDP 1645/1646, while FreeRADIUS listens on 1812/1813. The ports are set explicitly: `address ipv4 192.168.1.3 auth-port 1812 acct-port 1813`.

### Ubuntu DNS

 Package installation needed working DNS, so nameservers were added in netplan.

### Ubuntu SSH (convenience only)

OpenSSH was installed. Legacy `diffie-hellman-group14-sha1` and `ssh-rsa` were enabled for old IOS clients.

## 🧠Lessons Learned

- Line passwords alone give no accountability. Local users add usernames, and AAA adds flexible method lists.
- A method list that ends with `none` lets any user in if the earlier methods cannot find the account. It should not be used in production.
- With RADIUS, an Access-Reject ends authentication. Fallback to the local database happens only when the server is unreachable.
- Keeping a local privileged account protects against lockout when RADIUS is down. Testing that fallback matters.
- Router and server must agree on ports (1812/1813) and on the shared secret. A mismatch looks like a dead server.
- Comparing the router console, `freeradius -X` and Wireshark makes it easy to see where an authentication failure happens.

## 📁Files

| File | Description |
|---|---|
| `topology.png` | Network topology |
| `gns3/aaa-local-and-radius-authentication.gns3` | GNS3 project file |
| `configs/R1-config.txt` | Final R1 running configuration |
| `configs/R2-config.txt` | Final R2 running configuration |
| `configs/R3-config.txt` | Final R3 running configuration |
| `configs/ubuntu-freeradius/` | FreeRADIUS client and user snippets, netplan file |
| `screenshots/` | Verification screenshots |