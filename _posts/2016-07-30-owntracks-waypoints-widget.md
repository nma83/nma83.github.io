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

![Owntracks Waypoint widget](https://github.com/nma83/android/raw/waypoints_widget/src/main/res/drawable-nodpi/appwidget_preview.png)

# The 'What?'

Owntracks Waypoint widget is an Android home-screen widget that is
part of the Owntracks app (currently in [my
fork](https://github.com/nma83/android/tree/waypoints_widget) of
it). It uses 2 pieces of information from each user who has shared
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

The widget processes data on 2 events:

 * when a new location report is available
 * when a list of waypoints are published

If a location report from a friend does not have a corresponding list
of waypoints defined, the app sends out a [waypoints command](http://owntracks.org/booklet/tech/json/) -
`{"_type":"cmd","action":"waypoints"}`. But this requires the other
devices to have remote commands enabled. If remote commands are not
enabled, waypoints can be exported manually as well from the `Export` menu.

Once a location report is available with more than 1 waypoint
available for that friend, the widget walks through each waypoint pair
and computes:

 * a bounding box from the coordinates of the waypoint pair
 * the perimeter of the bounding box
 * the Manhattan distance from the location to waypoints

All these computations assume a Cartesian coordinate system, they
would need to be improved to match the Earth's curvature. But this
system works for small distances (< 100Km) and away from the poles.
If the location falls into the bounding box of any of the waypoint
pairs, the widget picks the pair with the least perimeter and uses its
data to display on the screen. 

The widget UI consists of a [StackView](https://developer.android.com/reference/android/widget/StackView.html
) with each item in the StackView collection reserved for a
friend. This view is updated with labels on both ends for the
waypoints detected and a floating label between them on a scale for
the distance of current location from each end.

There is an additional status label below the scale which indicates
when the location is out of all available bounding boxes or when there
is no waypoint available.
