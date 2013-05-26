---
layout: post
title: "Web Streaming with the Raspberry Pi Baby Monitor"
date: 2013-05-26 15:16
comments: true
categories: 
---

My [Raspberry Pi baby monitor](/blog/2012/12/01/raspberry-pi-as-a-baby-monitor) writeup got some love from the [Raspberry Pi site](http://www.raspberrypi.org/archives/2665) and [CNN](http://www.cnn.com/2012/12/21/tech/innovation/raspberry-pi-computer-upton).

To kick things up a notch, I added web streaming audio to the mix. With the help of [icecast2](http://www.icecast.org/) and [darkice](http://darkice.org/), the Raspberry Pi transcodes live audio to MP3 and broadcasts it over HTTP.

{% img /images/mk/icecast2.png 300 300 %}

<!-- more -->

[@t3node](http://twitter.com/t3node) wrote up an [excellent guide on live mp3 streaming using the Raspberry Pi](http://www.t3node.com/blog/live-streaming-mp3-audio-with-darkice-and-icecast2-on-raspberry-pi/). Using that post as a guide, here's what I did to get it working.

### Download darkice source

First, add the Raspbian source repository if it isn't already loaded. Add this line to `/etc/apt/sources.list`

    deb-src http://mirrordirector.raspbian.org/raspbian/ wheezy main contrib non-free rpi

Update packages and install the darkice dependencies.

    $ sudo apt-get update
    $ sudo apt-get --no-install-recommends install build-essential devscripts autotools-dev fakeroot dpkg-dev debhelper autotools-dev dh-make quilt ccache libsamplerate0-dev libpulse-dev libaudio-dev lame libjack-jackd2-dev libasound2-dev libtwolame-dev libfaad-dev libflac-dev libmp4v2-dev libshout3-dev libmp3lame-dev

Grab the source package for darkice.

    $ mkdir src; cd src/
    $ apt-get source darkice
    
Modify the compilation options to match Raspbian.

    $ cd darkice-1.0
    $ vi debian/rules
    
`debian/rules` should look like this:

``` makefile debian/rules
#!/usr/bin/make -f

%:
    dh $@

.PHONY: override_dh_auto_configure
override_dh_auto_configure:
    ln -s /usr/share/misc/config.guess .
    ln -s /usr/share/misc/config.sub .
    dh_auto_configure -- --prefix=/usr --sysconfdir=/usr/share/doc/darkice/examples --with-vorbis-prefix=/usr/lib/arm-linux-gnueabihf/ --with-jack-prefix=/usr/lib/arm-linux-gnueabihf/ --with-alsa-prefix=/usr/lib/arm-linux-gnueabihf/ --with-faac-prefix=/usr/lib/arm-linux-gnueabihf/ --with-aacplus-prefix=/usr/lib/arm-linux-gnueabihf/ --with-samplerate-prefix=/usr/lib/arm-linux-gnueabihf/ --with-lame-prefix=/usr/lib/arm-linux-gnueabihf/ CFLAGS='-march=armv6 -mfpu=vfp -mfloat-abi=hard'
```
                 
Beware, 'make' wants tabs instead of spaces. To be on the safe side, you can download [this rules file](/misc/debian/rules) instead.

Before you build the new .deb package, change the version to reflect MP3 support:

    $ debchange -v 1.0-999~mp3+1
    
Your editor will open to edit the changelog.
    
``` plain debian/changelog.dch

darkice (1.0-999~mp3+1) UNRELEASED; urgency=low

  * New build with mp3 support

 --  <pi@blueberrypi>  Sun, 26 May 2013 04:22:38 -0400
 
```

### Build and install darkice

Now build and install your custom darkice package.

    $ dpkg-buildpackage -rfakeroot -uc -b
    $ sudo dpkg -i ../darkice_1.0-999~mp3+1_armhf.deb

Darkice is now installed. To configure it, copy the template config file to `/etc`.

    $ sudo cp /usr/share/doc/darkice/examples/darkice.cfg /etc/

Here's my sample config file:

``` cfg /etc/darkice.cfg
# see the darkice.cfg man page for details

# this section describes general aspects of the live streaming session
[general]
duration      = 0                # duration of encoding, in seconds. 0 means forever
bufferSecs    = 5                # size of internal slip buffer, in seconds
reconnect     = yes              # reconnect to the server(s) if disconnected

# this section describes the audio input that will be streamed
[input]
device        = plughw:1,0       # Alsa soundcard device for the audio input
sampleRate    = 44100            # sample rate in Hz. try 11025, 22050 or 44100
bitsPerSample = 16               # bits per sample. try 16
channel       = 1                # channels. 1 = mono, 2 = stereo

# this section describes a streaming connection to an IceCast2 server
# there may be up to 8 of these sections, named [icecast2-0] ... [icecast2-7]
# these can be mixed with [icecast-x] and [shoutcast-x] sections
[icecast2-0]
bitrateMode   = vbr              # variable bit rate
format        = mp3              # format of the stream: mp3
quality       = 1.0              # quality of the stream sent to the server
bitrate       = 256
server        = localhost        # host name of the server
port          = 8000             # port of the IceCast2 server, usually 8000
password      = *SOURCE_PASS*    # source password to the IceCast2 server
mountPoint    = raspi            # mount point of this stream on the IceCast2 server
name          = RasPi            # name of the stream
description   = DarkIce on RasPi # description of the stream
url           = http://localhost # URL related to the stream
genre         = Baby             # genre of the stream
public        = no               # advertise this stream?
localDumpFile = recording.mp3    # Record also to a file
```

### Install icecast2 and begin monitoring

Now you'll need to install icecast2 to stream the audio over the network.

    $ sudo apt-get install icecast2
    
The installer will ask you for a few passwords, make sure you set the source password as the same one for darkice (`*SOURCE_PASS*` in the sample config above).

Now start icecast2 and darkice.

    $ sudo service icecast2 start
    $ sudo darkice
    DarkIce 1.0 live audio streamer, http://code.google.com/p/darkice/
    Copyright (c) 2000-2007, Tyrell Hungary, http://tyrell.hu/
    Copyright (c) 2008-2010, Akos Maroy and Rafael Diniz
    This is free software, and you are welcome to redistribute it
    under the terms of The GNU General Public License version 3 or
    any later version.

    Using config file: /etc/darkice.cfg
    Using ALSA DSP input device: plughw:1,0
    Using POSIX real-time scheduling, priority 98

Darkice will run in the foreground (CTRL+C to exit).

Now open up a browser and navigate to http://{Raspberry Pi IP}:8000. Click on **m3u** and your audio player should play the baby monitor audio. It even works on the iPhone.

The one downside is that there's a 5-10 second delay when using this method. But since I'm just interested in a baby monitor, 5-10 seconds doesn't matter that much. Also, take care to add some authentication and secure passwords before you try this over the Internet.