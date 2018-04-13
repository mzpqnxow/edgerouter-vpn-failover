# edgerouter-vpn-failover
Ubiquiti EdgeRouter configuration file and script(s) to perform auto-failover and Fallback when using a VPN

## Notes for UBNT Firmware v1.10.1

Per UBNT engineers, there is a bug in v1.10.1 that should be resolved like this:

`curl -O https://dl.ubnt.com/firmwares/edgemax/afomins/fix-lb-regression-in-1-10-1/ubnt-add-connected.pl && chmod +x ubnt-add-connected.pl && sudo cp ubnt-add-connected.pl /usr/sbin/`

I'm not sure if this is important or not and haven't examined it. It came from a UBNT engineer on a UBNT forum. FWIW.

## How to Use

```$ scp config/scripts/lb-transition 192.168.1.1:lb-transition
$ ssh 192.168.1.1
$ sudo mkdir -p /config/scripts && sudo mv lb-transition /config/scripts && sudo chmod 755 /config/scripts/lb-transition
```

Now, go about your business integrating the `load-balance` settings into your device configuration via the CLI

## OpenVPN Settings

My OpenVPN settings look like this. There may be a problem if yours are very different, so I am including them here

```
client
dev-type tun
proto udp
remote my.vpn.com 443
resolv-retry infinite
nobind
persist-key
persist-tun
persist-remote-ip
ca /config/auth/vpn_ca.pem
tls-client
remote-cert-tls server
auth-user-pass /config/auth/vpn.creds
comp-lzo
reneg-sec 0
link-mtu 1570
cipher AES-256-CBC
auth SHA256
keysize 256
persist-key
persist-tun
verify-x509-name my.vpn.com name
verb 6 
log /tmp/eth0
ping 5
ping-restart 15
status /tmp/VPNSTATUS
script-security 2
```
