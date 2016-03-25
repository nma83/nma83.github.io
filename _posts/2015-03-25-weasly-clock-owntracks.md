---
layout: post
title: Making a Weasly clock on top of OwnTracks
---

_This page is under construction._

[OwnTracks](http://owntracks.org) is an open-source app for sharing device location with other devices.
It uses the MQTT standard for communicating location updates. The protocol is generic and can be implemented using any MQTT broker.
I chose to use my existing [OpenShift](http://openshift.redhat.com) account which comes with 3 free instances for the MQTT broker.

## MQTT broker on OpenShift

* Create a [Node.js](https://developers.openshift.com/en/node-js-overview.html) application on OpenShift
* Install the [Mosca](https://github.com/mcollina/mosca) MQTT broker in that application
* Use MongoDB for persistence and set the DB up for the application
* Since OpenShift exposes only the [HTTP and WebSocket](https://developers.openshift.com/managing-your-applications/port-binding-routing.html) ports, [attach the Mosca broker](https://github.com/mcollina/mosca/wiki/MQTT-over-Websockets) to the HTTP server of the application
* The MQTT client can now connect to this broker using WebSocket protocol
* **TODO** Optionally, setup user authentication and ACL 

## Patching OwnTracks

* Use this [fork of OwnTracks](https://github.com/nma83/android) to get a WebSocket connection to the broker created in the previous step
* **TODO** Implement the clock-face in OwnTracks using available location and waypoint data
