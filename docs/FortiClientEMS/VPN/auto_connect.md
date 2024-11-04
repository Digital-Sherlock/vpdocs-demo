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