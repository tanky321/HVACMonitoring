# HVACMonitoring
## Know what you're doing
This project involves connections to the low-voltage AC portion of your furnace. While this is fairly straight forward, hire someone competent if you don't know what your doing.

## Background
This is an update/upgrade to my original HVAC monitoring project located here: https://github.com/tanky321/HVAC_Monitor
While that project was successful, it had a couple of drawbacks I wanted to resolve.
* Power derived from furnace. The original design was powered through the regulator on the Adafruit Huzzah32 via an external USB connection.
* The H20 detection circuit was not functional in its existing state.
* Adafruit Huzzah32 is a bit pricey. With the ability to have boards made by JLC, it made sense to go with an ESP32 Module
* Size. The old design was rather large, shrinking it down would be ideal.

## Setup
### Connections
![alt text](https://i.imgur.com/OJXgdR1.png)

### ESPHome
Each assembled board is pre-flashed with the firmware in this repository when shipped. You'll need to configure the WiFi and get it hooked into Home Assistant. Setup is simple!

Once the board is powered up, head over to [Improv-Wifi](https://www.improv-wifi.com/), using a device that has BLE capability.

Click on "Connect Device to WiFi"

![alt text](https://i.imgur.com/Wjgqj0u.png)

Select hvac-monitor from the list.

![alt text](https://i.imgur.com/UT5waBG.png)

After a moment, the system will prompt you for a network SSID and password. The ESP will provision and should eventually connect to your network.

![alt text](https://i.imgur.com/MuuLvju.png)

If all goes well, you should see the following

![alt text](https://i.imgur.com/4TMDPDZ.png)

Head back over to your ESPHome instance,and you should see that the device has been discovered, and is ready to adopt.

![alt text](https://i.imgur.com/GCHEvlh.png)

Click adopt, and enter a name of your choosing for the device.

![alt text](https://i.imgur.com/Dh5V0go.png)

You will be prompted for the API encryption key, click install to download the most recent yaml configuration and install it to the board.

![alt text](https://i.imgur.com/EqdbTR6.png)

Once the upload is complete, head over to HomeAssistant (Settings -> Devices & Services) and you should see the device waiting to be added. :)

![alt text](https://i.imgur.com/AinF8mz.png)

## System Architecture
### Power
System power is derived from the AC connections to the furnace. These connections are made via the 5 position TE header on the board.
Pinout for the connector is as follows:
1. 24VAC    (Typically labeled R)
2. COMMON   (Typically labeled C)
3. HEAT     (Typically labeled W)
4. COOL     (Typically labeled Y)
5. FAN      (Typically labeled G)
   
These connections are labeled on the PCB.

The input AC is fused, rectified and fed into a 3.3V stepdown converter. A TVS diode is also included for input protection.
![alt text](https://i.imgur.com/zW67Loc.png)

### Furnace State Sensors
Three opto-isolated inputs are provided to read the status of the three possible furnace states, Heat, Cool and Fan. Note that these signals are outputs from your thermostat, and inputs to your furnace. Reading of these signals is accomplished by the use of three FOD814AS optocouplers. These optocouplers contain two LEDs connected in anti-parallel, which is useful for the AC signals we're dealing with here. The transistor (emitter) output is connected to GND and the collector is connected to the ESP32 module via a pullup. Since we're dealing with an AC source, the output of the optocoupler will have brief periods where it turns off during the zero crossing portion of the AC waveform. A 10uF capacitor is used at the collector in order to smooth the ripple of the zero-crossing shutoff.
![alt text](https://i.imgur.com/vmi3Lcr.png)

### Temperature Measurement
Temperature measurement is accomplished by three MAX6675ISA+T ICs. These ICs can only measure K-Type thermocouples (which are the most common). These sensors operate using SPI. I have typically used the three channels to measure input (return air), output (supply air) and ambient temperature. Measuring the differential temperature across the furnace + AC coil, provides a good indication of how well the furnace and AC system are working. Ambient temperature isnt really critical, just a nice data point.
![alt text](https://i.imgur.com/eu5xeX9.png)

### Pressure Measurement
A Sensiron SDP810-500PA sensor is used for measurement of differential pressure. This sensor uses I2C to communicate. It is capable of measuring ±2.0 inH2O (±500pA) which should be more than adequate for just about all HVAC installations. I have used magnetic static pressure probes for mechanical interface to the system. [Amazon link here](https://www.amazon.com/Dwyer-Portable-Static-Plastic-Insertion/dp/B008HOWU6I/ref=sr_1_5?crid=2RSDY7RSZ3YQS&keywords=static%2Bpressure%2Bprobe&qid=1706146716&sprefix=static%2Bpressure%2Bprobe%2B%2Caps%2C130&sr=8-5&th=1)
![alt text](https://i.imgur.com/cBenJD0.png)

### Water Detection
Two simple water detection circuits are included for detection of leaks or drain overflow. For use, connect two wires to a channels connector (J5 or J6) and place them near each other (but not touching). If water completes the circuits between the two wires, the output to the ESP32 will drive LOW, indicating a water leak detected.
![alt text](https://i.imgur.com/kMcRsEr.png)

## Control
Main system control/interface is done with an ESP32-C3 module. A Status LED is included and connected to GPIO3.

USB connectivity is provided for upload of firmware.

![alt text](https://i.imgur.com/T55x2s4.png)
