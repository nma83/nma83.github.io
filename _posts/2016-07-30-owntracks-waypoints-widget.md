---
layout: post
title: Owntracks Waypoints widget
---

# The 'Why?'

I always wanted to build a [Weasley
clock](http://harrypotter.wikia.com/wiki/Weasley_Clock) from the Harry
Potter books, an app that shows locations of people in a
clock-face. After discovering the [OwnTracks](http://owntracks.org)
open-source app for location sharing, I began to explore building this
idea on top of the Owntracks infrastructure. The result is the
__Owntracks Waypoint widget__.

![Owntracks Waypoint widget]({{ site.url }}/assets/owntracks_widget.jpg)

# The 'What?'

Owntracks Waypoint widget is an Android home-screen widget that is
part of the Owntracks app (currently in [my
fork](https://github.com/nma83/android/tree/waypoints_widget) of
it). It uses 2 pices of information from each user who has shared
his/her location:

 * their current location
 * their shared waypoints

The widget compares the location with all possible pairs of waypoints
and (possibly) comes up with a pair that bounds the current location
best. The waypoints from this pair are shown on the ends of a linear
scale and the current location is placed on the scale based on
distance from each end.

This scale is created for each user from the 'Friends' list in the app
who meet the below criteria:

 * more than 1 waypoint is shared
 * the shared waypoints are exported (see below)
 * a valid location report exists
 * the location falls between 2 of the shared waypoints

And every device involved must be connected to a private MQTT broker
(which is an Owntracks restriction for sharing waypoints).
The widget also attempts to send a remote command to fetch waypoints
(set as shared) from every 'Friend'. If remote commands are disabled,
each user must explicitly export their shared waypoints using the
`Export->Export waypoints to endpoint` menu item.

# The 'How?'

_Will be continued..._
