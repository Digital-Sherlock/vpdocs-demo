Issues with Fabric Connector can vary and be categorized as following:

- Connector failure (can’t be established or goes up and down randomly)
- Tag/endpoint information exchange issues (FortiGate missing tags)

Let’s break them down and examine the strategies of addressing these issues.

## Connector Failure

![Fabric Connector failure image](assets/fabric%20connector%20failure.png)

Similar techniques can be employed for addressing connector failures or instabilities. The difference will be the timing due to sporadic nature of connector stability issues.

### Step 1 – Examine the error code.

Start with analyzing the error code presented by FortiGate’s EMS Fabric Connector. Even though most of the time connector issues will require diagnosing off of CLI, sometimes the error shown in the GUI can spill out some hints.

### Step 2 – Equip with FOS CLI tools.

A good place to start is running Fabric Connector verification commands and examining the output. Here’s few commands you want to have under your belt:

``` bash
diagnose endpoint fctems test-connectivity <EMS ID>
execute fctems verify <EMS ID> 
diagnose test application fcnacd 2
```

![fcnacd 1](assets/fcnacd%201.png)

The above are connection verification commands that don’t produce continuous output.

Next are the commands that show back-and-forth communication between FortiGate and EMS.

``` bash
diagnose debug application fcnacd -1
diagnose debug console timestamp enable
diagnose endpoint filter show-large-data yes
diagnose debug enable
```

The above commands return decoded data exchange between FortiGate and EMS. Most of the time, these CLI tools are sufficient for diagnosing the problem.

### Step 3 – Correlate with EMS logs.

Depending on EMS version (7.2 or 7.4) you may need to analyze the following logs:

- Python logs (api, debug and error) – FortiGate–EMS Fabric communication is carried over API calls and Python logs store this information
- fcmNotify (7.2) or notifyworker (7.4) – provide communication data like FortiGate registration attempts or periodic keep-alives (ping-pongs)

### Extras

Here’s some other commands that one may find useful.

**`diagnose endpoint fctems json gateway-mac-request`** - Outputs the JSON-formatted list of FortiGate’s interfaces (gateways) with IP- and MAC-addresses. This is the list FortiGate sends to EMS so the latter can identify which endpoints are directly connected to the firewall.

**`diagnose test application fcnacd 5`** - Makes FortiGate to execute API calls to EMS’ API endpoints on demand.

**`diagnose test application fcnacd 99`** - Restarts fcnacd daemon. Exercise caution when using this command. 

Remember to use FortiGate commands for verification purposes. If analysis points to FortiGate having issues, make sure to engage dedicated FortiGate support engineer and communicate the results of your findings.

---

!!! danger "Connector Stability Issies"
    Connector stability issues are harder to troubleshoot due to their sporadic nature. It may happen every few days, hours, or minutes, etc. Hence, one would either need to time them (in case there’s a pattern or failures happen every few minutes) or find a way to run diagnostics for a prolonged period of time. I’d recommend engaging FortiGate support for issues like this as they should have a knowledge of running CLI diagnostics for a prolonged period of time, collect crash dumps, etc.

## Tag Exchange Issues

Another common issue is missing tags or endpoint information.

More often than not, these issues arise due to misunderstanding of how the technology works.

!!! note 
    A common misconception is that having FortiClient connected to EMS and tagged is sufficient for the tags to appear on FortiGate ***which isn’t always correct.***

### Step 1 – Understand the setup and network topology.

<img src="../assets/ZTNA over VPN.png" alt="ZTNA topology" width="400" height="400">

*Are we talking about ZTNA or IP/MAC Access Control?*

*What is the topology - FortiClient connected via VPN to FortiGate, FortiClient is having FortiGate as default gateway, there’s L3 device in between FortiGate and FortiClient, etc.?*

*What are tagging rules a desired tag is comprised of?*

---

Having answers to questions like these will guide further troubleshooting efforts.

It goes without saying that an engineer must understand how different scenarios outlined above will affect tag sharing mechanics. 

### Step 2 – Collect logs.

Logs have to be collected from each component of ZT infrastructure – FortiClient, FortiGate, and EMS – though may vary depending on a given case [^1]. 

[^1]: See Logs section under ZTNA [landing page](index.md) for a complete list.

1. FortiClient
    - FortiClient diagnostics and logs (FortiESNAC or epctrl.log depending on OS) will tell whether machine has desired tag(s). This is the first place to start verifying tag mismatch issues.
2. EMS diagnostics
    - Since EMS can only have debug log enabled for 30 minutes, an engineer would have to align issue occurrence with EMS running in a debug mode.
3. FortiGate
    - Same goes with FortiGate – issue has to be reproduced while FortiGate CLI commands are executed.

!!! info "Note"
    Here’s few common cases when the issue is reproduceable or can be reproduced on demand:

    - FortiClient connects to FortiGate VPN server which doesn’t reflect client’s IP-address matching a tag

    - FortiClient located behind FortiGate doesn’t have its tags synced causing policy violation

    - FortiClient attempts to reach ZTNA destination but FortiGate denies the request due to no policy matched

    Cases like these are easier to capture the diagnostic data for as they can be replicated when required.

### Step 3 – Analyze the data.

**FortiClient** is first to start with since it’s a subject for ZTNA evaluation and the instance that initiates the connection. Hence, make sure FortiClient has the tags required and is properly configured.

**FortiGate** is where connection terminates. Check its logs and diagnostics and identify the cause for policy violation. FortiGate either has the information required for FortiClient’s posture evaluatuation or requests this information from EMS when the connection is made which makes EMS the next step in log analysis.

**EMS** sends tagging rules to FortiClient and shares endpoint information (including tags) to FortiGate. EMS "reacts" to FortiGate requests so make sure to track them in earlier mentioned EMS logs. 