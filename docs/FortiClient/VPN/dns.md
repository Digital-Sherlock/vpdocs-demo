# DNS Issues

When dealing with DNS issues, it’s critical to fully understand customer’s setup and how exactly DNS fail to work.

Here’s few examples of questions one might ask:

- What exactly about DNS isn’t working?
- Does DNS fail when on-, off-VPN or both?
- Does it fail to resolve private, public, or both domain names?

# Step 1 - Understand customer’s setup.

Good place to start is to request FortiGate and FortiClient configuration files as well as FortiClient diagnostics/logs. You probably want to know whether you’re dealing with regular or split-DNS setup, right?

Windows diagnostics will contain output from common networking commands like ipconfig that may be helpful while VPN logs will have networking configuration pushed down from FortiGate which includes DNS configuration.

See example of SSLVPN log [^1] showing XML-formatted VPN configuration pushed down from FortiGate (1):
{ .annotate }

1. Below log is a result of FortiClient calling {==/remote/fortisslvpn_xml==} API endpoint on FortiGate. This can also been seen in **`diagnose application sslvpn -1`** output on FortiGate CLI.

``` xml
[2024-10-29 10:41:56.1631116 UTC-07:00] [6716:7300] [sslvpnlib  1488    info] <?xml version='1.0' encoding='utf-8'?><sslvpn-tunnel ver='2' dtls='1' patch='1'><dtls-config ver='2' heartbeat-interval='3' heartbeat-fail-count='3' heartbeat-idle-timeout='3' client-hello-timeout='10' dtls-accept-check-time ='1'/><tunnel-method value='ppp' /><tunnel-method value='tun' /><tunnel-method value='websocket' /><auth-ses check-src-ip='1' tun-connect-without-reauth='0' tun-user-ses-timeout='30' /><client-config save-password='on' keep-alive='on' auto-connect='on' /><ipv4><split-dns domains='vpdocs.net' dnsserver1='10.10.10.103' dnsserver2='10.10.10.104' /><assigned-addr ipv4='10.212.134.200' /><split-tunnel-info><addr ip='10.10.10.0' mask='255.255.255.0' /><addr ip='10.10.11.0' mask='255.255.255.0' /></split-tunnel-info></ipv4><idle-timeout val='300' /><auth-timeout val='28800' /></sslvpn-tunnel>
```

[^1]: See full FortiClient VPN log reference in [FortiClient VPN](index.md) landing page.

## Step 2 - Understand the environment FortiClient is in.

Evaluate what is installed on workstation that can cause DNS malfunction. For instance, it’s quite common to have 3d party DNS software present on workstation (i.e. Cisco Umbrella) which may affect DNS resolution when ON and OFF VPN.

## Step 3 – Use tools to analyze the problem.

Common tools that an engineering may find helpful include:

- Command line tools: (Windows) ipconfig, ping, nslookup; (macOS) scutil –dns, dig, nslookup
- Third-party tools: Wireshark

## Extras

Here's a helpful resource explaining [how DNS works on Windows](https://serverfault.com/questions/84291/how-does-windows-decide-which-dns-server-to-use-when-resolving-names).

## Demo

A demo going over DNS issues on macOS.
