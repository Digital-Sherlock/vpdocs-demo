![ZTNA topology](../assets/ZTNA%20topology.png)

There are two types of ZTNA connections – TCP Forwarding Access Proxy and HTTPS Access Proxy – and FortiClient treats them differently which leads to different troubleshooting methodologies.

First, remember the difference between them and how to access TFAP and HTTPS Access Proxy destinations from FortiClient.

![](../assets/tfap%20vs%20https.png)

TFAP destination is accessed via **Destination Host** – RDP into machine using win.rdp.fortitest.net address - while HTTPS Access proxy destination via **Proxy Gateway** – navigating to webserver.fortitest.net:8888 URL.

## Step 1 – TFAP vs HTTPS Access Proxy.

Figure out what method is used and make sure destination is accessed the way it’s supposed to.

## Step 2 – Collect logs and backup files.

Request FortiClient diagnostics and backup as well as FortiGate configuration file. Verify the setup.

## Step 3 – Analyze the logs.

FortiClient has two log files for ZTNA – {==fortitcs==} (ZTNA connections), {==transctrl==} (DNS for ZTNA FQDN-based destinations).

Start with fortitcs and identify ZTNA connection attempts. Here's how it might look like:

``` bash
[2024-10-31 09:20:49.4462239] [fortitcs] SAML address: https://172.31.200.244:9833/tcp?address=win.rdp.fortitest.net&port=3389&tls=1
[2024-10-31 09:20:49.4462429] [fortitcs] Cache address: https://172.31.200.244:9833 # (1)
[2024-10-31 09:20:49.4462695] [fortitcs] GET /tcp?address=in.rdp.fortitest.net&port=3389&tls=1 HTTP/1.1 # (2)
Host: 172.31.200.244:9833
User-Agent: Forticlient
Accept: */*
Upgrade: tcp-forwarding/1.0
Connection: Upgrade
Cookie:
Authorization: Basic
```

1. Proxy Gateway address. Commonly, public-facing FortiGate interface.
2. Destinaion host address commonly referred to as Real Server.

Based on the output, proceed further.

Common issues are:

- ZTNA gateway unreachable (network issues, FortiGate misconfiguration)
- ZTNA certificate issues
- Policy denials (has to be verified further on FortiGate)
- EMS Telemetry and Fabric Connector issues (common error is “endpoint is offline”)

!!! example "New Feature"
    New feature promising to deliver more failproof architecture to address Fabric Connector issues (i.e. EMS downtime) - [Tag sharing via JWT](https://docs.fortinet.com/document/forticlient/7.4.0/new-features/592722/jwt-support-for-ztna-uid-and-tag-sharing). 

Let's break them down below.

## Common issues with connecting to ZTNA gateway

### ZTNA certificate issues

ZTNA certificate issues are covered in its own section - [ZTNA Certificate](../ztna_certificate).

### Policy denials

Obviously, policy denials happen on FortiGate, hence, troubleshooting has to start there and specifically with ZTNA traffic logs.

Commonly, if ZTNA policy is configured properly (src/dst interfaces, destination real server, etc.), the issue may be with Zero Trust tags. Namely, endpoint not matching a tag installed on a policy. Simple way to confirm? Copy problemaic policy, remove tags, and insert it above. If traffic starts flowing through, then verify tag exchange between FortiGate and EMS. Here's few command line utilities that may help.

{==FortiGate WAD diagnostics.==}

``` bash
diagnose debug reset
diagnose debug console timestamp enable

diagnose wad filter clear
diagnose wad filter src X.X.X.X # (1)
diagnose wad debug enable category all
diagnose wad debug enable level verbose

diagnose debug enable
```

1. Public IP of endpoint.

WAD diagnostics produce a significant output that may be troublesome to navigate around. Hence, try using filters like in the command above to decrease the output size. 

Refer to [FortiGate WAD diagnostics analysis](../wad_diags) case study for details on the WAD output.

### Fabric Connector issues

Fabric Connector issues are covered in its own section - [Fabric Connector issues](../fabric_connector).

## Extras

In case of ZTNA connection timing out or throwing DNS errors in the browser with fortitcs not providing any logs, verify {==transctrl.log==} that outputs ZTNA DNS-related entries. You may encounter DNS errors when FortiClient attempts to resolve FQND-based ZTNA destinations. I.e.:
```
[transctrl 655] DNS Error: flag = 0x8183, length = 48, 
[transctrl 727] cannot modify DNSNameError, because length is not enough!
```