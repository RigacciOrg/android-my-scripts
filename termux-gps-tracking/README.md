# GPS Remote Tracking with Termux:API

**Python script for remote logging of Android locations**

The Python script **termux-gps-track** runs on Android devices, 
requiring the **Termux** and **Termux:API** apps. It acquires 
GPS positions (actually Android locations) via the Termux:API at 
specified time intervals, then send collected data to a remote 
host via UDP datagrams, at other specified intervals.
    
The main loop runs for TIME_TO_LIVE seconds, collecting location
data at SAMPLE_PERIOD interval and sending it to REMOTE_HOST at
SEND_DATA_PERIOD interval.

This script was supposed to be run by the Android Job Scheduler, 
which has a minimum period of 15 minutes. So the main loop 
executes for that time, then exits. You can also run it manually 
or at bootstrap, so an endless loop is also configurable.

* Required Android apps: **Termux** and **Termux:API**, 
suggested: **Termux:Widget** and **Termux:Boot**.
* Required Termux package: **termux-api**.
* Required Android permission: **ACCESS_FINE_LOCATION** and/or
**ACCESS_COARSE_LOCATION** for Termux:API.

## Features

* Uses the 
**[termux-location](https://wiki.termux.com/wiki/Termux-location)** 
command line tool to get position coordinates; this tool is 
provided by the the **termux-api** package and relies on the 
**[Termux:API](https://wiki.termux.com/wiki/Termux:API)** app to 
communicate with the Android API. It have several options to get 
the location, to balance position accuracy with battery 
duration.
* Communicates with a remote server using **UDP datagrams**, for 
less overhead on unreliable network connections.
* If **network is not available**, keeps data into a buffer for 
**later sending**.
* When buffer grows over the user-selected limit, **unsent data 
are saved** to local storage.
* On program exit, unsent data is **stored locally and 
reloaded** on next execution.
* **Data is signed** with username and password (PSK), to avoid 
tampering.
* Data is sent in **JSON format, compressed**.

## Installation

* Copy the **termux-gps-track** script into a folder, e.g.
**/data/data/com.termux/files/home/bin/**.
* In the same directory of the script, create a 
**termux-gps-track.ini** file using the provided sample as a 
template, edit the values for **my_name**, **remote_host** and 
**remote_psk**.
* Start the script manually using a 
**[Termux:Widget](https://wiki.termux.com/wiki/Termux:Widget)** 
shortcut or periodically via the 
**[termux-job-scheduler](https://wiki.termux.com/wiki/Termux-job-scheduler)**. 
You can also run it automatically at device bootsrtap, using the 
**[Termux:Boot](https://wiki.termux.com/wiki/Termux:Boot)** app. 
For each mode, be sure to select the proper **time_to_live** 
configuration option.

The **receive-termux-data** is a proof of concpet script for a
daemon receiving the UDP datagrams. It listens on UDP port 33897
and stores location data either into a GPX file or into
a PostgreSQL database. Create a **receive-termux-data.ini** file
in the same directory (see the sample) and add username and password
used by the termux-gps-track script to verfy received data.

## Running a script using the Android Job Scheduler

When you start a script with the termux-job-scheduler, it will 
run immediately a first time. Then it will be scheduled at the 
specified interval.

A Termux instance will be shown in the recent apps list, but no 
icons will be displayed into the Android notification bar.

Beware that if you remove (clean) the Termux app from recent 
list, the running script will terminated. Eventually it will be 
restarted by the Job Scheduler at the next interval.

Next executions of the script will instead appear into the
Android notification bar, but not into the recent apps list.

A script launched by the Android Job Scheduler, will be 
suspended when the Android device falls asleep (screen off, 
etc.), unless you have acquired some wake locks. It could even 
be terminated.

When the Android device is in sleep state, the scheduled
job will be launched anyway, but it will be eventually
suspended or even terminated shortly. 
