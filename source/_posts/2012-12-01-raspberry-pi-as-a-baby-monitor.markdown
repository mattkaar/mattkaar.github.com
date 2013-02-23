---
layout: post
title: "Raspberry Pi as a Baby Monitor"
date: 2012-12-01 10:41
comments: true
categories: 
---

As we headed out of town for Thanksgiving, I was in search of my next tech project. On a whim I threw a [Raspberry Pi](http://www.raspberrypi.org) and Yeti USB microphone in my bag, determined to create a networked, high-fidelity baby monitor.

It turned out to be easier than I thought.

{% img /images/mk/raspberry_pi_monitor.jpg 315 315 %}

<!-- more -->

First, the Yeti worked without needing a powered USB hub… score. Then Raspbian picked it out without any issues.

    $ arecord -l
    **** List of CAPTURE Hardware Devices ****
    card 1: Microphone [Yeti Stereo Microphone], device 0: USB Audio [USB Audio]
      Subdevices: 1/1
      Subdevice #0: subdevice #0

Finally, I stumbled across [this excellent blog post](http://mutsuda.com/2012/09/07/raspberry-pi-into-an-audio-spying-device/) from [@mutsuda](https://twitter.com/mutsuda) that shows how to use `aplay` and `arecord` to send captured audio to a remote system for playback. With these instructions, you'll need another Raspberry Pi or separate Linux system for the listening side.

{% tweet https://twitter.com/Raspberry_Pi/status/271617154087002112 %}

Here we go. First, make sure the levels are correct for both input and output (mine were muted by default).

    $ alsamixer

Then run this test capture on the Raspberry Pi.

    $ arecord -D plughw:1,0 -f cd -d 10 test.wav

And then this should play the test file out of headphone jack on the RPi.

    $ aplay test.wav

Finally, to send sound to a remote system, give this a shot.

    $ arecord -D plughw:1,0 -f dat | ssh -C user@remoteip aplay -f dat

You should hear remote audio out of the far end provided those mixer settings are correct too.

Enjoy your new Raspberry Pi baby monitor!
