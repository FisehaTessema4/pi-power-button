
# Final Project CNE 350 

# pi-power-button

# Why is a Raspberry Pi power button important?
You should never "yank" the power cord out of your Pi as this can lead to severe data corruption (and in some cases, physically damage your SD card). You can safely shut down your Pi via a software command or, even better, use a power button or switch

Scripts used in our official [Raspberry Pi power button guide](https://howchoo.com/g/mwnlytk3zmm/how-to-add-a-power-button-to-your-raspberry-pi).

## Installation

# Understanding the wake functionality
There's nothing to build here, but we need to understand how to wake up the Pi from a halt state before we build the shutdown functionality. Simply put, shorting pins 5 and 6 (GPIO3 and GND) together will wake the Pi up from a halt state.

An easy way to test this is to shutdown the Pi with sudo shutdown -h now, and connect pins 5 and 6 with a female to female cable. You only need to short them momentarily. Then you should find that the Pi is "awake".
# Building the sleep functionality
There are two options for building the sleep functionality: using our install script or installing everything manually. I recommend using the install script, but the manual approach will help you understand how this works.

Option 1: Use the install script (easiest)
The simplest way to install the required scripts is to clone our power button repository, and run the install script.

SSH into your Pi, install git (if it's not already), and then run:
```
git clone https://github.com/Howchoo/pi-power-button.git

./pi-power-button/script/install
```
First, we need to connect to the Pi via SSH. Then, we'll use a script called listen-for-shutdown.py.
```
sudo nano listen-for-shutdown.py
```
Then, paste the following code into that file, and press CTRL-X to exit, and Y to save when prompted.
```
#!/usr/bin/env python

import RPi.GPIO as GPIO
import subprocess


GPIO.setmode(GPIO.BCM)
GPIO.setup(3, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.wait_for_edge(3, GPIO.FALLING)

subprocess.call(['shutdown', '-h', 'now'], shell=False)
```
Next we need to start this script on boot. So we'll place the script in /usr/local/bin and make it executable:
```
sudo mv listen-for-shutdown.py /usr/local/bin/
sudo chmod +x /usr/local/bin/listen-for-shutdown.py
```

Now add another script called listen-for-shutdown.sh that will start/stop our service. To create the script:
```
sudo nano listen-for-shutdown.sh
```
Enter the following code in that file and save it:
```
#! /bin/sh

### BEGIN INIT INFO
# Provides:          listen-for-shutdown.py
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
### END INIT INFO

# If you want a command to always run, put it here

# Carry out specific functions when asked to by the system
case "$1" in
  start)
    echo "Starting listen-for-shutdown.py"
    /usr/local/bin/listen-for-shutdown.py &
    ;;
  stop)
    echo "Stopping listen-for-shutdown.py"
    pkill -f /usr/local/bin/listen-for-shutdown.py
    ;;
  *)
    echo "Usage: /etc/init.d/listen-for-shutdown.sh {start|stop}"
    exit 1
    ;;
esac

exit 0
```
Place this file in /etc/init.d and make it executable.
```
sudo mv listen-for-shutdown.sh /etc/init.d/
sudo chmod +x /etc/init.d/listen-for-shutdown.sh
```
Now we'll register the script to run on boot.
```
sudo update-rc.d listen-for-shutdown.sh defaults
```
Since the script won't be running, we'll go ahead and start it with:
```
sudo /etc/init.d/listen-for-shutdown.sh start
```


1. [Connect to your Raspberry Pi via SSH](https://howchoo.com/g/mgi3mdnlnjq/how-to-log-in-to-a-raspberry-pi-via-ssh)
1. Clone this repo: `git clone https://github.com/Howchoo/pi-power-button.git`
1. Optional: Edit line 9/10 in listen-for-shutdown.py to your preferred pin (Please see "Is it possible to use another pin other than Pin 5 (GPIO 3/SCL)?" below!)
1. Run the setup script: `./pi-power-button/script/install`

## Uninstallation

If you need to uninstall the power button script in order to use GPIO3 for another project or something:

1. Run the uninstall script: `./pi-power-button/script/uninstall`

## Hardware

A full list of what you'll need can be found [here](https://howchoo.com/g/mwnlytk3zmm/how-to-add-a-power-button-to-your-raspberry-pi#parts-list). At a minimum, you'll need a normally-open (NO) power button, some jumper wires, and a soldering iron. If you _don't_ have a soldering iron or don't feel like breaking it out, you can use [this prebuilt button](https://howchoo.com/shop/product/prebuilt-raspberry-pi-power-button?utm_source=github&utm_medium=referral&utm_campaign=git-repo-readme) instead.

Connect the power button to Pin 5 (GPIO 3/SCL) and Pin 6 (GND) as shown in this diagram:

![Connection Diagram](https://raw.githubusercontent.com/Howchoo/pi-power-button/master/diagrams/pinout.png)

### Is it possible to use another pin other than Pin 5 (GPIO 3/SCL)?

Not for full functionality, no. There are two main features of the power button:

1. **Shutdown functionality:** Shut the Pi down safely when the button is pressed. The Pi now consumes zero power.
1. **Wake functionality:** Turn the Pi back on when the button is pressed again.

The **wake functionality** requires the SCL pin, Pin 5 (GPIO 3). There's simply no other pin that can "hardware" wake the Pi from a zero-power state. If you don't care about turning the Pi back _on_ using the power button, you could use a different GPIO pin for the **shutdown functionality** and still have a working shutdown button. Then, to turn the Pi back on, you'll just need to disconnect and reconnect power (or use a cord with a physical switch in it) to "wake" the Pi.

Of course, for the GND connection, you can use [any other ground pin you want](https://pinout.xyz/).
