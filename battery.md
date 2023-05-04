---
layout: default
title: Battery reading
parent: Instructions
nav_order: 1
---
<h1>Battery status reading</h1>
Reading the charging status of a battery is not as straightforward as it would seem. Victron energy provides this information from some of their MPPTs through a propietary port called VE.direct. If using an MPTT wihout port, an external battery monitor can be installed, like [Victron Smart Battery Shunt](https://www.victronenergy.com/battery-monitors/smart-battery-shunt). Then one needs a [VE.direct to USB cable](https://www.victronenergy.com/accessories/ve-direct-to-usb-interface) to connect the MPPT/shunt to the server through USB.

Once the setup is working the data can be read using serial. The port should be updated to reflect where the cable is connected.
```py
import serial
sio = serial.Serial(port='/dev/ttyUSB0',baudrate=19200,parity=serial.PARITY_NONE,stopbits=serial.STOPBITS_ONE,bytesize=serial.EIGHTBITS,timeout=12)
```
Detail information about the VE protocol can be found in the [VE.direct protocel whitepaper](https://www.victronenergy.com/support-and-downloads/technical-information). For our application we just use battery charge and load in watt.
First all lines are read one by one, this can be printed to identify other values that may be interesting.
```py
line = sio.readline().decode('iso-8859-1')
```
The the lines for power in Watt and percentage of battery are identified and the value extracted from the string.
```py
if (line.find("P	") != -1):
  kw = float(line[2:])
if (line.find("SOC") != -1):
  battery = float(line[4:])/10
```
The whole code can be found below and at [github](https://github.com/zapico/SolarServer). It saves the data in a json file for the website to read, but the data can be saved in a database, sent using MQTT or any other option. It reads the data every 10 minutes, this can be changed in the time.sleep() line.
```py
import serial
from datetime import datetime
import time
import json
import os

# Connect to serial
sio = serial.Serial(port='/dev/ttyUSB0',baudrate=19200,parity=serial.PARITY_NONE,stopbits=serial.STOPBITS_ONE,bytesize=serial.EIGHTBITS,timeout=12)
print("connected to: " + sio.portstr)
# Init variables
count=1
kw = 0
battery = 0

# Loop
while True:
    # Check that serial has read both values
    if (kw!=0 and battery!=0) or (battery==100):
        dictionary = {
           "kw": kw,
           "battery": battery,
           "time": str(datetime.now())
        }
        # Save as status json and to history
        with open("/var/www/thehackfarm/html/battery.json", "w") as outfile:
            json.dump(dictionary, outfile)
        # Append to history.json. This dictionary can then be read with:
        # with open('my_file') as f:
        #   my_list = [json.loads(line) for line in f]
        with open('/var/www/thehackfarm/html/history.json', 'a') as f:
            json.dump(dictionary, f)
            f.write(os.linesep)
        # Reset numbers and sleep for 10 minutes
        kw = 0
        battery = 0
        time.sleep(600)

    line = sio.readline().decode('iso-8859-1')
    # read kwh in
    if (-1 != line.find("P	")):
        kw = float(line[2:])
        print(kw)
    # read battery
    if (-1 != line.find("SOC")):
       battery = float(line[4:])/10
       print(battery)

```
