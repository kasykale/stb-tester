======
 stbt
======

----------------------------------------------------
A record + playback testing system for set-top boxes
----------------------------------------------------

:Copyright: Copyright © 2012 YouView TV Ltd.
:License: LGPL v2.1 or any later version (see LICENSE file for details)
:Version: fac6848
:Manual section: 1
:Manual group: stb-tester

SYNOPSIS
========

stbt record *hostname* > *testscript*

stbt run *testscript* *hostname*


DESCRIPTION
===========

**stbt record** will record a test case by listening for remote-control
keypresses, taking screenshots from the set-top box as it goes.

You then (manually) crop the screenshots to the region of interest.

(Optionally) you manually edit the generated test script, which will look
something like this::

    press("MENU")
    wait_for_match("Guide.png")
    press("OK")
    wait_for_match("BBC One.png")

**stbt run** will play back the given test script, returning an exit status of
success or failure for easy integration with your existing test reporting
system.


HARDWARE REQUIREMENTS
=====================

The test rig consists of a Linux server, with:

* A video-capture card (for capturing the output from the system under test)
* An infrared receiver (for recording test cases)
* An infrared emitter (for controlling the system under test)

Video capture card
------------------

You'll need a capture card with drivers supporting the V4L2 API
(Video-for-Linux 2). We recommend a capture card with mature open-source
drivers, preferably drivers already present in recent versions of the Linux
kernel.

The Hauppauge HD PVR works well (and works out of the box on recent versions of
Fedora), though it doesn't support 1080p. If you need an HDCP stripper, try the
HD Fury III.

Infra-red emitter and receiver
------------------------------

An IR emitter+receiver such as the RedRat3, plus a LIRC configuration file
with the key codes for your set-top box's remote control.

Using software components instead
---------------------------------

If you don't mind instrumenting the system under test, you don't even need the
above hardware components.

stb-tester uses gstreamer, an open source multimedia framework. Instead of a
video-capture card you can use any gstreamer video-source element. For example:

* If you run tests against a VM running the set-top box software instead
  of a physical set-top box, you could use the ximagesrc gstreamer
  element to capture video from the VM's X Window.

* If your set-top box uses DirectFB, you could install the (not yet written)
  DirectFBSource gstreamer element on the set-top box to stream video to a
  tcpclientsrc or tcpserversrc gstreamer element on the test rig.

Instead of a hardware infra-red receiver + emitter, you can use a software
equivalent (for example a server running on the set-top box that listens on
a TCP socket instead of listening for infra-red signals, and your own
application for emulating remote-control keypresses). Using a software remote
control avoids all issues of IR interference in rigs testing multiple set-top
boxes at once.

Linux server
------------

We expect that an 8-core machine will be able to drive 4 set-top boxes
simultaneously with at least 1 frame per second per set-top box.
(TODO: Assuming enough bandwidth on the USB bus -- need to test this).


SOFTWARE REQUIREMENTS
=====================

* Drivers for any required hardware components

* python (we have tested with 2.6 and 2.7)

* gstreamer 0.10 (multimedia framework)

* OpenCV (image processing library) version >= 2.0.0 and <= 2.3.1
  (the version restrictions are imposed by gst-plugins-bad).

* gst-plugins-bad (for the gstreamer wrappers around OpenCV)
  built from source from the head of the 0.10 branch, with the patches from
  https://bugzilla.gnome.org/show_bug.cgi?id=678485
  (until such time as the patches are accepted upstream).

  A github repo with the same patches applied is available at
  https://github.com/drothlis/gst-plugins-bad (branch templatematch-fixes).


INSTALLING FROM SOURCE
======================

Run "make install" from the stb-tester source directory.

Requires python-docutils (for building the documentation).


TEST SCRIPT FORMAT
==================

The test scripts produced and run by **stbt record** and **stbt run**,
respectively, are actually python scripts, so you can use the full power of
python. Don't get too carried away, though; aim for simplicity, readability,
and maintainability.

The following functions are available:

* press("*key name*")

* wait_for_match("*filename.png*")

* press_until_match("*key name*", "*filename.png*")


TEST SCRIPT BEST PRACTICES
==========================

* When cropping images to be matched by a test case, you must select a region
  that will *not* be present when the test case fails, and that does *not*
  contain *any* elements that might be absent when the test case succeeds. For
  example, you must not include any part of a live TV stream (which will be
  different each time the test case is run), nor translucent menu overlays with
  live TV showing through.

* Don't crop tiny images: Instead of selecting just the text in a menu button,
  select the whole button. (Larger images provide a greater gap between the
  "match certainty" reported for non-matching vs. matching images, which makes
  for more robust tests).


SEE ALSO
========

* github.com/???


AUTHORS
=======

* Will Manley <will@williammanley.net>
* David Röthlisberger <david@rothlis.net>