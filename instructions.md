% Project Nephology: Wireguard

# Why

## Speed
Wireguard is a lot faster, almost twice as fast in my testing on a Raspberry Pi 4

## Modern Crypto
Includes only modern crypto suites, no legacy, and uses higher performance encryption

## UDP
Only uses UDP for transport (boo, kinda)

## Simple
Includes very few features, complex behavior can be built on top of it though

# Terms

## Peer
Any remote connection from one system to another.  In a standard client-server type setup, the client would call the server a peer, and vice versa.

## Pre-Shared-Key
A pre-shared symmetric encryption key which provides additional security, and should be resistent to quantum computing attacks.

# Installation

Since Wireguard is built as a kernel module, for newer kernels, you won't need to install anything.  You will still have to install the command line tools though.

 - Install command line tools `wireguard-tools` in Arch
 - Install kernel module (not needed for kernel >= 5.6)

# Generate Keys

You weill need to generate a public private keypair for each peer.  So for a simple client-server setup, you'll need two keypairs.  You can also optionally generate a pre-shared-key for post-quantum crypto resistence.

## Server Key (on server)

- `wg genkey > /etc/wireguard/server.key`
- `chmod 600 /etc/wireguard/server.key` 
- `wg pubkey < /etc/wireguard/server.key > /etc/wireguard/server.pub`

## Laptop key (on laptop)

- `wg genkey > /etc/wireguard/laptop.key`
- `chmod 600 /etc/wireguard/laptop.key` 
- `wg pubkey < /etc/wireguard/laptop.key > /etc/wireguard/laptop.pub`

## (Optional) Generate pre-shared keys, copied to both server and laptop

- `wg genpsk > /etc/wireguard/server-laptop.psk`
- `chmod 600 /etc/wireguard/server-laptop.psk`

# Setup host 

You an control wireguard via native wg-tools commands, systemd-networkd, netctl, or NetworkManager.  Netctl or systemd-network should be used for server / headless installations, but NetworkManager is the easiest for user machines.

## Setup the server - wireguard config

`/etc/wireguard/wg0.conf`
```
[Interface]
ListenPort = 51871
PrivateKey = SERVER_PRIVATE_KEY

[Peer]
PublicKey = LAPTOP_PUBLIC_KEY
PresharedKey = SERVER_LAPTOP_PRE_SHARED_KEY
AllowedIPs = 192.168.47.0/24, 192.168.1.0/24
```
    
## Setup the server - netctl config

`/etc/netctl/wg0`
```
Description="WireGuard VPN Server"
Interface=wg0
Connection=wireguard
WGConfigFile=/etc/wireguard/wg0.conf

IP=static
Address=('192.168.47.1/24')
```

## Setup the server - enable connection

`netctl start wg0`
    
# Setup client
In a client mode, you have a similar config, but you specify a peer endpoint that wireguard will connect to, and optionally a list of routes that should be sent through the remote peer.
 
## Setup client - wireguard config

`/etc/wireguard/wg0.conf`
```
[Interface]
PrivateKey = LAPTOP_PRIVATE_KEY

[Peer]
PublicKey = SERVER_PUBLIC_KEY
PresharedKey = SERVER_LAPTOP_PRE_SHARED_KEY
AllowedIPs = 192.168.47.1/32
Endpoint = SERVER_ADDRESS_AND_PORT
```
 
## Setup client - netctl config

`/etc/netctl/wg0`
```
Description="WireGuard VPN Server"
Interface=wg0
Connection=wireguard
WGConfigFile=/etc/wireguard/wg0.conf

IP=static
Address=('192.168.47.10/24')
Routes=('192.168.1.0/24')
Routes=('0.0.0.0/0')
```
        
## Setup client - Enable the connection
`netctl start wg0`

# Automated GUIs

A manual config would probably be my preference for joining two networks, or setting up a static remote connection, but if you have multiple devices you want to connect, one of the automated guis is the way to go.  A few different ones I've tried.
  
## wireguard-access-server

This is the one I ended up going with, supports several different auth modes, and makes it dead simple to add clients.  Also includes a handy DNS proxy that makes some DNS setups easier.  This is the main reason I ended up going with it.

## wg-ui

I had trouble with DNS on this one, and it was kinda flaky and hard to debug

## subspace
Didn't try this one, seems to be abandoned, no commits in over a year, does look the most polished though


## wg-gen-web
More complicated than the others, has more config options though
