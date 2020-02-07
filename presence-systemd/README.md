## Introduction
This script is a variant on a theme of the [Python script](https://www.domoticz.com/wiki/Presence_detection#Linux_.28Raspberry_Pi.29) from the Domoticz Wiki which allows a virtual switch (or switches) to be updated in Domoticz based upon the output of a the Python script.

This variant uses systemd rather than cron and is (in my opinion) more flexible than the [existing](https://github.com/jorijnsmit/onlineChecker) systemd variant referenced by the Wiki
## Installation

1. If Python isn't already installed on your system, install it. The script requires Python 2 rather than the newer Python 3 a present
 >  sudo apt install python

2. Clone this repository somewhere writable (e.g. /home/pi)
 > git https://github.com/DJBenson/domoticz-things.git /home/pi/domoticz-things

3. Copy the check_device_online.py somewhere safe
 > cd /home/pi/domoticz-things/presence-systemd
 > cp check_device_online.py /home/pi/domoticz/scripts

(*the rest of the instructions assume you are still in /home/pi/domoticz-things/presence-systemd as the current directory*)

4. Edit the config file to meet your requirements. The file contains three parameters; IDX (the switch ID of the virtual switch in Domoticz), INTERVAL (how often the device should be pinged, default 10 seconds) and COOLDOWN (how long after going offline should that status be reported to Domoticz, default 120 seconds).
> nano ./etc/default/presence@example

5. Copy the config file to /etc/default (requires root priveleges). The bit after the '@' symbol is the device name/IP address so when you copy the file to its final destination, use something meaningful for this name. In this example I use 'iphone'. I recommend using your devices DNS hostname rather than its IP address so that if it gets a new IP, your address still works. On iOS, the device name in Settings > General > About > Name is the hostname. Android devices probably have a similar setting.

> sudo cp ./etc/default/presence@example /etc/default/presence@iphone

6. Edit the systemd unit file ensuring the script location is correct (change the path to whatever you set it to in step 3)

> nano ./etc/systemd/system/presence@.service

    [Unit]
    Description=Monitor presence by arping %i
    After=network.target
    
    [Service]
    Environment="LOCAL_ADDR=localhost"
    EnvironmentFile=/etc/default/presence@%i
    ExecStart=/usr/bin/python2 /home/pi/domoticz/scripts/check_device_online.py %i ${IDX} ${INTERVAL} ${COOLDOWN}
    ExecStopPost=/bin/rm /home/pi/domoticz/scripts/check_device_online.py_%i.pid
    RestartSec=5
    Restart=always
    Type=simple
    MemoryHigh=10M
    MemoryMax=15M
    TasksMax=3
    
    [Install]
    WantedBy=multi-user.target

The "MemoryHigh", "MemoryMax" and "TasksMax" were put in place to stop the script running away with resources becasue it spawns additional shell processes which seemed to be consuming increasing amounts of memory. These defaults should keep each process to about 10 megabytes of RAM.

7. Copy the systemd service file to its final destination
>sudo cp ./etc/systemd/presence@.service /etc/systemd/system

8. Start the service
>sudo systemctrl start presence@iphone.service

9. Confirm the service is running
>systemctrl status presence@iphone.service

**domoticz@raspberrypi**:~**$** systemctl status presence@iphone.service
    ● presence@jessicas-iphone.service - Monitor presence by arping iphone
    Loaded: loaded (/etc/systemd/system/presence@.service; enabled; vendor preset: enabled)
    Active: **active (running)** since Fri 2020-02-07 00:35:31 GMT; 16h ago
    Main PID: 83293 (python2)
    Tasks: 1 (limit: 3)
    Memory: 9.8M (high: 10.0M max: 15.0M)
    CGroup: /system.slice/system-presence.slice/presence@iphone.service
    └─83293 /usr/bin/python2 /home/pi/domoticz/scripts/check_device_online.py iphone 78 10 120 

10. Enable the service to start at boot
>sudo systemctrl enable presence@iphone

Repeat the instructions from step 4 onwards for each device you wish to monitor. You will only ever have one service file in /etc/systemd/system but you will have multiple config files in /etc/default which will be used by the systemd service as an alias. Say for example you want to add a new device called 'android', simply repeat from step 4 creating a new file called presence@android in /etc/default then enable the service by issuing sudo systemctrl enable presence@android.service.
