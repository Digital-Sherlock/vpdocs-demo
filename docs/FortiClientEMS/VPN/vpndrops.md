# VPN Disconnects

VPN disconnects are often random and unpredictable which makes it hard to troubleshoot at times unless a systematic approach is followed. 

## Step 1 – Start with logs.

Client side logs

**Windows**: *C:\Program Files\Fortinet\FortiClient\logs\trace\sslvpndaemon_* locally on workstation or general\logs\trace\sslvpndaemon_ in Diagnostic_Result.zip

**macOS**: */Users/username/Library/Application Support/Fortinet/FortiClient/Logs\vpn-provider.log* or within the log export (FortiClient > Settings > Export Logs)

**FortiGate** side logs - search for VPN disconnection logs under VPN Events.

## Step 2 – Identify disconnect event.

Identify disconnect event by searching for keywords. Common ones are ***error***, ***disconnect***, ***terminated***, etc. In case of DTLS, also check for timeout errors - ***error: DTLS polling recv timeout***. When using IPsec with DPD, look for dead peer logs. It’s always a good idea to check with a customer regarding time of when the issue occurred.

See examples below:

**(FortiClient) IPSec DPD logs**

![FortiClient DPD dead peer](assets/dpd%20dead%20fct.png)

**(FortiGate) IPSec DPD logs**

![FortiGate DPD dead peer](assets/fos%20cli%20dpd.png)

FortiGate CLI commands providing output like the one above:

```
diag debug reset
diag debug console timestamp enable
diag vpn ike filter src-addr4 X.X.X.X. (public address of the endpoint)
diag debug app ike -1
diag debug enable
```

## Step 3 - Apply acquired knowledge towards problem resolution.

Often VPN disconnects are caused by network issues either within computer itself (driver or NIC issues, application errors, security program blocks, etc.) or outside - ISP, firewall issues.

It’s quite common to see Windows socket errors causing VPN drop so getting familiar with [Windows socket errors](https://learn.microsoft.com/en-us/windows/win32/winsock/windows-sockets-error-codes-2) will be helpful. Note, sometimes they might be represented in hexadecimal form like ***WSAGetLastError():2745*** which should be converted to decimal and then looked up on the Microsoft website.

Make sure to carefully examine software installed on workstation. Specifically, check for 3d party security software (common offenders are EDR, AV, WF, proxy security programs).

- ***Allproducts.xml*** file part of every FortiClient's diagnostics contains a list of installed applications (Diagnostic_Result.zip\FCDiagData\install\Allproducts.xml)
- ***SystemInfo.txt*** included in every diagnostics file shows system information including applications, crashes, drivers, OS details, etc.

Keep an eye on [broken pipes](../init_conn/#broken-pipes) as they can also show up in the logs and cause VPN to disconnect.

## Tools

In cases when disconnects are predictable (i.e. happens within a minute after VPN connection), use sniffers on both endpoint and firewall simultaneously to identify any packet loss.

Wireshark is a great choice for running a sniffer on endpoint. FortiGate has GUI packet capture tool as well as diagnose sniffer packet CLI utility. Sample command: ```diagnose sniffer packet any “host 173.222.34.12 and 10443” 6 0 l``` – combination of **6 0 l** switches will allow to convert the sniffer CLI output into .pcap format for use in Wireshark.

## Extras

Use sent/recvd counters in FortiClient sslvpndaemon to identify network timeouts.

For example in case of DTLS, it gets timed out after around 10 seconds if not receiving any traffic on FortiClient’s vNIC adapter. The 10-second of lack of any data on SSLVPN interface can be tracked within the same sslvpndaemon_x.log log by looking at recvd counters:

**bytes sent:29025412 bytes recvd:227333471 [handle_ssl_sock_recv_event]**

Find an entry similar to the one above and go up the log checking whether the recvd counter remains the same for 10 seconds – indication of no data received on the virtual interface causing disconnect.

## Demo

This demo walks through a VPN disconnect scenario suggesting a troubleshooting approach that one can follow to address it.

<div style="max-width: 640px"><div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;"><iframe src="https://fortinet-my.sharepoint.com/personal/vpolovnikov_fortinet-us_com/_layouts/15/embed.aspx?UniqueId=3d412b8b-1af2-468d-8a93-14f953f5b0bf&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Demos-20241021_102111-Meeting Recording.mp4" style="border:none; position: absolute; top: 0; left: 0; right: 0; bottom: 0; height: 100%; max-width: 100%;"></iframe></div></div>