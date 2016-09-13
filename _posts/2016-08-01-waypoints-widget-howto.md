---
layout: post
title: How to setup the Owntracks Waypoints widget
---

The [Owntracks](https://owntracks.org) Waypoint widget is an Android
home-screen widget that summarises a 'Friend's location between a pair
of his/her published waypoints. This post is meant to be a
step-by-step guide for setting the widget up.

__At the time of writing, the widget is sitting in a
[pull-request](https://github.com/owntracks/android/pull/381) waiting
to be merged into the master branch__

For an overview of the widget, please read the [previous post]({%
post_url 2016-07-30-owntracks-waypoints-widget %}). 

# Basic requirements

* The following steps assume the Owntracks installation includes the
  widget (release TBD). 
* The device (and friends) must be connected to a private MQTT broker.
* The waypoints to be displayed to others must be set as shared waypoints

# Add the widget to the home screen

Select the 'Owntracks waypoints' widget from the widget selector.

![Home]({{ site.github.url }}/assets/owntracks-widget-add.png)

At first, the widget will be blank, asking you to configure it from Owntracks.

# Enable the widget in Owntracks

Communication to the widget from Owntracks is disabled by
default. Enable it from the menu item `Notifications->Waypoint widget`.

![Menu]({{ site.github.url }}/assets/owntracks-widget-menu.png)

# Export waypoints from other Friends

The list of waypoints to be displayed must be explicitly exported from
each Friend via the menu item `Export->Export waypoints to endpoint`.

# Done!

Once a location update is received from a Friend who has a set of
waypoints already shared and exported, the widget will start reporting
the current status as compared to the waypoints.

![Done](https://github.com/nma83/android/raw/waypoints_widget/project/app/src/main/res/drawable-nodpi/appwidget_preview.png)

