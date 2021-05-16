#Config ProtonVPN for Mikrotik
```shell
/ip ipsec mode-config
add connection-mark=ProtonVPN name=ProtonVPN responder=no
```

```shell
/ip ipsec policy group
add name=ProtonVPN
```

```shell
/ip ipsec profile
add dh-group=modp4096,modp2048 enc-algorithm=aes-256 hash-algorithm=sha256 name=ProtonVPN
```

```shell
/ip ipsec peer
add address=nl-free-04.protonvpn.com disabled=yes exchange-mode=ike2 name=ProtonVPN profile=ProtonVPN send-initial-contact=no
```


```shell
/ip ipsec proposal
set [ find default=yes ] auth-algorithms=sha256 enc-algorithms=aes-256-cbc pfs-group=modp2048
```


```shell
/ip ipsec identity
add auth-method=eap certificate=ProtonVPN_ike_root.der_0 eap-methods=eap-mschapv2 generate-policy=port-override mode-config=ProtonVPN peer=ProtonVPN policy-template-group=ProtonVPN username=USERNAME password=PASSWORD
```


```shell
/ip ipsec policy
add dst-address=0.0.0.0/0 group=ProtonVPN src-address=0.0.0.0/0 template=yes
set [ find name=ProtonVPN ] connection-mark=ProtonVPN
```

```shell
/ip firewall mangle
add action=mark-connection chain=prerouting dst-address-list=VPN new-connection-mark=ProtonVPN passthrough=yes
```