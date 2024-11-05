Zero Trust tags are critical constituent of Zero Trust architecture used for posture check not only on FortiGate but on FortiClient as well (pre-VPN compliance verification[^1]). Hence, effectively navigating FortiClient’s ZTNA elements is paramount for troubleshooting ZTNA failures.

[^1]: **pre-VPN compliance verification** is referred to FortiClient allowing or blocking VPN connection based on applied Posture tags. For more information see [official documentation](https://docs.fortinet.com/document/forticlient/7.4.0/ems-administration-guide/701440/configuring-a-profile-to-allow-or-block-endpoint-from-vpn-tunnel-connection-based-on-the-applied-security-posture-tag)

A common problem we’ll explore here is Zero Trust tags assignment which can affect ZTNA connections, IP/MAC Access Control as well as pre-VPN compliance check.

## Step 1 – Identify current ZT tags assignment state.

Is FortiClient missing one, few, or all tags?

There are multiple ways to confirm what tags are assigned to endpoit.

- On FortiClient, under user's avatar.

<img src="../assets/FC tags.png" alt="FC tags" width="500" height="500">

- On EMS, under endpoint's summary tab.

<img src="../assets/EMS endpoint info.png" alt="EMS endpoint info" width="500" height="500">

- On EMS, under tag monitor.

<img src="../assets/Tag monitor.png" alt="Tag monitor">

## Step 2 – Verify tag assigning rules constituting the tag(s).

Before executing log analysis, make sure to check what are the rules constituting the missing tag(s).

EMS sends tagging rules - security posture, outbreak, classification, fabric tagging rules - to FortiClient that stores them locally and evaluates on each sync with EMS. On Windows, these rules can be checked in {==host_verification.plain==} file under **`C:\Program Files\Fortinet\FortiClient\logs\ec`**.

Here's a snippet of *EMS Management* rule from host_verification.plain file:

``` xml
<rule_flag>24</rule_flag>
<name><![CDATA[Endpoint Management]]></name>
<os>windows</os>
<criteria>
<feature>ems_management</feature>
<criterion>
<content>
<![CDATA[FortiClient installed and Telemetry connected to EMS]]>
</content>
```

This is how it looks on EMS.

![EMS tagging rule](../assets/ems%20tagging%20rules.png)

## Step 3 – Examine the logs.

Depending on the tagging rule, either FortiClient’s or EMS’ logs have to be examined.

- FortiClient: (Windows) FortiESNAC.log, (macOS) epctrl.log
- EMS: tagworker, kaworker, fcmdaemon

!!! warning "Logs"
    Log files' names are subject to change and may differ depending on EMS version.

Here's an example of FortiESNAC.log's snippet:

![FortiESNAC log](assets/fortiesnac%20tag%20evaluation.png)

Note, the rule number highlighted in the log (24) which corresponds to `<rule_flag>24</rule_flag>` in the host_verification.plain file (see above).

!!! tip "Tag evaluation"
    Some tags are evaluated on EMS and some on FortiClient. When a tag is calculated on EMS, you'll find `checkrule unsupported` in the FortiESNAC log.

## Step 4 – Determine a plan of action.

FortiClient and EMS endpoint control logs combined should suffice in determining a plan of action for addressing the issue. That plan will heavily depend on the tagging rule type and whether it's evaluated on FortiClient or EMS (1).
{ .annotate}

1. Common example is ***User in AD Group*** rule which can be configured to be evaluted either on FortiClient or EMS. Consult with official documentation for different [rule types](https://docs.fortinet.com/document/forticlient/7.4.0/ems-administration-guide/342488/security-posture-tagging-rule-types).