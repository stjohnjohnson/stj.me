---
layout: post
title: OpenVPN and DD-WRT
date: 2016-02-17
---

I'm not sure why but there is apparently *no* simple tutorial on how to get started with an OpenVPN server on DD-WRT.

When I set this up, I followed about 15 different guides and had to steal some configuration from a friend (who doesn't remember where he found it).

This shouldn't ever be this difficult, so I decided to write it all down for the next person to have an easier time.

<!--more-->

## Step 1: Install Tunnelblick

Okay, so the first thing we need is a OpenVPN client.  I chose [Tunnelblick][tunnelblick] for OSX because it's free, open source, and comes bundled with [easy-rsa][easy-rsa] (which will make our life REALLY easy).

Installing is simple, just install via [their website][tunnelblick-download] or via [Homebrew][homebrew]'s [Cask][cask].

```
$ brew cask install tunnelblick
```

## Step 2: Generate the CA Certificate

Once we have [Tunnelblick][tunnelblick] installed, we can use the built-in copy of [easy-rsa][easy-rsa] to generate all the certs we need.  If you chose to install an alternative client in step 1, then you need to get [easy-rsa][easy-rsa] on your own.

Tunnelblick installs easy-rsa in your local home directory:

    $ cd "~/Library/Application Support/Tunnelblick/easy-rsa"

First thing we have to do is export the nice default variables they provide in `vars` and clean the environment.  You can change them, but they are just defaults and you will be prompted for them later.

    $ . vars
    $ ./clean-all

This has now created the `keys/` directory in this folder.  This is where all the goodness will go.

Okay, now that everything is setup, we can finally make our global CA, the thing we're going to use to sign all the device certificates.  You can easily make this by running:

    $ ./pkitool --initca

You will now have two very important files:

    -rw-r--r--   1 root  wheel  1387 Feb 17 18:49 ca.crt
    -rw-------   1 root  wheel   890 Feb 17 18:49 ca.key

The `ca.crt` needs to be made available to all the devices you want to connect with (and serve from), but the `ca.key` should be kept secret (as it can be used to sign new device certificates).

## Step 3: Generate the Server's Certificate

Now that you have a CA, you can start creating some device certificates!  Start by making one for the server:

    $ ./build-key-server server

***NOTE: It's very important NOT to pick a password for this certificate, otherwise DD-WRT is not going to be able to decrypt it***

This process will generate some new files which will be needed for the server:

    -rw-r--r--   1 root  wheel  4174 Feb 17 18:50 server.crt
    -rw-------   1 root  wheel   890 Feb 17 18:50 server.key

These two will be used later in the DD-WRT configuration, so hang on to them.

## Step 4: Generate Diffie-Hellman Parameters

Now that we have a server certificate, we can make the Diffie-Hellman parameters for the SSL/TLS connection.

Easily done by running:

    $ ./build-dh

And you'll get a fantastic pem file like this one:

    -rw-r--r--   1 root  wheel   244 Feb 17 18:50 dh1024.pem

We'll use this later when configuring DD-WRT.

## Step 5: Generate your Device Certificates

We're going to make two devices here, an iPhone and Laptop.  I'm going to call the devices by those names in this tutorial, but your names should be more unique to the devices themselves (like hostnames for example).

    $ ./build-key iphone
    $ ./build-key laptop

***NOTE: I recommend creating passwords for these, but you don't HAVE to.***

This will generate us similar files as the previous steps:

    -rw-r--r--   1 root  wheel  4001 Feb 17 18:50 iphone.crt
    -rw-------   1 root  wheel   883 Feb 17 18:50 iphone.key
    -rw-r--r--   1 root  wheel  4014 Feb 17 18:50 laptop.crt
    -rw-------   1 root  wheel   882 Feb 17 18:50 laptop.key

Hang on to them for your individual device setup.

## Step 6: Configure DD-WRT VPN Settings

Now go to your router's VPN tab under Services and let's start flipping some settings!  All of these are under "OpenVPN Server/Daemon".

### Basic Settings

 - OpenVPN: `Enable`
 - Start Type: `WAN Up` (we want to have internet before starting)
 - Config as: `Server` (we're a server, yeah?)
 - Server Mode: `Router (TUN)` (I don't know)
 - Network: `10.0.2.0` (I chose this as it's a dedicated segment of my network, my normal routing is `10.0.0.0` in this network)
 - Netmask: `255.255.255.0`
 - Port: `1194` (I like to leave the default here)
 - Tunnel Protocol: `UDP` (and here)
 - Encryption Cipher: `Blowfish CBC`
 - Hash Algoritm: `SHA1`
 - Advanced Options `Disable` (don't need any of that)

Now for the big text fields, we're going to go through them one by one.

### Public Server Cert
Take the `server.crt` and find where `-----BEGIN CERTIFICATE-----` starts.  Copy from there until the end of the file, including the line `-----END CERTIFICATE-----`.  Put this in that field.

### CA Cert
Just stick the contents of `ca.crt` in here.

### Private Server Key
The contents of `server.key` go in here.

### DH PEM
This is where we put the `dh1024.pem` contents.

### Additional Config
For here, we don't need to do much.  I just have a couple basic tweaks I add:

    # Route traffic from this new network into my existing network
    push "route 10.0.0.0 255.255.255.0"

    # DNS server is 10.0.0.1, a.k.a. my router.
    push "dhcp-option DNS 10.0.0.1"

    # Use the tun0 device (so we can refer to it later)
    dev tun0

    # Ping every 10 seconds, and restart after 120 seconds of inactivity
    keepalive 10 120

Okay, that's it for this page.  Save and Apply changes!

## Step 7: Configure the Firewall

Now that we have the service running, there's some final configuration required to ensure that the VPNed traffic can talk to the internet AND the intranet (inside your network).  Go to `Administration` and then `Commands`.

This part may look a little scary, but it's very simple when you break down each command.

We're going to put the following Firewall configuration into the `Command Shell` / `Commands` section.

    # Accepts incoming traffic via port 1194 UDP
    iptables -I INPUT 1 -p udp --dport 1194 -j ACCEPT

    # Allows the VPN client access to router internal
    # processes, e.g. Web admin, SSH etc
    iptables -I INPUT 3 -i tun0 -j ACCEPT

    # Allows connections between VPN clients, if
    # client-to-client is enabled in OpenVPN server
    iptables -I FORWARD 3 -i tun0 -o tun0 -j ACCEPT

    # Allows connection from local VPN to the internet
    iptables -I FORWARD 1 --source 10.0.2.0/24 -j ACCEPT
    iptables -t nat -A POSTROUTING -s 10.0.2.0/24 -j MASQUERADE

    # Allows connections from local network to VPN network
    # and other way around (br0 is LAN and WIFI)
    iptables -I FORWARD -i br0 -o tun0 -j ACCEPT
    iptables -I FORWARD -i tun0 -o br0 -j ACCEPT

Great, now hit `Save Firewall` and your server is done!

## Step 8: Configure your devices

Now that you have a server, you need to setup your different devices.  Like I said before, we're going to stick to the `laptop` and `iphone` device types we made before.

### iPhone
 - Install OpenVPN Client (free on AppStore)
 - Create a folder to store stuff in locally (let's call it `iphone/`)
 - Put the `iphone.key`, `iphone.crt`, and `ca.crt` in this directory
 - Create a new file called `config.ovpn` with the contents:

        # Specify that we are a client and that we
        # will be pulling certain config file directives
        # from the server.
        client

        # Use the same setting as you are using on
        # the server.
        dev tun0

        # Are we connecting to a TCP or
        # UDP server?  Use the same setting as
        # on the server.
        proto udp

        # The hostname/IP and port of the server.
        remote YOURHOSTNAMEHERE 1194

        # Keep trying indefinitely to resolve the
        # host name of the OpenVPN server.  Very useful
        # on machines which are not permanently connected
        # to the internet such as laptops.
        resolv-retry infinite

        # Most clients don't need to bind to
        # a specific local port number.
        nobind

        # Try to preserve some state across restarts.
        persist-key
        persist-tun

        # SSL/TLS parms.
        ca ca.crt
        cert iphone.crt
        key iphone.key

        # Verify server certificate by checking
        # that the certicate has the nsCertType
        # field set to "server".  This is an
        # important precaution to protect against
        # a potential attack discussed here:
        #  http://openvpn.net/howto.html#mitm
        ns-cert-type server

        # Enable compression on the VPN link.
        comp-lzo

        # Allow me to change my IP address
        # and/or port number (if I get a new
        # local IP address at Starbucks).
        float

 - Open iTunes, go to your device, then Applications.  At the very bottom you will see `OpenVPN`, click on it.  Drag all four files you created in the `iphone/` folder into the box on the right.  This uploads them into the device.  Click `sync`.
 - Open up the app on your phone, it will allow you to create a new profile based on this input.
 - Done!  Connect to your network!

# Laptop
Okay, the reason I said to do the iPhone first, was that the Laptop is basically the same thing!

 - Create a folder to store stuff in locally (let's call it `HomeVPN/`)
 - Put the `laptop.key`, `laptop.crt`, and `ca.crt` in this directory
 - Create a new file called `config.ovpn` with similar contents as before (but replace `iphone` with `laptop`)
 - Rename the folder to `HomeVPN.tblk/`
 - Double-click it and import into Tunnelblick

 [tunnelblick]: https://www.tunnelblick.net/
 [tunnelblick-download]: https://www.tunnelblick.net/downloads.html
 [easy-rsa]: https://github.com/OpenVPN/easy-rsa
 [homebrew]: http://brew.sh/
 [cask]: http://caskroom.io/
