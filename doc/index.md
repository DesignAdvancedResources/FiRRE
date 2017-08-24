FiRRE Development Documentation
===============================

Batteries
---------

 * [great guide about LiPo batteries](https://rogershobbycenter.com/lipoguide/)

Telemetry
---------

There are [several possible alternatives], currently we are using [QGroundControl] both on desktop and mobile.

[several possible alternatives]: http://ardupilot.org/copter/docs/common-choosing-a-ground-station.html

[QGroundControl]: http://qgroundcontrol.com

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

    v4l2-ctl -d /dev/videoX

the loopback device should have the following properties:

    Width/Height: 80/60
    Pixel Format: 'RGB3'
    Frames per second: 30.000 (30/1)

on another terminal (you can use `tmux` to do this if you are connecting via ssh) we can start streaming from this device using gstreamer:

    gst-launch-1.0 v4l2src device=/dev/videoX ! tcpserversink host=127.0.0.1 port=999

on another computer use the following pipeline to test the stream:

    gst-launch-1.0 tcpclientsrc host=firrecam.local port=9999 ! rawvideoparse width=80 height=60 framerate=30/1 format=15 ! videoconvert ! autovideosink

Custom Servo Release Mechanism
------------------------------

