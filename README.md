# Long Range (LoRa) Radio Sensors for AKUINO
After connecting wired 17 sensors to a single AKUINO, we were convinced that wireless is very useful...

With the advent of LoRA transmission technology which ensures a really good indoor range, with the availability of ready to use modules like AdaFruit Feather M0 able to transmit data for years with simple batteries, we are convinced that a very performant solution can be created.
## Hardware

Transmitters (working on batteries, collecting and transmitting sensors data, sleeping most of the time) are explained further here: https://github.com/AKUINO/RadioSensor/tree/master/transmitter

Receiver(s) (powered, collecting transmitted data, archiving it locally, transmitting on Internet, never sleeping) is explained further here: https://github.com/AKUINO/RadioSensor/tree/master/receiver

General design considerations:

* Transmitters: Arduino MKR WAN 1300 https://store.arduino.cc/mkr-wan-1300 (was AdaFruit Feather M0: https://www.adafruit.com/product/3179 but AdaFruit products are rarely certified "CE")
* Receiver: for transmitters, we are also switching to an integrated "CE" marked product, M5STACK Code Module: https://m5stack.com/collections/m5-core
* 1-Wire sensors have the advantage of being self-identified, "hot-pluggable" and could be connected to one sensor node or another: https://www.maximintegrated.com/en/app-notes/index.mvp/id/4206
  * Serial adapter for 1-Wire gives better reliability than bit-banging (e.g. https://www.mikroe.com/uart-1-wire-click ) but requires 5V. As most 1-Wire devices are happy (but sometimes slower) at 3.3V, fewer devices not too far away will make bit-banging possible.
  * Thermocron (IButton: https://be.farnell.com/fr-BE/c/semiconducteurs-circuits-integres/memoires/accessoires-iboutons/iboutons ) can receive a "mission" and record temperature OFF-LINE for very long period of time: they can be used to ensure cold-chain continuity during transportation. They can even be programmed by a remote radio module: http://arduinofun.blogspot.com/2010/04/arduino-ibutton-data-logger-programmer.html . We have programmed data download and missioning of Thermocron by ELSA
  * Mains Switch: https://www.axiris.eu/en/index.php/1-wire/1-wire-mains-switch
* I2C sensors are often working at 3.3V and 5V. Short wires though (20 cm max: should be in the same box than the radio module)
  * I2C CO2+temperature+humidity: https://befr.rs-online.com/web/p/products/1720552/?grossPrice=Y&cm_mmc=BE-PLA-DS3A-_-google-_-PLA_BE_NL_Semiconductors-_-Sensor_Ics|Temperature_And_Humidity_Sensors-_-PRODUCT_GROUP&matchtype=&pla-544508151584&gclid=Cj0KCQiAzKnjBRDPARIsAKxfTRA_NGOGs_dE9fBuCA61ZyCIf1GovlkXcqFPyUBE3JYgCDP7rXKEHxUaAuaREALw_wcB&gclsrc=aw.ds
  * I2C Temperature and/or humidity: https://befr.rs-online.com/web/c/semiconductors/sensor-ics/temperature-sensors-humidity-sensors/?searchTerm=i2c%20sensor&applied-dimensions=4293448979,4294510394,4294490065,4294505396,4294510395,4294510203,4294356324,4294821500 and also https://www.tindie.com/products/akdracom/temperature-humidity-sensor-probe-precision-ic/
  * Grove Wiring standard is often used and is similar to QWIIC from Sparkfun: https://www.sparkfun.com/qwiic (Grove as a normal pitch connector, Sparkfun is smaller; Beware: AdaFruit standard is different!):
  1. Yellow - SCL
  1. White - SDA (Blue for QWIIC)
  1. Red - VCC on all Grove Connectors
  1. Black - GND on all Grove Connectors
  M5Stack Core is equipped with a Grove I2C connector.
* Lithium Battery: https://shop.mchobby.be/accu-et-regulateur/746-accu-lipo-37v-4400mah-3232100007468.html , https://be.farnell.com/fr-BE/mikroelektronika/mikroe-1120/batterie-lithium-polymer-3-7v/dp/2786900 , https://www.onlylipo.com/21-1s-37-volts-1-element
* Enclosures:
  * We selected KRADEX 64mm x 88mm x 42mm: https://www.tme.eu/fr/details/z-56jh-abs/boitiers-universels/kradex/z56jh-abs/ looking like CEL-MAR 1-Wire boxes that we also use.
  * 8 x 8cm, 5cm deep: many electrical junction boxes
  * 10 x 10cm, 6cm deep: http://be.farnell.com/fr-BE/fibox/pcm-95-60-g/coffret-boite-polycarbonate-gris/dp/2473443

## Software
* RadioHead libraries: http://www.airspayce.com/mikem/arduino/RadioHead/index.html Be sure to use the latest version (1.89)
  * for LoRa: http://www.airspayce.com/mikem/arduino/RadioHead/classRH__RF95.html

* 1-wire:
  * Host driver: https://www.maximintegrated.com/en/products/ibutton/software/1wire/wirekit.cfm
  * Recommended by Adafruit: https://www.pjrc.com/teensy/td_libs_OneWire.html
  * Slave device emulation library: https://github.com/orgua/OneWireHub (in case we want to connect to host through 1-wire)

* AdaFruit Feather M0, Arduino IDE SAMD support: https://learn.adafruit.com/adafruit-feather-m0-basic-proto/using-with-arduino-ide

## LoRa
* Allowed duty cycle and power in different 868MHz bands: https://www.disk91.com/2017/technology/internet-of-things-technology/all-what-you-need-to-know-about-regulation-on-rf-868mhz-for-lpwan/ From this document, the best local bandwidth could be 869,3 to 869,4 limited to 10mW but with no duty cycle constraint.
  * We measured 7 packets of 37 characters per seconds when testing this radio band.  
* 433 MHz may not be allowed in production within Europe: the project may have to be announced as "RFID" if we stick to those frequencies
  * In this band, we reached 540 meters in a corn field (emmiter and receiver at the corn height; single wire antenna)
* Calculator of Allowed duty cycle for different LoRa configurations: https://docs.google.com/spreadsheets/d/1COg2J1at6j8meehTtvfj5TIiM8pbj4SvKf7sa-Oq5fU/edit#gid=1628588258
## Sensors
Each node can receive many kind of sensors and its firmware can be adapted for each situation: the USB programming port of the node can be used to either change the firmware or to access a terminal console and change parameters.

An important information is battery voltage: its value will always be read and transmitted (register 1).

### I2C
I2C runs on 5V but also on 3.3V (not all sensors but most). Wires must be kept very short (20cm max, less is better).

The commands are different for each type of sensors: the bus cannot be really "auto discovered" like with 1-Wire (see below). The only possible thing is to associate I2C addresses with possible devices...

### 1-Wire
1-Wire ensures auto-configuration (devices can be "discovered" on the bus). Each 1-Wire device have a unique 64 bits address which can be auto-discovered by polling the 1-Wire bus (owdir on Linux, the equivalent will have to be programmed using libraries like https://www.pjrc.com/teensy/td_libs_OneWire.html ). The 8 first bits of this address indicates a generic device type which will have to be known by the firmware to adapt itself to the characteristics of each device type (e.g. number of values, polling protocol, parasitic powering...). The 8 last bits are a CRC8 checksum of the 7 previous bytes. It must be verified after each transmission (wire or radio) to ensure data integrity.

List of existing device types: http://owfs.org/index.php?page=family-code-list

The device types that we really need to support at first are:
* 26 for the DS2438 Battery Monitor (used as an ADC for various types of sensors like RH, CO2...)
* 28 for the DS18B20 Temperature Sensor; a library directly supporting this device seems to exist: https://github.com/milesburton/Arduino-Temperature-Control-Library This may be the best way to start supporting some sensors...
* 01 for the DS1990 simulated by RFID badge reader

A central directory of 1-Wire devices will be kept locally but also in the Receiver (and in ELSA application).

Every minutes or 5 (or any delay set in configuration), the node will transmit the raw data (no scaling) read for the different sensors. The allocation table will be transmitted every hour (or delay set in configuration) or on request by the central node (after detection of a checksum mismatch with the allocation table).

Data will be processed by the sensor node to transmit only specific JSON data values like "temperature" and not all registers of the device.

## Messages
* Encryption: http://www.airspayce.com/mikem/arduino/RadioHead/classRHEncryptedDriver.html#details
* Header with:
  * "FROM" (for sensors, the id of current node; for central node, id to distinguish the right central node)
	  * encoded on one byte
  * "TO" (for sensors, id of their central node; for central node, 255 for broadcast or id of a specific node to be queried)
	  * encoded on one byte
  * "ID", for a given "FROM"-"TO"-"FLAGS" combination, is a sequence ID of the message (overflow at 255): it allows the receiver to detect it has missed messages (and take action to correct this situation if needed).
  * "FLAGS" specifies the type of message: sensors data vs actuators (registers) setting; retransmission request (messages missed by receiver, the receiver signals which ID are missing); devices table to identify connected 1-Wire or I2C devices (from sensor node to central); relaying (MESH protocol, to be designed)
  * Sensors Data message: see below

## Sensors Data Messages
There are many encoding formats (see bottom of https://github.com/AKUINO/JBCDIC/blob/master/README.md ).

Please look at our proposal, the project AKUINO/JBCDIC: https://github.com/AKUINO/JBCDIC !

## Protocol
* A potential source of inspiration: https://github.com/fredilarsen/ModuleInterface/blob/master/documentation/Protocol.md
* A remote module (sensors) is sleeping most of the time (does not listen to radio) and awakens from time to time (around 3 minutes) to collect data and send it to the central module.
* A central module is listening all the time and sends actuators settings (or retransmission requests) only just after receiving sensors data from a remote module.
* The sender listen to the air and wait for silence (random delay) to avoid collision (this is done by the radio chip SX127x but needs to be configured. Looking at RadioHead library source code, it can be done: https://github.com/adafruit/RadioHead/blob/master/RH_RF95.h ). This is called LBT (Listen Before Talk).
* The message is sent using RadioHead library (encrypted but no ACK required).
* For remote module, the sender keeps listening for eventual input messages from the destination (actuators settings, sensors data retransmission requests) during a short period of time.
* If a remote module notices that some actuators settings are missing, it requests their retransmission.
* Sensor nodes send their 1-Wire / I2C Allocation table either periodically (about an hour), either when some connection changes is detected or whenever a request for it is received from the central node
* Retransmission request: timestamp + pairs of bytes for each ID intervals ("begin ID"-"end ID") that needs to be (re)transmitted. Overflow may occur (increment of 255 is 0) and "end ID" can be lower than "begin ID".
* Retransmission: message with exactly the same Header (TO, FROM, ID, FLAGS) and Payload than the one transmitted before

All nodes will be responsible to keep a buffer of the last messages (254 last messages for sensors and 254 last messages for actuators) they sent.

Redundancy of central module is possible: the two modules then share the same ID. A central module may not have received a package and therefore ask to retransmit a message well received by the other module: redundant transmissions are therefore discarded based on ID+timestamp.

## Configuration
We will use a serial terminal (USB) to set the configuration parameters (ANSI character codes for colors). We have the necessary code in C++ for PIC32. The configuration will have to assign a key (0 to 31) to the physically connected ports:
* Digital inputs: for a 1st version, we will statically map some GPIO as 0/1 ports
* Analog inputs (12 bits): for a 1st version, we will statically map some ADC inputs including at least the battery level and 2 ADC lines
* Digital outputs (PWM, pull-up, delays) to be SET (based on messages received from central): for a 1st version, we will statically map some PWM lines with basic local configuration attributes.
* 1-Wire (multiple sensors: 1-Wire addresses AUTOMATICALLY mapped to different keys but a selection of registries may have to be done based on the device type. ABANDONED for now due to lack of 5V...)
* Serial link (not for a 1st version except if a precise need arise)
* Campbell SDI (not for a 1st version except if a precise need arise)
* I2C device initialization command and register read: this is not well standardized but we will try to orthogonalize the possibilities.
A basic polling cycle (e.g. 3 minutes) has to be set. Each key can be obtained either at each poll or at a configured multiple.

## References
* The Ten Commandments of Wireless Communications: http://www.bb-elec.com/Learning-Center/All-White-Papers/Wireless-Cellular/10-Commandments-of-Wireless-Communications/10_Commandments_Wireless_WP033_R2-1112.pdf
* Detailed RadioHead interface: https://github.com/adafruit/RadioHead/blob/master/RH_RF95.h
* The standard pinout of Feather M0 is: https://cdn-learn.adafruit.com/assets/assets/000/046/244/original/adafruit_products_Feather_M0_Basic_Proto_v2.2-1.png?1504885373
* Similar project but for very long range: http://cpham.perso.univ-pau.fr/LORA/WAZIUP/FAQ.pdf
  * http://cpham.perso.univ-pau.fr/LORA/WAZIUP/SUTS-demo-slides.pdf
* CayenneLPP: https://mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload
* PJON, an alternative transmission protocol: https://github.com/fredilarsen/ModuleInterface/blob/master/documentation/Protocol.md
* PlugAndPlay mechanism for small IoT networks using "off the shelf" sensors: http://pitkaaho.net/varasto/2013_IJSNet.pdf
* MainFlux does not seems to be locking anything (to double check!) and is involved in FIWARE and is very near of a good gateway for ELSA: https://www.mainflux.com/index.html
* LORD-MicroStrain is locking by the hardware : https://github.com/LORD-MicroStrain/
* UbiWorX is locking by the platform: https://ubiworx.com/documentation/introduction/introduction.html
* W3C API: https://w3c.github.io/sensors/
