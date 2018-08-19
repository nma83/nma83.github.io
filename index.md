---
layout: page
---

{:center: style="text-align: center"}

# Narendra Acharya
{:center}
ASIC engineer for profit, hobbyist programmer for fun.
{:center}

## My projects

### Desktop
* Enhance face detection in [shotwell](https://gitlab.gnome.org/GNOME/shotwell) and add face recognition using DNN

### Android
* [Owntracks Waypoint widget](https://github.com/nma83/android) - inspired by the [Weasley clock](http://harrypotter.wikia.com/wiki/Weasley_Clock) in the Harry Potter books, an app that shows locations of people with respect to waypoints, built on top of [OwnTracks](http://owntracks.org)
* [StorageTrac](/SDCardTrac) - track phone internal storage usage

### Web
* Enhance [Home-assistant](https://home-assistant.io) zone plugin to capture waypoint list published from [OwnTracks](http://owntracks.org)

### Emacs
* [diff-actor](https://github.com/nma83/diff-actor) - Act out diff's between files in Emacs
* [org-autonum](https://github.com/nma83/org-autonum) - automatic section numbering in [org-mode](http://orgmode.org)

## What I would like to do

### Web
* Photo (stored on Google/Facebook) flashbacks based on Hindu calendar events
* An authentication and ACL management web app for the [Mosquitto](https://mosquitto.org) broker, using OpenID and [mosquitto-auth-plug](https://github.com/jpmens/mosquitto-auth-plug)

### Android
* Encourage kids to hand-write code on paper that can then be scanned by an app to execute on device

### Tools
* WiT - Waveform in Text - show digital waveforms in ASCII art, want to write in Haskell using ncurses bindings

## Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

## Get in touch

<b>[Twitter](http://twitter.com/narendra_m_a)</b> | <b>[LinkedIn](http://in.linkedin.com/in/narendrama)</b> |
<b>[Ello](http://ello.co/nma83)</b>
{:center}
