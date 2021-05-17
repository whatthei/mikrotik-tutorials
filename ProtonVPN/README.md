# 🛡️ Config ProtonVPN for Mikrotik
> RouterOS version - **6.48.2**
1. Open the terminal in your RouterOS settings.
2. Install the ProtonVPN root CA certificate by running the following commands:
```shell
/tool fetch url="https://protonvpn.com/download/ProtonVPN_ike_root.der"
/certificate import file-name=ProtonVPN_ike_root.der
```
3. Now you have to set up the IPsec tunnel. It is advised to create a separate Phase 1 profile and Phase 2 proposal configurations to avoid interfering with any existing or future IPsec configuration:
```shell
/ip ipsec profile
add dh-group=modp4096,modp2048 enc-algorithm=aes-256 hash-algorithm=sha256 name=ProtonVPN
```
```shell
/ip ipsec proposal
set [ find default=yes ] auth-algorithms=sha256 enc-algorithms=aes-256-cbc pfs-group=modp2048
```
While it is possible to use the default policy template for policy generation, it is better to create a new policy group and template to separate this configuration from any other IPsec configuration.
```shell
/ip ipsec policy group
add name=ProtonVPN
```
```shell
/ip ipsec policy
add dst-address=0.0.0.0/0 group=ProtonVPN src-address=0.0.0.0/0 template=yes
set [ find name=ProtonVPN ] connection-mark=ProtonVPN
```

4. Create a new mode config entry with responder=no that will request configuration parameters from the server:
```shell
/ip ipsec mode-config
add connection-mark=ProtonVPN name=ProtonVPN responder=no
```
5. Create peer and identity configurations. Specify your ProtonVPN **server** and credentials in the **username** and **password** parameters:
```shell
/ip ipsec peer
add address=nl-free-04.protonvpn.com disabled=yes exchange-mode=ike2 name=ProtonVPN profile=ProtonVPN send-initial-contact=no
```
```shell
/ip ipsec identity
add auth-method=eap certificate=ProtonVPN_ike_root.der_0 eap-methods=eap-mschapv2 generate-policy=port-override mode-config=ProtonVPN peer=ProtonVPN policy-template-group=ProtonVPN username=USERNAME password=PASSWORD
```
You can find your ProtonVPN service credentials in the ProtonVPN Account dashboard. Copy the credentials using “Copy” the buttons on the right.

6. Now choose what to send over the VPN tunnel. 
   
It is possible to send only specific traffic over the tunnel by using the connection-mark parameter in Mangle firewall.

- Create a new address list:
```shell
/ip firewall address-list
add address=mikrotik.com list=VPN
add address=protonvpn.com list=VPN
```
- Apply connection-mark to traffic matching the created address list:
```shell
/ip firewall mangle
add action=mark-connection chain=prerouting dst-address-list=VPN new-connection-mark=ProtonVPN passthrough=yes
```
## REFERENCES
- [Linux IKEv2 ProtonVPN tutorial](https://protonvpn.com/support/linux-ikev2-protonvpn/)
- [IKEv2 EAP between VPN and RouterOS](https://wiki.mikrotik.com/wiki/IKEv2_EAP_between_NordVPN_and_RouterOS)