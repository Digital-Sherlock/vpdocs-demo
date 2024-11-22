# Auto-Connect and Always-up Issues

Troubleshooting auto-connect and always-up features comes down to:

1. Knowing what log files to analyze
2. "following" the log

## Step 1 – Understand the setup.

Request customer’s FortiClient configuration to understand how their auto-connect and always-up is configured. For instance, is it a regular auto-connect or auto-connect only when off-fabric? Or maybe customer uses **`tunnel-connect-without-reauth`** (1) feature on FortiGate. This knowledge will guide further troubleshooting.
{ .annotate }

1. Community article about this feature - [Configuring SSL-VPN to allow tunnel reconnection without requiring reauthentication](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Configuring-SSL-VPN-to-allow-tunnel-reconnection/ta-p/220498)

## Step 2 – Identify the log.

The first logs to check when dealing with auto-connect and always-up features are {==FortiVPN==} and {==FortiTray==}. Start with FortiVPN when dealing with auto-connect and FortiTray when having issues with always-up.

## Step 3 - Look for keywords and follow the log.

Both of the logs used for troubleshooting here may get overwhelming due to amount of entries. So here’s few hints that can help in narrowing down the search area.

- Ask customer for a timestamp of auto-connect failure

- Pay attention to {==when==} the FortiVPN / FortiTray processes load which indicates workstation login usually followed by auto-connect (see below as an example)

```
[2023-02-17 14:03:32.0914862] [5676:7340] [fortitray 1216] (Wed Aug 31 12:42:36 2022, 3643178s) 2023-02-17 14:03:32.083 type: 0x10f size: 512 memory: 0x0000000000000000
[2023-02-17 14:03:32.0915313] [5676:7340] [fortitray 171 debug] CFortiTrayApp::InitInstance 596
[2023-02-17 14:03:32.0915362] [5676:7340] [fortitray 171 debug] FortiTray.exe Version:7.0.7.345
```

- Search for keywords (autoconnect, always up, error, fail, etc.). If auto-connect is used only when off-fabric, search for **epc**, **endpointcontrol** keywords in addition to the earlier mentioned ones. See example below:

```
[fortitray 1239 debug] WaitScheduleEventThread epc_state=4
[fortitray 174 debug] CFortiTrayDlg::CheckAutoConnectTunnel called with bFromEC = 1
[fortitray 45 debug] epc::IEndpointControl::queryState {"status":2,"status_msg":"ready","onnet":false}
```

Once the area for analysis is located follow the log and search for further clues.

For FortiVPN, {==“In State: ”==} is a great keyword. For example, to {==“In State: UserLogin”==} or {==“In State: StartConnection”==}.

![In state](assets/in%20state.png)

## Demo

Below demo demonstrates an approach of investigating auto-connect issue by examining FortiClient {==FortiVPN==} log. It also provides general guidance on tackling auto-connect cases.

Part 1.

<div style="max-width: 640px"><div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;"><iframe src="https://fortinet-my.sharepoint.com/personal/vpolovnikov_fortinet-us_com/_layouts/15/embed.aspx?UniqueId=d1ede85f-bf86-40fb-b277-990edbdab2d0&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Demos-20241104_151047-Meeting Recording.mp4" style="border:none; position: absolute; top: 0; left: 0; right: 0; bottom: 0; height: 100%; max-width: 100%;"></iframe></div></div>

Part 2.

<div style="max-width: 640px"><div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;"><iframe src="https://fortinet-my.sharepoint.com/personal/vpolovnikov_fortinet-us_com/_layouts/15/embed.aspx?UniqueId=539af19a-78ea-4be7-981b-49743b963430&embed=%7B%22ust%22%3Atrue%2C%22hv%22%3A%22CopyEmbedCode%22%7D&referrer=StreamWebApp&referrerScenario=EmbedDialog.Create" width="640" height="360" frameborder="0" scrolling="no" allowfullscreen title="Demos-20241104_151800-Meeting Recording.mp4" style="border:none; position: absolute; top: 0; left: 0; right: 0; bottom: 0; height: 100%; max-width: 100%;"></iframe></div></div>