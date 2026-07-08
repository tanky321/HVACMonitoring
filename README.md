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
### NOTE! THE BOARD MUST BE POWERED THROUGH THE 24VAC AND COM CONNECTIONS, THE BOARD WILL NOT POWER UP ON USB
*To be corrected in future revisions*

![alt text](https://i.imgur.com/OJXgdR1.png)



### ESPHome
Each assembled board is pre-flashed with the firmware in this repository when shipped. You'll need to configure the WiFi and get it hooked into Home Assistant. Setup is simple!

Once the board is powered, allow a few minutes for the fallback hotspot to be created by the device. This will take about 1 minute. 
Connect to the SSID: "HVAC Monitor Fallback Hotspot"
Password is "**pressure**"

A prompt should appear to sign into the network (on Android at least, not tested on iOS). Follow that prompt. If a prompt does not appear, navigate to [http://192.168.4.1/](http://192.168.4.1/)

A screen simillar to this one will appear. Either select your SSID, or enter it manually in the box at the bottom. Enter the appropriate password and click save
![alt_text](https://i.imgur.com/Kv9ugGL.png)

You can now navigate to [hvac-monitor.local](hvac-monitor.local) (or the assigned IP if mDNS doesn't work) to open the web interface. This will display all of the sensor readings as shown below.
![alt_text](https://i.imgur.com/fGyhH5C.png)

Navigate back to your Home Assistant instance and then to your ESPHome instance. ESPHome will prompt that it discovered a device, click **show** and then click **Take control**
![alt_text](https://i.imgur.com/qlA69nn.png)

On the next window, you can modify the friendly name if needed. Once finished, click **Take control**
![alt_text](https://i.imgur.com/EzSLltm.png)

In Home Assistant go to **Settings -> Devices & services**, you should see a prompt to add the monitoring system as shown below. Click **Add**
![alt_text](https://i.imgur.com/ZqSnpKg.png)

Click **submit** on the following window
![alt_text](https://i.imgur.com/lrty0q9.png)

The **Name and assign** window will be presented, modify the Device name if required. When finished click **Skip and finish**

You will now be presented with all of the device info, and the device has been succesfully added to Home Assistant
![alt_text](https://i.imgur.com/pV59pPa.png)


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
