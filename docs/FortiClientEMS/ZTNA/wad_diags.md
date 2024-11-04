Even though FortiClient\EMS support team aren’t required to troubleshoot FortiGate, navigating around one and knowing how it contributes to Zero Trust architecture is critical.

Since the majority of information one would need has been already covered in the previous sections, we’ll consider a ZTNA connection case study - endpoint performs RDP connection over ZTNA – and follow FortiGate’s WAD daemon logs. This should provide an intuition of how FortiGate processes ZTNA connections and an understanding in reading ZTNA logs.

## Case Study

1. **Endpoint performes RDP connection** to 172.16.1.10:3389. This connection is received from client on external IP and external port of ZTNA Server VIP.

    ![wad 1](assets/wad%201.png)

2. **FortiGate requests ZTNA certificate,** which will be verified by EMS ZTNA Root CA of available EMS Connectors.

    ![wad 2](assets/wad%202.png)

3. **FortiClient provides ZTNA certificate**. If it is not cached on FortiGate, its serial number is shown, and then verified by EMS ZTNA Root CA.

    ![wad 3](assets/wad%203.png)

4. If **it is in cache**, action taken is from cached result.

    ![wad 4](assets/wad%204.png)

5. **TLS handshake completes** with FortiGate and ticket is offered to client.

    ![wad 5](assets/wad%205.png)

6. **FortiClient makes ZTNA app request** after successful ZTNA certificate verification.

    ![wad 7](assets/wad%207.png)

7. **Request payload is decapsulated** and then following checks are done in order:
    - API Gateway (ZTNA Gateway address)
    - Virtual Host and then Pattern
    - Real Server
    - FQDN (DNS query) and IP-address
    - Route lookup to check for destination reachability (if not, likely a “504 Gateway Timeout” is presented)
    - Policy match starts and source address and interface are checked

    ![wad 8](assets/wad%208.png)

8. **Policy Matching** continues after source address and interface are checked.
    - Device check (ZTNA Certificate S/Nu and ZTNA Tags)
    - Authentication

    ![wad 9](assets/wad%209.png)

9. **Authentication** process starts.

    ![wad 10](assets/wad%2010.png)

10. Authentication completes and **FortiGate connects to Real Server**.

    ![wad 11](assets/wad%2011.png)

11. **Data flow starts passing** between FortiClient and server protected via ZTNA Access Proxy.

    ![wad 12](assets/wad%2012.png)

## Notes

The output above is retreived by running the following commands:

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

