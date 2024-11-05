Valid ZTNA certificate ensures FortiClient making connection to ZTNA destination is legitimate and managed by trusted EMS.

## Step 1 – Identify ZTNA certificate issue.
Mainly, there are two common issues with ZTNA certificate: it can be missing or incorrect.

These issues are quite easy to spot since both FortiClient and FortiGate provide clear logging. Some of the logs are more visual and should be checked first -  FortiClient browser error, FortiGate ZTNA event error.

![ZTNA certificate error](assets/ZTNA%20certificate%20missing.png)

Others can be discovered while undertaking more in-depth analysis - FortiClient fortitcs log, FortiGate wad diagnostics.

Example of FortiGate wad diagnostics:

``` bash
diagnose wad filter clear
diagnose wad filter src X.X.X.X # (1)
diagnose wad debug enable category all
diagnose wad debug enable level verbose
diagnose debug enable
```

1. X.X.X.X is a public address of an endpoint

## Step 2 – Verify certificate presence and/or validity.

On Windows, the ZTNA certificate is stored in the personal user certificate store.

<img src="../assets/ZTNA Windows cert.jpg" alt="ZTNA Windows cert" width="400" height="400">

On MacOS, the certificate is located under the {==Keychain Access > login > Certificates==}.

<img src="../assets/ZTNA macOS cert.png" alt="ZTNA macOS cert" width="700" height="400">

On EMS, you can verify the certificate by checking endpoint information from the endpoints list.

<img src="../assets/ZTNA_cert_EMS.png" alt="ZTNA cert on EMS" width="700" height="400">

## Step 3 – Check ZTNA certificate installation details.

The log that describes ZTNA certificate installation is FortiESNAC on Windows and epctrl.log on macOS. See Appendix for a complete ZTNA log reference.

A good place to start with is locating FortiClient-EMS keep-alive (KA) exchange that follows synchronization attempt with EMS:

**`[FortiESNAC  948    info] Attempting to sync with EMS`**

During each KA communication, FortiClient checks for ZTNA certificate presence. Example below:

**`[FortiESNAC  264   debug] check ztna cert exist in store 54B10426C8884D5CAAC27DD988DBF0AB ret=1`**

Alternatively, one can directly filter for FortiClient UID or keywords like “ZTNA” or “cert” in the FortiESNAC or epctrl log to quickly locate ZTNA certificate related log entries.

## Step 4 – Explore solutions.

Sometimes, all it takes to resolve FortiClient ZTNA certificate issues is re-connecting FortiClient to EMS which triggers a new CSR request sent by the latter and signed by the former. This exchange also can be tracked via FortiESNAC or epctrl logs depending on OS.

Other solutions may require in-depth analysis of FortiClient – EMS Telemetry communication which involves earlier mentioned endpoint logs as well as EMS’ such as kaworker, regworker, and others.

Since FortiGate plays a role in certificate verification, an engineer should be comfortable with navigating the command line and specifically ZTNA suite of CLI tools. Note, these commands should help in understanding what component – FortiClient, EMS, or FortiGate -  is a point of failure. If FortiGate is confirmed to be causing an issue, it has to be communicated clearly to a customer and a qualified FortiGate specialist has to be involved for further troubleshooting.

Here’s few CLI utilities that will help verify ZTNA certificate issues:

```
diagnose endpoint record list
diagnose wad dev query-by uid UID EMS-SN TENANT-ID
diagnose test application fcnacd 7
```
!!! note "Note"
    `diagnose endpoint record list` was replaced by `diagnose endpoint ep-shm list` in newer FortiOS versions.

Upon initial ZTNA connection, FortiGate queries EMS using the details it retrieves from FortiClient’s ZTNA certificate. Specifically certificate’s common name which EMS sends a serial number for back to the FortiGate so it can verify whether a connecting FortiClient’s ZTNA certificate is valid.

To track this communication use the following commands as a template:
``` bash
diag debug reset
diag debug console timestamp enable
diag debug app fcnacd -1
diag endpoint filter show-large-data yes
diag wad filter clear
diag wad filter src X.X.X.X # (1)
diag wad debug enable category all
diag wad debug enable level verbose
diag debug enable
```

1. X.X.X.X is a public address of an endpoint

Once debugging starts, attempt the ZTNA connection from FortiClient.