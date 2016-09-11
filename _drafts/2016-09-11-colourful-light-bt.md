---
layout: post
title: Notes on controlling a BLE LED light
---

Android app that controls the light: [ColorfulLight](https://play.google.com/store/apps/details?id=com.cloudlink.bleled)

Usage of `hcitool` and/or `gatttool` to control this light from Linux.

```shell
$ sudo hcitool -i hci0 lecc --random FD:E8:56:E1:40:1F
Returns a handle
$ sudo hcitool -i hci0 ledc <handle>

$ sudo gatttool -b FD:E8:56:E1:40:1F -I -t random -l medium
> connect
# Full brightness
> char-write-cmd 0x0010 0x02f5f5f5f5
# Off 
> char-write-cmd 0x0010 0x0200000000

Other op codes TBD
```

Useful links:

* https://dobots.nl/2014/07/23/linux-and-ble/
* https://github.com/Betree/magicblue/wiki/How-to-use-manually-with-Gatttool

