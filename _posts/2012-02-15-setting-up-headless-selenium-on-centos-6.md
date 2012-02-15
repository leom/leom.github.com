---
layout: post
title: "Setting up headless selenium on CentOS 6 w/ a custom Firefox profile"
categories: [centos, selenium, testing]
---
{% include JB/setup %}

So it turns out setting up a headless [Selenium RC](http://seleniumhq.org) instance is
relatively straightforward on CentOS 6, but there are a couple gotchas. Here's what I
had to do to get this hotness blazing. (Are you excited? You should be, because nothing
is sexier than the command line)

Oh! I'm going to bypass the "what is selenium" portion of this blog, because you're a smart cookie and there's
no need to go down that road. Alternatively, you could interpret that as "laziness" and I would not argue in the slightest 
(mainly because arguing requires effort).

## Setup directions

### 1. Go download selenium and put it into /opt/selenium
This should be straightforward. Go download [Selenium RC] (http://seleniumhq.org/download/) -
at the time of this writing is 2.19.0 - and put it into it's own directory in opt, then
symlink to that jar.

    #> cd /opt && mkdir selenium && cd selenium
    #> wget http://selenium.googlecode.com/files/selenium-server-standalone-2.19.0.jar
    #> ln -s selenium-server-standalone-2.19.0.jar selenium-server.jar

### 2. Run the following yum install command

This downloads xvfb along with some fonts, firefox, fluxbox, and x11vnc. I can hear your cries of
"But I thought we were running headless! why do I need a WM and VNC?" To this I say: 

1. You need to speak more softly. We aren't even in the same room. The fact I can hear you concerns me.
2. We are going to create a custom Firefox profile, and the best way to do that is through VNC.

If it really bugs you that much, as soon as the profile is created you can remove fluxbox and x11vnc.
Really, I promise. Now if you'll just keep your voice down, this walkthrough can continue.

    #> sudo yum install xorg-x11-server-Xvfb.i686 mesa-dri-drivers.i686 xorg-x11-fonts-100dpi.noarch xorg-x11-fonts-75dpi.noarch xorg-x11-fonts-cyrillic.noarch fluxbox x11vnc.i686 xterm.i686 firefox java-1.6.0-openjdk.i686 

### 3. Create the xvfb-service script.

I modified this extensively from http://www.labelmedia.co.uk/blog/posts/setting-up-selenium-server-on-a-headless-jenkins-ci-build-machine.html
but I figure the core concepts still apply. Oh, and by "I" I really mean my coworker Marco but I have no idea if he has
a blog so the credit goes to the ether!

Cut and paste this into /etc/init.d/xvfb-service

    #!/bin/bash -x
    #
    # Service to start Xvfb
    # chkconfig 2345 99 00
    #

    RETVAL=0
    start() {
            echo -n "Starting Xvfb"
            /usr/bin/Xvfb :99 -ac -screen 0 1280x1024x16 &
            }

    stop()  {
            killall Xvfb
            }

    case "$1" in
          start) 
              start ;;
          stop)
              stop ;;
          *)  echo "Usage $0 {start|stop}"
              RETVAL=1
    esac

    exit $RETVAL

Once that's done, start up xvfb with #>/etc/init.d/xvfb-service start

*If you get an error similar to:* `process 9741: D-Bus library appears to be incorrectly set up; failed to read machine uuid: Failed to open "/var/lib/dbus/machine-id": No such file or directory`
you need to generate a new machine-id. You can do that from the command line very easily:
    
    #> dbus-uuidgen > /var/lib/dbus/machine-id

### 4. Create the new Firefox profile.

Admittedly, this might be overkill. But, I absolutely hate using defaults with unmonitored systems, and I like to know
that there's a profile in FF dedicated solely to testing - so in the event something screws up with the default, it's 
all still very isolated and clean. Cleanliness is something something, after all!

Start fluxbox as our WM. The xvfb script binds to :99, so everything that ensues has to do the same.

    #> DISPLAY=:99 fluxbox &

Now start x11vnc so that we can create our profile.

    #> x11vnc -display :99 -bg -nopw -listen $IP_ADDRESS -xkb

Once that's done, fire up your VNC client and connect to `$IP_ADDRESS`. From within your VNC, open up an xterm
(left click on the desktop, and a menu will pop up) and open up firefox's profile manager.

    (from within your VNC session)
    $> firefox --profilemanager

Now, create a new profile and store it in a folder dedicated to the profile. Yeah, make sure you create a new
folder, or else it'll get real messy real fast. For my purposes, I've created the new profile in `/home/selenium/ff-selenium-profile`

All done? You can now close VNC, you silly goose. We're all done in the land of point and click!

### 5. Start selenium server

Now that everything's configured, you can start selenium server. 

    #> DISPLAY=:99 java -DfirefoxDefaultPath=/usr/lib/firefox-3.6/firefox -jar /var/lib/selenium/selenium-server.jar -debug -firefoxProfileTemplate "/home/selenium/ff-selenium-profile/"

It's important that you pass DISPLAY=:99, or else you will get errors like `DEBUG - waiting for window 'null' local frame 'null'` in the selenium logs
and that error is extremely frustrating. Also, since CentOS wraps the firefox binary with a script, just do an `$> ls /usr/lib/f*` to find out what the most recent
firefox version you have is.

You want to wrap this in a service script too? Well, just follow the [great directions on this blog](http://robfan.com/post/122618829/continuous-integration-selenium-firefox-flash#notes)
and you'll be all set. I'd also claim credit for it, but meh. Ther'es no need for that, now is there?
