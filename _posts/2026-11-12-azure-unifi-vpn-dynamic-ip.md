---
layout: post
title: "Building an Azure Site-to-Site VPN with a UniFi Cloud Gateway and a Dynamic Public IP"
date: 2026-11-12 15:00:00 +0100
categories: [Azure, Networking]
tags: [Azure VPN Gateway, UniFi, IPsec, DDNS, FreeDNS]
description: "How to build and troubleshoot a resilient IPsec Site-to-Site VPN when the UniFi peer uses a dynamically assigned public IPv4 address."
---

## Introduction

A Site-to-Site VPN between a UniFi Cloud Gateway and Azure VPN Gateway is straightforward when both peers have static public IP addresses. The design becomes more interesting when the UniFi side uses PPPoE and receives a dynamically assigned public IPv4 address.

The objective of this lab was to ensure that the VPN could recover automatically after the ISP changed the public IP, without manually updating Azure.

The solution combines:

- UniFi Dynamic DNS
- FreeDNS / Afraid.org
- Azure Local Network Gateway configured with an FQDN
- IPsec/IKE Site-to-Site VPN

> All hostnames, addresses and account identifiers in this article are examples. Never publish DDNS passwords, update keys or production VPN secrets.

## Architecture

```text
Branch LAN
    |
UniFi Cloud Gateway
Dynamic PPPoE public IP
    |
    +---- DDNS ----> FreeDNS / Afraid.org
    |                    |
    |                 vpn.example.com
    |                    |
    +------ IPsec / IKE -+
                         |
                 Azure VPN Gateway
                         |
                     Azure VNet
```

The FQDN is the stable abstraction between the dynamic ISP address and Azure.

## Prerequisites

- A UniFi Cloud Gateway with a real public IPv4 address on the WAN interface
- An Azure VPN Gateway
- A FreeDNS/Afraid account and an A record
- Matching IPsec/IKE parameters and pre-shared key on both sides
- The on-premises and Azure address spaces must not overlap

If the WAN address belongs to an RFC1918 range or `100.64.0.0/10`, investigate upstream NAT or CGNAT before continuing.

## Configure Dynamic DNS on UniFi

In the UniFi Network application, open the WAN settings and configure Dynamic DNS:

```text
Service:   Afraid
Hostname:  vpn.example.com
Username:  <FreeDNS UserID>
Password:  <FreeDNS password>
Server:    empty
```

### The important detail: use the FreeDNS UserID

The first configuration used the account e-mail address as the username. Web sign-in worked, but DDNS updates failed.

FreeDNS exposes a separate **UserID** in the account preferences. In this environment, the UniFi/Inadyn integration required that UserID rather than the e-mail address.

## Inspect the DDNS client on the UCG

UniFi uses Inadyn underneath. Connect to the gateway over SSH and check the process:

```bash
ps -ax | grep inadyn
```

For a PPPoE WAN, the process referenced a generated configuration similar to:

```text
/run/ddns-ppp0-inadyn.conf
```

Inspect it carefully:

```bash
cat /run/ddns-ppp0-inadyn.conf
```

Conceptually, the generated configuration looked like this:

```text
iface = ppp0

provider afraid:1 {
    hostname = "vpn.example.com"
    username = "<FreeDNS UserID>"
    password = "<password>"
}
```

> This file can expose the DDNS password in clear text. Redact the output before sharing screenshots or logs.

## Troubleshoot authentication

Review recent Inadyn events:

```bash
journalctl | grep -i inadyn | tail -50
```

The initial failure was explicit:

```text
Update forced for alias vpn.example.com, new IP# 203.0.113.10
=> ERROR: Could not authenticate.
```

This proved that:

- Inadyn was running
- The PPPoE public IP was detected correctly
- The update was being triggered
- The failure was isolated to authentication

After replacing the e-mail address with the FreeDNS UserID, the log changed to:

```text
Update forced for alias vpn.example.com, new IP# 203.0.113.10
Updating IPv4 cache for vpn.example.com
```

No custom update URL was required in the UniFi configuration.

## Verify DDNS

Check the current PPPoE address on the UCG:

```bash
ip -4 addr show ppp0
```

Then resolve the FQDN from another system.

PowerShell:

```powershell
Resolve-DnsName vpn.example.com
```

Linux:

```bash
dig +short vpn.example.com
```

The returned A record must match the public IPv4 address assigned to `ppp0`.

## Configure Azure with the FQDN

Create the Azure Local Network Gateway using the DDNS hostname as its endpoint:

```text
Endpoint type: FQDN
FQDN:         vpn.example.com
Address space: <branch LAN prefixes>
```

Create the Site-to-Site connection between the Azure VPN Gateway and the Local Network Gateway, using the same pre-shared key and compatible IKE/IPsec parameters configured on UniFi.

## Validate recovery after an IP change

The definitive test is a real WAN reconnection or gateway reboot.

Monitor Inadyn:

```bash
journalctl -f | grep -i inadyn
```

After the PPPoE address changed, the expected log appeared:

```text
Update needed for alias vpn.example.com, new IP# 203.0.113.27
Updating IPv4 cache for vpn.example.com
```

The validation sequence was:

1. Confirm the new IPv4 address on `ppp0`.
2. Confirm the Inadyn update completed without an authentication error.
3. Resolve the FQDN and confirm that it returned the new address.
4. Check that the UniFi Site-to-Site VPN returned to **Established**.
5. Check the Azure VPN connection status.
6. Repeat the reboot test to confirm the recovery was not accidental.

The tunnel recovered successfully after two consecutive reboots. Recovery was not instantaneous because the sequence includes PPPoE establishment, DDNS update, DNS refresh and IKE/IPsec renegotiation.

## Troubleshooting runbook

### 1. Check the WAN address

```bash
ip -4 addr show ppp0
```

### 2. Confirm Inadyn is running

```bash
ps -ax | grep inadyn
```

### 3. Inspect the generated configuration

```bash
cat /run/ddns-ppp0-inadyn.conf
```

Verify the interface, provider, hostname and UserID. Protect the password.

### 4. Review or follow the logs

```bash
journalctl | grep -i inadyn | tail -50
```

```bash
journalctl -f | grep -i inadyn
```

If the output contains `Could not authenticate`, verify the FreeDNS UserID, password and hostname.

### 5. Compare WAN IP and public DNS

```powershell
Resolve-DnsName vpn.example.com
```

The result must match the WAN address.

### 6. If DDNS is correct but the VPN is down

Move the investigation to IPsec and Azure:

- Local Network Gateway endpoint and address prefixes
- Pre-shared key
- IKE/IPsec policy compatibility
- Azure VPN Gateway public IP configured as the UniFi remote peer
- Traffic selectors and non-overlapping address spaces
- Firewall rules
- IKE negotiation logs and connection status

## Lessons learned

The key to troubleshooting this design is to validate each layer separately:

```text
PPPoE address
   -> Inadyn
   -> FreeDNS update
   -> Public DNS resolution
   -> Azure FQDN endpoint
   -> IKE/IPsec negotiation
   -> Tunnel established
```

Dynamic DNS is the abstraction that allows Azure to keep referring to the same peer name even when the ISP changes the underlying public IP address.
