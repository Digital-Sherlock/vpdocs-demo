# VPN Disconnects

VPN disconnects are often random and unpredictable which makes it hard to troubleshoot at times unless a systematic approach is followed. 

## Step 1 – Start with logs.

Client side logs

**Windows**: *C:\Program Files\Fortinet\FortiClient\logs\trace\sslvpndaemon_* locally on workstation or general\logs\trace\sslvpndaemon_ in Diagnostic_Result.zip

**macOS**: */Users/username/Library/Application Support/Fortinet/FortiClient/Logs\vpn-provider.log* or within the log export (FortiClient > Settings > Export Logs)

**FortiGate** side logs - search for VPN disconnection logs under VPN Events.

## Step 2 – Identify disconnect event.

Identify disconnect event by searching for keywords. Common ones are error, disconnect, terminated, etc. In case of DTLS, also check for timeout errors - error: DTLS polling recv timeout. When using IPsec with DPD, look for dead peer logs. It’s always a good idea to check with a customer regarding time of when the issue occurred.
