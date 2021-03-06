## myo_raw_osc

- Added OSC built-in compatibility to the myo raw data
- Orientation angles accurately computed 
- Optimal usage with attached myo-firmware-0.8.18-revd (www.myo.com/firmwareupdate)
- for example: https://vimeo.com/151326521

## Usage: 
python myo_raw_osc.py -v -s -d -r -n -i ... 

    - -v --verbose: 0 or 1 \t print the messages. Default to 1

    - -s --send: 0 or 1 \t send/receive the data over OSC. Default to 1

    - -d --destination: [ip,port]  add an OSC client to where send the data
        ip 0 will expand to localhost 127.0.0.1
        multiple clients might be registered by reusing the -a option
        Default address set to "127.0.0.1",7110

    - -r --receive: [ip,port]  IP address and port where to receive OSC incoming messages (vibration)
        ip 0 will expand to localhost 127.0.0.1 
        
    - -n --donglename: specify a usb port name (Linux only).
        0 for /dev/ttyACM0, 1 for /dev/ttyACM1, etc
        if not specified, system will try to find one and use it (be careful with selecting an already used dongle)
        
     - -i --deviceid: [int] specify the desired device to be connected.
        If not, it will connect to first available device
        Please look at the code and change the signature according to your device signature


## Output OSC messages:
- [/myo/emg, (8 values with raw EMG data)]
- [/myo/imu, yaw(azimuth), roll, pitch(elevation), accX, accY, accZ]

## Input OSC messages:
- [/myo/vib, {integer between 1 (shorter) and 3 (longer)}]


## Examples:
- print the data without sending

python myo_raw_osc.py -v 1
- send to localhost port 57120 and remote ip 127.0.0.4 port 1235

python myo_raw_osc.py -v 0 -s 1 -d [0,57120] -d [127.0.0.4,12345]
        
  
## Dependencies
  - pyOSC (https://trac.v2.nl/wiki/pyOSC)
  - transforms3d (https://pypi.python.org/pypi/transforms3d)
  
## TODO
  - On-the-fly choose an availabe, not used usb dongle
  - Adapt to multi-platform

------------------------------------------------------------------------------

# myo_raw_osc_gui

- Plot in real-time all raw myo data channels
- Receives OSC messages from external process myo_raw_osc.py
- Please refer to myo_raw_osc.py for the incoming OSC messages format
- for example: https://vimeo.com/150127407

## Usage: 
python myo_raw_osc_gui.py -i -p

    - -i --ip: server ip address. Default to "localhost"

    - -p --port: server ip port. Default to 7110
 
## Dependencies
  - pyOSC (https://trac.v2.nl/wiki/pyOSC)
  - pyQtgraph (http://www.pyqtgraph.org/)

Andrés Pérez López ---> www.andresperezlopez.com

------------------------------------------------------------------------------
------------------------------------------------------------------------------
# Overview

This project provides an interface to communicate with the Thalmic Myo,
providing the ability to scan for and connect to a nearby Myo, and giving access
to data from the EMG sensors and the IMU. For Myo firmware v1.0 and up, access
to the output of Thalmic's own gesture recognition is also available.

The code is primarily developed on Linux and has been tested on Windows and
MacOS.

Thanks to Jeff Rowberg's example bglib implementations
(https://github.com/jrowberg/bglib/), which helped me get started with
understanding the protocol.


# Requirements

- python >=2.6
- pySerial
- enum34 (for Python <3.4)
- pygame, for the example visualization and classifier program
- numpy, for the classifier program
- sklearn, for a more efficient classifier (and easy access to smarter classifiers)


# Dongle device name

To use these programs, you might need to know the name of the device
corresponding to the Myo dongle. The programs will attempt to detect it
automatically, but if that doesn't work, here's how to find it out manually:

- Linux: Run the command `ls /dev/ttyACM*`. One of the names it prints (there
  will probably only be one) is the device. Try them each if there are multiple,
  or unplug the dongle and see which one disappears if you run the command
  again. If you get a permissions error, running `sudo usermod -aG dialout
  $USER` will probably fix it.

- Windows: Open Device Manager (run `devmgmt.msc`) and look under "Ports (COM &
  LPT)". Find a device whose name includes "Bluegiga". The name you need is in
  parentheses at the end of the line (it will be "COM" followed by a number).

- Mac: Same as Linux, replacing `ttyACM` with `tty.usb`.


# Included files

## myo_raw.py (access to EMG/IMU data)

myo_raw.py contains the MyoRaw class, which implements the communication
protocol with a Myo. If run as a standalone script, it provides a graphical
display of EMG readings as they come in. A command-line argument is interpreted
as the device name for the dongle; no argument means to auto-detect. You can
also press 1, 2, or 3 on the keyboard to make the Myo perform a short, medium,
or long vibration.

To process the data yourself, you can call MyoRaw.add_emg_handler or
MyoRaw.add_imu_handler; see the code for examples.

If your Myo has firmware v1.0 and up, it also performs Thalmic's gesture
classification onboard, and returns that information. Use MyoRaw.add_arm_handler
and MyoRaw.add_pose_handler. Note that you will need to perform the sync gesture
after starting the program (the Myo will vibrate as normal when it is synced).

## classify_myo.py (example pose classification and training program)

classify_myo.py contains a very basic pose classifier that uses the EMG
readings. You have to train it yourself; make up your own poses and assign
numbers (0-9) to them. As long as a number key is held down, the current EMG
readings will be recorded as belonging to the pose of that number. Any time a
new reading comes in, the program compares it against the stored values to
determine which pose it looks most like. The screen displays the number of
samples currently labeled as belonging to each pose, and a histogram displaying
the classifications of the last 25 inputs. The most common classification among
the last 25 is shown in green and should be taken as the program's best estimate
of the current pose.

This method works fine as long as the Myo isn't moved, but, in my experience, it
takes quite a large amount of training data to handle different positions
well. Of course, the classifier could be made much, much smarter, but I haven't
had the chance to tinker with it yet.

## myo.py (Myo library with built-in classifier and pose event handlers)

After you've done training with classify_myo.py, the Myo class in this file can
be used to notify a program each time a pose starts. If run as a standalone
script, it will simply print out the pose number each time a new pose is
detected. Use Myo.add_raw_pose_handler (rather than add_pose_handler) to be
notified of poses from this class's classifier, rather than Thalmic's onboard
processing.

Tips for classification:

- make sure to only press the number keys while the pose is being held, not
  while your hand is moving to or from the pose
- try moving your hand around a little in the pose while recording data to give
  the program a more flexible idea of what the pose is
- the rest pose needs to be trained as a pose in itself


# Caveats/issues

- on Windows, the readings become more and more delayed as time goes on
- doesn't have access to Thalmic's pose recognition (for firmware < v1.0)
- may or may not work with a Myo that has never been plugged in and set up with
  Myo Connect
- classify_myo.py segfaults on exit under certain circumstances (probably
  related to Pygame version)
