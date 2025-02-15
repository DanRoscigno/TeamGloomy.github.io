---
title: Connecting a Fly-407ZG via Wifi
tags: []
keywords: 
last_updated: 11/03/2021
summary: "Connecting a Fly-407ZG via Wifi"
sidebar: mydoc_sidebar
permalink: fly_407zg_connected_wifi.html
folder: mydoc
comments: false
toc: false
datatable: true 
---

## Overview

The Fly-407ZG is an STM32F407ZGT6 based board.

## Firmware File

Choose the correct corresponding firmware (firmware-stm43f4-esp8266wifi.bin) from [here](https://github.com/gloomyandy/RepRapFirmware/releases). Remember to rename it to firmware.bin. Put it in the root of a FAT32 formatted SD card.

## Wifi

Use a nodemcu ESP8266 with USB programming as it already 5v tolerant and it allows for updating via USB.

### BOM

* 1 x nodemcu ESP8266 or Wemos D1 mini
* 3 x 47R resistor
* 1 x 470R resistor
* 3 x 2200R resistor
* jumpers or other ways of connecting to the Fly-407ZG

### Preparing the ESP

Follow the instructions [here](stm32_esp.html).

### Connecting the ESP

The pinout for the Fly-407ZG can be found [here](https://github.com/FLYmaker/FLYF407ZG/blob/master/picture/Pin%20diagram.png) and the schematic for the Duet 2 Wifi for reference can be found [here](https://github.com/T3P3/Duet/blob/master/Duet2/Duet2v1.04/DuetWifiv1.04a_Schematic.pdf). 

The table below shows the pins required on the ESP8266 and what they are connected to on the Fly-407ZG. Please ensure that your cables are no longer than 30cm although they should ideally be as short as possible.  

<div class="datatable-begin"></div>

| ESP Pin       | Fly-407ZG Pin       | Resistor Value  |
| :-------------: |:-------------:| :---------------:|
| RST           | PB2 on EXP2         | 470R            |
| CS/GPIO15     | PF11 on EXP2         | 2200R           |
| MOSI/GPIO13   | PB15 on EXP2         | 47R             |
| MISO/GPIO12   | PB14 on EXP2         | 47R             |
| SCLK/GPIO14  | PB13 on EXP2         | 47R             |
| ESP_DATA_Ready/GPIO0   | PE15 on EXP1         | 2200R             |
| LPC_DATA_Ready/GPIO4   | PB10 on EXP1         | None            |
| VIN(5v)   | 5v on EXP1          | None             |
| GND   | GND on EXP1          | 2200R to RST             |

<div class="datatable-end"></div>

{% include warning.html content="The cables used need to be very very short. Even 10cm ones don't work so they must be shorter than that" %}

If you would rather use a pre-made wifi board, then the one produced by [flymaker](https://www.aliexpress.com/item/1005001370540066.html) is recommended.  

### Prepare the SD Card

Follow the instructions on [Getting Started with RRF3](getting_started.html)

### Board.txt file

You will also need a board.txt file in the sys folder. Below are the contents that should be used.

```
//Config for Fly-407ZG
board = fly_f407zg
8266wifi.espDataReadyPin = E.15
8266wifi.TfrReadyPin = B.10
8266wifi.espResetPin = B.2
8266wifi.csPin = F.11
heat.tempSensePins = { F.3, A.0, C.1, C.0, F.10, F.5, F.4 }
```

### Smart Drivers

If using TMC5160 or TMC22XX drivers (where 22XX is either the TMC2208, TMC2209, TMC2225 or TMC2226), the following line must also be added to the board.txt file
```
stepper.numSmartDrivers = X
```
Where X is the number of drivers fitted in total.

#### TMC22XX UART Drivers

The drivers must be continuous and start at unit 0 (unless TMC5160 are also used, which case they must be installed after them). So, for the SKR board, if you have say 3 TMC2208s and 1 other driver, the 2208s must be in slots 0, 1, 2 and the remainiong driver in slot 3 or 4. You can use RRF to assign any of those slots to an axis/extruder. 

#### TMC5160 SPI Drivers

TMC5160 drivers are supported from 3.3 beta 1 onwards.
If using TMC5160 drivers, the following lines must also be added to the board.txt file.  
```
stepper.num5160Drivers = X
stepper.spiChannel = 2
```
Where X is the number of 5160 drivers fitted. The drivers must be continuous and start at unit 0. So, if you have say 3 TMC5160s and 1 TMC22XX and 1 other driver, the 5160s must be in slots 0, 1, and 2, the TMC22XX in slot 3 and the remainiong driver in 4. You can use RRF to assign any of those slots to an axis/extruder.  


#### Sensorless Homing

To be able to use sensorless homing on this board, a jumper cable needs to be installed between the diag pin of the driver and an endstop input.  
For more information about setting up sensorless homing, please read [this](sensorless.html).   

### Board.txt Location

Place the *board.txt* file in a directory called "sys" on the SD card and install the SD card in the Fly-407ZG.   

### Final Setup

Once connected, power up the board using 12-24v and connect to the USB port on the board. Using a program such as [termite](https://www.compuphase.com/software_termite.htm), connect to the board. As of release 3.2_4, the recommended terminal program is [YAT](https://sourceforge.net/projects/y-a-terminal/). Then type in the following

```
M552 S0
M587 S"your SSID" P"your password"
M552 S1
```

{% include warning.html content="**DO NOT USE PRONTERFACE** it will convert all text to upper case. If you really must, please do the following. <br/>  If you wanted to use “PassWord”, you would write P”P’a’s’sW’o’r’d” with the ‘ indicating the following letter should be lower case. Explanation [here](https://duet3d.dozuki.com/Wiki/Gcode#Section_M587_Add_WiFi_host_network_to_remembered_list_or_list_remembered_networks)." %}

The blue light on the wifi chip shoould then flash blue and will go solid when a connection has been established. The ip address will be shown on the serial connection. It is also possible to type just M552 to get the current ip address reported back.

The final thing to do is add the line “M552 S1” to your config file. This can be done through the web interface. This just ensures that the wifi connection is started at start up. There is no need to add the M587 command as this is written permanently to the flash of the ESP chip.