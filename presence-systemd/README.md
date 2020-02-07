## Introduction
This script is a variant on a theme of the [Python script](https://www.domoticz.com/wiki/Presence_detection#Linux_.28Raspberry_Pi.29) from the Domoticz Wiki which allows a virtual switch (or switches) to be updated in Domoticz based upon the output of a the Python script.

This variant uses systemd rather than cron and is (in my opinion) more flexible than the [existing](https://github.com/jorijnsmit/onlineChecker) systemd variant referenced by the Wiki
## Installation

1. If Python isn't already installed on your system, install it. The script requires Python 2 rather than the newer Python 3 a present
 >  sudo apt install python

2. Clone this repository somewhere writable (e.g. /home/pi)
 > TEST

3. Copy the check_device_online.py somewhere safe
 >cp check_device_online.py /home/pi/domoticz/scripts

4. Edit the config file to meet your requirements. The file contains three parameters; IDX (the switch ID of the virtual switch in Domoticz), INTERVAL (how often the device should be pinged, default 10 seconds) and COOLDOWN (how long after going offline should that status be reported to Domoticz, default 120 seconds).
> TEST

6. Edit the systemd unit file ensuring the script location is correct (change the path to whatever you set it to in step 3)
