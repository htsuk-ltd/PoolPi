# Sensors

## Temperature Sensors
The system has a total of 5 DS18B20 temperature sensors for monitoring the general environment and water temperatures.

**1.) Ambient Air Temperature**
This is located behind the Spa in shade. It gives an indiation of the current outside temperature. It is located in shade to limit the effect of direct sunlight.

**2.) Pool Temperature**
Whilst the Lay-Z-Spa provided pump/heater has a temperature readout, I wanted to be able to log the temperature over time and with a greater precison than that provided by bestway. It is also important to be able to have Splunk read the current pool temperature to make decisions about what mode to run in.

**3.) Solar Array Temperature**
The Solar Array has a sensor inside the enclosure which provides a reading of the effect of the sun on the coil. This sensor is used to determin if there is any appricable effect from solar radiation and if there is benefit from activating the solar system.

**4.) Solar Inlet Temperature**
The inlet temperature is often close to the value of the pool temperature, however as the actual temperature can fluctuate in the pool with depth the inlet temperature tends to reflect the lowest temperature value as the water is drawn up from the very bottom of the pool where typically coldest.

**5.) Solar Outlet Temperature**
This provides the temperature of the water returned from the Solar coil. On a sunny day this can be many degrees higher than the inlet temperature, however this sensor is key in ensuring the solar system turns off when there is no longer any value in running the solar system.

## Flow Sensor
WIP

## DS18B20 Wiring

## Flow Sensor Wiring

## Python Script

```
import os
import glob
import time

from datetime import datetime
# Initialize the GPIO Pins

os.system('modprobe w1-gpio')  # Turns on the GPIO module
os.system('modprobe w1-therm') # Turns on the Temperature module

# A function that reads the sensors data
def read_temp_raw(device):
  f = open(device, 'r') # Opens the temperature device file
  lines = f.readlines() # Returns the text
  f.close()
  return lines

# Convert the value of the sensor into a temperature
def read_temp(device):
  lines = read_temp_raw(device) # Read the temperature 'device file'

  # While the first line does not contain 'YES', wait for 0.2s
  # and then read the device file again.
  while lines[0].strip()[-3:] != 'YES':
    time.sleep(0.2)
    lines = read_temp_raw(device)

  # Look for the position of the '=' in the second line of the
  # device file.
  equals_pos = lines[1].find('t=')

  # If the '=' is found, convert the rest of the line after the
  # '=' into degrees Celsius, then degrees Fahrenheit
  if equals_pos != -1:
    temp_string = lines[1][equals_pos+2:]
    temp_c = float(temp_string) / 1000.0
    temp_f = temp_c * 9.0 / 5.0 + 32.0
    return temp_c

ambient=read_temp('/sys/bus/w1/devices/28-03167183deff/w1_slave')
pool=read_temp('/sys/bus/w1/devices/28-0316713730ff/w1_slave')
inlet=read_temp('/sys/bus/w1/devices/28-03167124f2ff/w1_slave')
outlet=read_temp('/sys/bus/w1/devices/28-0416718d83ff/w1_slave')
array=read_temp('/sys/bus/w1/devices/28-0316717bb1ff/w1_slave')

timestamp=datetime.now()
print str(timestamp) + ' ambient='+ str(ambient) + ' pool='+str(pool)+' inlet='+str(inlet)+' outlet='+str(outlet)+' array='+str(array)
    
```

The resulting output from an execution of this script is thus:

```
2020-05-10 17:23:00.885254 ambient=12.562 pool=29.937 inlet=30.25 outlet=29.625 array=17.125
```
**It's horrible outside! :(**
