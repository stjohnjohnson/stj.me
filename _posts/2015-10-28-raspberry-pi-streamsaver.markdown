---
layout: post
title: Raspberry Pi Streamsaver
date: 2015-10-28
---

Personally, I'm a huge fan of home automation.  Ever since my wife and I bought
our house, I've been trying to find ways to secure and automate it.  This obviously
included the basic toys like the [Amazon Echo][echo] and [Nest Thermostat][nest].

But one of the other areas I'm poking around in is security cameras, specifically
monitoring the entrances of the house.  I live in a quiet neighborhood, but I do
hear a lot of complaints on [NextDoor][nextdoor] of people stealing packages.

This always made me uncomfortable, so I consider the cameras as a method to
[enhance my calm.][twit]

Anwyway, so the problem I seem to have now is that the two cameras I purchased
([D-Link DCS-932LB1][dlink] and a [Foscam FI8910W][foscam]) will only perform two
operations on motion/sound detection:
 - Send email of 1 picture per second for 6 seconds
 - Upload same pictures to FTP server

It works, kinda.  It's just not ideal.

So how can I make this better?  Bam, spare Raspberry Pi!

<!--more-->

# Motion Alerts

I wanted to trigger recording of the ASF or MJPEG stream from the cameras if
motion is detected.  The first idea I had was to run a fake FTP server and
get notifications that way.

I was super excited to see someone else had the same idea and wrote up a
[blog post about it][foscamblog1].  Now this was 3 years ago and they wrote everything
in shell (yuck).  Apparently he came back and decided to do this in C++
[a year later][foscamblog2].  For some insane reason, he didn't provide links to
any of his source code, only screenshots of it in vim.

I'm not a C programmer and I don't want to visually copy/paste this guy's code.
But at least I felt I was on the right track.

His approach was to use GStreamer (so have others in the past) on the Raspberry Pi
so you can leverage the H264 support.  So I got started with a vanilla
[Raspbian][raspbian] installation and some basic instructions.

# GStreamer Troubles

Most blogs/guides were around installing/compiling 1.0 as 0.10 was the only version
available on Raspbian at the time.  Luckily it's now available as part of their
latest release.

A quick installation of the extra plugins and I was ready to start (I think).
{% gist stjohnjohnson/763d6799ce573e8113d8 apt-get %}

Understanding GStreamer and its pipeline is extremely confusing.  Documentation
is quite sparse.  But this is what I hobbled together:
{% gist stjohnjohnson/763d6799ce573e8113d8 recommended %}

This, kinda worked.  But the quality was absolutely impossible:
![Bad Image]({% asset_path badimage.png %})

At this point, I gave up and moved onto another project (hoping to pick this up later).

[nextdoor]: https://nextdoor.com/amazon/?r=nrhndd
[dlink]: http://amzn.to/1Hcd6h4
[foscam]: http://amzn.to/1HccZCb
[echo]: http://amzn.to/1Hcdb4r
[nest]: http://amzn.to/1Hcde00
[twit]: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes
[foscamblog1]: http://wrice.blogspot.com/2013/07/how-to-setup-foscam-ip-camera-for.html
[foscamblog2]: http://raspberrypiprogramming.blogspot.com/2014/09/foscam-video-capture-server-at-motion.html
[raspbian]: http://raspbian.org
