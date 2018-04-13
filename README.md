# edgerouter-vpn-failover
Ubiquiti EdgeRouter configuration file and script(s) to perform auto-failover and fallback when using a VPN. This was implemented on an EdgeRouter-6P but it should work on any EdgeRouter.

## Why, when there's a wizard?

The Wizard doesn't work when you introduce a VPN into the equation. The key things here seem to be tweaks in the `*.ovpn` file as well as the script. I am not claiming that I have written some magic configuration from scratch, but this did take me several hours to get working as desired so I thought I would share. Hope it is helpful.

## Assumptions

This assumes the following:
* eth0 is your primary ISP and serves you DHCP
* eth1 is your secondary (fail-over only) ISP and serves you DHCP
* You want to fail-over when necessary but fall back as soon as your primary is back up

## Notes for UBNT Firmware v1.10.1

Per UBNT engineers, there is a bug in v1.10.1 that should be resolved like this:

`curl -O https://dl.ubnt.com/firmwares/edgemax/afomins/fix-lb-regression-in-1-10-1/ubnt-add-connected.pl && chmod +x ubnt-add-connected.pl && sudo cp ubnt-add-connected.pl /usr/sbin/`

I'm not sure if this is important or not and haven't examined it. It came from a UBNT engineer on a UBNT forum. FWIW.

## How to Use

```
$ scp config/scripts/lb-transition 192.168.1.1:lb-transition
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
ping 5
ping-restart 15
script-security 2
```

## End State Behavior

Assuming eth0 is your first ISP and eth1 is your second, your first ISP is designated as your primary with your second as failover. If eth0 is yanked out or otherwise goes dead, traffic switches over to the second ISP and the VPN is re-established through that link. When the first ISP is again "up", things will very quickly dynamically change back to using that link. This fallback to primary as soon as possible behavior is very important if your secondary ISP is something like a pricey LTE/4G uplink!

## Credits

UBNT support forums and reddit were very helpful in getting this to work
