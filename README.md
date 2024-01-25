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

The input AC is fused, recitfied and fed into a 3.3V stepdown converter.
![alt text](https://i.imgur.com/zW67Loc.png)
