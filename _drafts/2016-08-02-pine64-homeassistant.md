---
layout: post
title: Pine64 Home-assistant + MQTT setup
---

My notes on how I setup a Pine64 to be a headless server for my
home-assistant and MQTT broker, with only an SD-card, 5V/2A power
supply and Ethernet cable in hand.

Commands are denoted by a leading `$>` and text which goes into files
is left bare.

# Install basic image
* Download debian jessie image (longsleep)
* Mount the image on a host machine, either as a loop device or after writing to an sd-card
* Edit /etc/network/interfaces in the image for setting eth0 to static
  IP by adding:

```
auto eth0
iface eth0 inet static
   address 192.168.1.z
   netmask 255.255.255.0
   gateway 192.168.1.1
```

* The `z` in `192.168.1.z` is any number that makes the whole IP address unique in your LAN
* sshd is enabled by default in this image
* So hook eth to your LAN and boot!
* You should be able to ssh into debian@192.168.1.z from the host

# Strip down

* Disable booting into the GUI

        $> sudo systemctl set-default multi-user.target

* Most of the GUI utilities like gimp, kodi etc are redundant in a
  headless system, so they can be removed
  
        $> sudo apt-get --purge remove <pkg>
  
* __TBD__ Try to follow steps in https://wiki.debian.org/ReduceDebian

# Install home-assistant

[Home-assistant](https://home-assistant.io) is a fantastic home
automation platform. I will be using it along with
[Mosquitto](https://mosquitto.org) as an MQTT broker and
[OwnTracks](https://owntracks.org) as a device tracker. Mosquitto is
installed and enabled to run on boot by default in the debian images.

* Install Home-assistant

        $> sudo apt-get install python3-pip
        $> sudo pip3 install homeassistant

* Enable basic HTTP authentication in `~homa/.homeassistant/configuration.yaml`

        http:
          api_password: <password>
        
* Create a user under whom home-assistant will be run (`homa` in this example)

        $> sudo useradd -m -g sudo -s /bin/bash homa

* `homa` is added to `sudo` group initially for convinience
* Create a `systemd` service to run home-assistant at boot as `homa`
* Create a file named
  `/lib/systemd/system/home-assistant@homa.service`, with below contents

        [Unit]
        Description=Home Assistant for %i
        After=network.target mosquitto.service
        Wants=mosquitto.service

        [Service]
        Type=simple
        User=%i
        ExecStart=/usr/local/bin/hass --runner
        SendSIGKILL=no
        RestartForceExitStatus=100

        [Install]
        WantedBy=multi-user.target

* Enable and start the service

        $> sudo systemctl --system daemon-reload
        $> sudo systemctl enable home-assistant@homa
        $> sudo systemctl start home-assistant@homa

* Dependencies on `mosquitto` are added to ensure home-assistant can
  connect to the broker
* You should have a working default home-assistant setup now
  * Try it out at: http://192.168.1.z:8123

# Setup mosquitto

* The default `mosquitto` installation has no authentication and ACL
  setup
* Edit the config at `/etc/mosquitto/conf.d/mosquitto.conf` to add:
  * Disallow anonymous
  
        allow_anonymous false
  
  * Setup password and ACL files (we will create the files later)


        password_file /etc/mosquitto/passwd
        acl_file /etc/mosquitto/acl

 * `mosquitto` users and passwords have to be setup using the
   `mosquitto_passwd` command
 * First create the passwd file
  
        $> sudo mosquitto_passwd -c /etc/mosquitto/passwd user1
        Enter the password at the following prompts

* Then for subsequent users, remember to change the option to update
  the file (-U)
    
        $> sudo mosquitto_passwd -U /etc/mosquitto/passwd user2
        
* Now we can create the ACL for these users (`user1` and `user2`),
  assuming we are dealing with OwnTracks talking to this broker. Edit
  `/etc/mosquitto/acl` and add these lines:
  
        user user1
        topic owntracks/#
        user user2
        topic owntracks/#
        user homa
        topic read owntracks/#
        
  * The first 4 lines give read/write access to the topic
    `owntracks/#` (`#` is a
    [multi-level wildcard](https://mosquitto.org/man/mqtt-7.html))
    to the users added 
  * I am assuming a user `homa` was added for home-assistant to access
    the broker, which needs just read access
* Restart `mosquitto` for the changes to take effect

        $> sudo service mosquitto restart

# Connect home-assistant to mosquitto

* Add the `mqtt` section in `~homa/.homeassistant/configuration.yaml`

        mqtt:
          broker: localhost
          port: 1883
          client_id: home-assistant-1
          keepalive: 60
          username: homa
          password: <passwd for homa in mosquitto>
          protocol: 3.1

* Add the OwnTracks device tracker module

        device_tracker:
          platform: owntracks
          max_gps_accuracy: 200

# Open the gates to the Internet

The Home-assistant and Mosquitto instances are accessible to only the
LAN at this point. To expose these to the big, bad Internet, their
ports must be forwarded out. Add the following ports to
the port-forwarding table in the router.

* Port 8123 for Home-assistant
* Port 1883 for Mosquitto

# Security!

This setup is highly insecure since all communication is in
plain-text. The next post will be about adding a layer of security
using TLS to both Home-assistant and Mosquitto.
