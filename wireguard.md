# Wireguard VPN

This Documentation shows the installation process on Debian 9 Stretch GNU/Linux.

## Installation

On both the Server and the Client, run the following commands:

~~~bash
echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable.list
printf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' > /etc/apt/preferences.d/limit-unstable
apt update
apt install wireguard # start the installation
~~~

## Key Generation

You need to generate Keys on both Machines:

~~~bash
cd /etc/wireguard
unmask 0777
wg genkey | tee privatekey | wg pubkey > publickey # generates a new keypair
~~~

You can show the generated key by typing:

~~~bash
cd /etc/wireguard
cat privatekey # shows the private key
cat publickey # shows the public key
~~~

## Configuration

To set up the Server Configuration, create a file called wg0.conf and insert the following content:

~~~
[Interface]
Address = 192.168.2.1/32
PrivateKey = # insert servers private key
ListenPort = 51820

[Peer]
PublicKey = # insert clients public key
AllowedIPs = 192.168.2.2/32
~~~

Then enable IP Forwarding inside /etc/sysctl.conf

You also need to set up an iptables rule at boot time (use the rc.local script or a systemd service):

> iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

Make sure you are using the correct Interface Name (you may need to replace eth0).

Furthermore, enable Port Forwarding on your Router to make your Server reachable from the Internet.

On the Client System the config file (/etc/wireguard/wg0.conf) looks as follows:

~~~
[Interface]
Address = 192.168.2.2/32
PrivateKey = # insert clients private key
ListenPort = 5555

[Peer]
PublicKey = # insert servers public key
Endpoint = machine:51820 # replace 'machine' with hostname of the server
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
~~~

## Usage

To start the VPN Tunnel on the Server and the Client, run

~~~bash
wg-quick up wg0 # enables the vpn connection
~~~

on both machines.

To shut down the tunnel, use:

~~~bash
wg-quick down wg0 # disables the vpn connection
~~~

You then should be good to go.