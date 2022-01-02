# ⛰️ Config NordVPN for Mikrotik
> Сhecked RouterOS version - **7.1.1**
1. Open the terminal in your RouterOS settings.

2. Install the NordVPN root CA certificate by running the following commands:
```shell
/tool fetch url="https://downloads.nordcdn.com/certificates/root.der"
/certificate import file-name=root.der name="NordVPN" passphrase=""
```

3. Create a new mode config entry with responder=no that will request configuration parameters from the server:
```shell
/ip ipsec mode-config
add connection-mark="NordVPN" name="NordVPN" responder=no
```

4. Now you have to set up the IPsec tunnel. It is advised to create a separate Phase 1 profile and Phase 2 proposal configurations to avoid interfering with any existing or future IPsec configuration:
```shell
/ip ipsec profile
add dh-group=modp2048 enc-algorithm=aes-256 hash-algorithm=sha512 name="NordVPN"
```
```shell
/ip ipsec proposal
add auth-algorithms=sha256 enc-algorithms=aes-256-cbc lifetime=0s name="NordVPN" pfs-group=none
```
While it is possible to use the default policy template for policy generation, it is better to create a new policy group and template to separate this configuration from any other IPsec configuration.
```shell
/ip ipsec policy group
add name="NordVPN"
```
```shell
/ip ipsec policy
add dst-address=0.0.0.0/0 group="NordVPN" proposal="NordVPN" src-address=0.0.0.0/0 template=yes
```

5. Create peer and identity configurations. Specify your NordVPN **server** and credentials in the **username** and **password** parameters:

Go to https://nordvpn.com/servers/tools/ to find out the hostname of the server recommended for you. 
```shell
/ip ipsec peer
add address=XXXXXXXXXX exchange-mode=ike2 name="NordVPN" profile="NordVPN"
```
You can find your NordVPN service credentials in the Nord Account https://my.nordaccount.com/dashboard/nordvpn/. Copy the credentials using “Copy” the buttons on the right.
```shell
/ip ipsec identity
add auth-method=eap certificate="NordVPN" eap-methods=eap-mschapv2 generate-policy=port-strict mode-config="NordVPN" peer="NordVPN" policy-template-group="NordVPN" username=XXXXXXXXXX password=XXXXXXXXXX
```

6. Now choose what to send over the VPN tunnel. 

    - All traffic or specific traffic (by source)
    
    Create a new address list:

    fro all traffic:
    ```shell
    /ip firewall address-list
    add address=192.168.88.0/24 list="VPN"
    ```
    or specific:
    ```shell
    /ip firewall address-list
    add address=192.168.88.252 list="VPN"
    add address=192.168.88.248 list="VPN"
    ```
    Apply connection-mark to traffic matching the created address list:
    ```shell
    /ip firewall mangle
    add action=mark-connection chain=prerouting src-address-list="VPN" new-connection-mark="NordVPN" passthrough=yes
    ```
    
    - Specific traffic (by destination)

    Create a new address list:
    ```shell
    /ip firewall address-list
    add address=mikrotik.com list="VPN"
    add address=nordvpn.com list="VPN"
    ```
    Apply connection-mark to traffic matching the created address list:
    ```shell
    /ip firewall mangle
    action=mark-connection chain=prerouting dst-address-list="VPN" new-connection-mark="NordVPN" passthrough=yes
    ```

7. Exclude such VPN traffic from fasttrack
```shell
/ip firewall filter
add action=accept chain=forward connection-mark="NordVPN" place-before=[find where action=fasttrack-connection]
```

8. # Reduce MSS
```shell
/ip firewall mangle
add action=change-mss chain=forward new-mss=1360 passthrough=yes protocol=tcp connection-mark="NordVPN" tcp-flags=syn tcp-mss=!0-1360
```

## REFERENCES
- 🔥 [NordVPN (IPSEC/IKEv2) + killswitch (For ROS6)](https://forum.mikrotik.com/viewtopic.php?f=23&t=169273) from erkexzcx
- [IKEv2 EAP between VPN and RouterOS](https://wiki.mikrotik.com/wiki/IKEv2_EAP_between_NordVPN_and_RouterOS) from Mikrotik wiki
- [MikroTik IKEv2 setup with NordVPN](https://support.nordvpn.com/Connectivity/Router/1360295132/MikroTik-IKEv2-setup-with-NordVPN.htm) from NordVPN support