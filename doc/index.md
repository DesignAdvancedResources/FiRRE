FiRRE Drone Development Notes
=============================

Power
-----

 * [great guide about LiPo batteries](https://rogershobbycenter.com/lipoguide/)
 * [power parts terminology explanation (ESC, BEC, SBEC, UBEC, Opto)](https://hobbyking.com/media/file/57396352X1338440X50.pdf)

Calibration
-----------

 * [PixHawk Video Series](https://www.youtube.com/watch?v=uH2iCRA9G7k)
 * make sure the Pixhawk and the other sensors (e.g GPS/compass) are on the same reference frame (i.e. same plane and orientation)
 * use ground station to check sensor state after calibration
 * make sure elevator channel is reversed
 * if aiming for autonomous flight, we shouln't trim the radio (or at least trim the radio, copy the trims to the drone and then reset the radio trims)

Radio Control
-------------

 * [Using Taranis Radio with Pixhawk](http://open-txu.org/home/special-interests/multirotor/opentx-apm-px4-pixhawk/)

Telemetry
---------

There are [several possible alternatives], we have tested both [APM Planner] and [QGroundControl] on desktop and [QGroundControl] and [Tower] on mobile.

Video stream telemetry inside the ground station is tricky to get working right. [APM Planner]'s implementation currently doesn't compile and [QGroundControl]'s requires a [very specific structure].

It is possible to use the Raspberry Pi Zero W as a sort of telemetry (e.g. for video streaming), but first it is necessary to [set it up as a wifi access point] to make connecting to it a bit easier.

[several possible alternatives]: http://ardupilot.org/copter/docs/common-choosing-a-ground-station.html

[QGroundControl]: http://qgroundcontrol.com

[APM Planner]: http://ardupilot.org/planner2/

[Tower]: https://github.com/DroidPlanner/Tower

[very specific structure]: https://github.com/mavlink/qgroundcontrol/blob/master/src/VideoStreaming/README.md

[set it up as a wifi access point]: https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md

LWIR Camera
-----------

 * [FLIR Lepton Module](https://cdn.sparkfun.com/datasheets/Sensors/Infrared/FLIR_Lepton_Data_Brief.pdf)
 * [Pure Engineering Breakout Module](http://www.pureengineering.com/projects/lepton)
   * [datasheet](https://drive.google.com/file/d/0B3wmCw6bdPqFYlE1aUFTOWh6c3c/edit))
   * [example code](https://github.com/groupgets/LeptonModule)
 * [3D-printed Lepton Breakout Case](https://www.thingiverse.com/thing:1563825)
 * [Raspberry Pi hookup guide](https://learn.sparkfun.com/tutorials/flir-lepton-hookup-guide)

### Using the example code and gstreamer to test video streaming

build and install the v4l2loopback module:

    sudo apt-get install raspberrypi-kernel-headers
    git clone https://github.com/umlaeute/v4l2loopback.git
    cd v4l2loopback
    make && sudo make install

build and install the v4l2lepton app:

    git clone https://github.com/groupgets/LeptonModule.git
    cd LeptonModule/software/v4l2lepton
    make && sudo make install

start v4l2lepton:

    sudo modprobe v4l2loopback
    v4l2lepton /dev/videoX

(/dev/videoX should be replaced by the name of the real device created by the v4l2loopback module, i.e. /dev/video0 if no camera is attached to the pi)

use v4l2-ctl to check the device is working ok:

    v4l2-ctl -d /dev/videoX --all

the loopback device should have the following properties:

    Width/Height: 80/60
    Pixel Format: 'RGB3'
    Frames per second: 30.000 (30/1)

on another terminal (you can use `tmux` to do this if you are connecting via ssh) we can start streaming from this device using gstreamer:

    gst-launch-1.0 v4l2src device=/dev/videoX ! tcpserversink host=127.0.0.1 port=9999

on another computer use the following pipeline to test the stream:

    gst-launch-1.0 tcpclientsrc host=firrecam.local port=9999 ! rawvideoparse width=80 height=60 framerate=30/1 format=15 ! videoconvert ! xvimagesink sync=false

To output the stream to the HDMI port we can use this pipeline:

    gst-launch-1.0 -v videotestsrc ! fbdevsink

To show the module's output we can use:

    gst-launch-1.0 v4l2src device=/dev/video0 ! videoparse width=80 height=60 framerate=10/1 format=15 ! videoconvert ! videoflip method=5 ! videoscale ! video/x-raw,width=640,height=480,framerate=10/1 ! fbdevsink

### Sidenote: run script after boot completes

To automatically run a command or set of commands after the Raspberry Pi finishes booting, we need to add these commands to the /etc/rc.local file.

### Sidenote: disable console blanking

In the default configuration, the Raspberry Pi console will blank after about 10 minutes of keyboard inactivity. To avoid having the video stream cut abruptly by this, we need to add `consoleblank=0` to the kernel command line or call `setterm --blank 0` from the console.

### Sidenote: create wifi access point

Custom Servo Release Mechanism
------------------------------

 * [Controlling servos in Arducopter](http://ardupilot.org/copter/docs/common-servo.html)

[BEC](https://en.wikipedia.org/wiki/Battery_eliminator_circuit)

Fire Tracker
------------

### Libraries to evaluate

 * App structure
   * [asyncio](https://docs.python.org/3/library/asyncio.html) (requires python3)
   * [kaa-base](http://api.freevo.org/kaa-base/)
   * [rospy](http://wiki.ros.org/rospy/) (useful if using other ROS packages)
 * Lepton module control
   * [pylepton](https://github.com/groupgets/pylepton)
 * drone control
   * [dronekit-python](http://python.dronekit.io)
   * [MAVLlink protocol](http://mavlink.org)
   * [ROS](http://ros.org)
 * computer vision
   * [opencv](http://opencv.org)
   * [simplecv](http://simplecv.org)
 * framebuffer graphics
   * [fbpy](https://pythonhosted.org/fbpy/)
   * [kaa-display](https://github.com/freevo/kaa-display)
   * [pyopengles](https://github.com/peterderivaz/pyopengles)
   * [pygame](https://www.pygame.org)
