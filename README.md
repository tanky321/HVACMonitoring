# HVACMonitoring
## Know what you're doing
This project involves connected to the low-voltage AC portion of your furnace. While this is fairly straight forward, hire someone competent if you don't know what your doing.

## Background
This is an update/upgrade to my original HVAC monitoring project located here: https://github.com/tanky321/HVAC_Monitor
While that project was succesful, it had a couple of drawbacks I wanted to resolve.
* Power derived from furnace. The original design was powered through the regulator on the Adafruit Huzzah32 via an external USB connection.
* The H20 detection circuit was not functional in its existing state.
* Adafruit Huzzah32 is a bit pricey. With the ability to have boards made by JLC, it made sense to go with an ESP32 Module
* Size. The old design was rather large, shrinking it down would be ideal.

## System Architecture
### Power
System power is derived from the AC connections to the furnace. These connections are made via the 5 position TE header on the board.
Pinout for the connector is as follows:
1. 24VAC    (Typicaly labeled R)
2. COMMON   (Typicaly labeled C)
3. HEAT     (Typicaly labeled W)
4. COOL     (Typicaly labeled Y)
5. FAN      (Typicaly labeled G)
   
These connections are labeled on the PCB.

The input AC is fused, rectified and fed into a 3.3V stepdown converter. A TVS diode is also included for input protection.
![alt text](https://i.imgur.com/zW67Loc.png)

### Furnace State Sensors
Three opto-isolated inputs are provided to read the status of the three possible furnace states, Heat, Cool and Fan. Note that these signals are outputs from your thermostat, and inputs to your furnace. Reading of these signals is accomplished by the use of three FOD814AS optocouplers. These optocouplers contain two LEDs connected in anti-parallel, which is useful for the AC signals we're dealing with here. The transistor (emitter) output is connected to GND and the collector is connected to the ESP32 module via a pullup. Since we're dealing with an AC source, the output of the optocoupler will have brief periods where it turns off during the zero crossing portion of the AC waveform. A 10uF capacitor is used at the collector in order to smooth the ripple of the zero-crossing shutoff.
![alt text](https://i.imgur.com/vmi3Lcr.png)

### Temperature Measurement
Temperature measurement is accomplished by three MAX6675ISA+T ICs. These ICs can only measure K-Type thermocouples (which are the most common). I have typically used the three channels to measure input (return air), output (supply air) and ambient temperature. Measuring the differential temperature across the furnace + AC coil, provides a good indication of how well the furnace and AC system are working. Ambient temperature isnt really critical, just a nice data point.
![alt text](https://i.imgur.com/eu5xeX9.png)

