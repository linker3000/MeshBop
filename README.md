# REDTastic
A simple messaging service for Meshtastic, written as a set of Node-RED flows.

*The accuracy and results of using this app depend on your setup work and third party code.
This app is supplied 'as is'. Use at your own risk. No responsibility is accepted for 
consequential loss or errors. Feedback is welcome at the address below or via the Issues tab.*

```
Author:	Linker3000 (linker3000@gmail-dot-thingy)
License: Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International CC BY-NC-SA 4.0)
 
You are free to:
  - Share — copy and redistribute the material in any medium or format
  - Adapt — remix, transform, and build upon the material
 
 Under the following terms:
  - Attribution — You must give appropriate credit.
  - NonCommercial — You may not use the material for commercial purposes.
  - ShareAlike — If you remix, transform, or build upon the material, you must distribute your contributions under the same license.
 
Full License Text: https://creativecommons.org/licenses/by-nc-sa/4.0/
```
This package installs a set of muti-tab flows that comprise a trigger/response messaging service for Meshtastic using the Meshtastic TEXTMSG method of message sending and receiving. There is also a dashboard page for sending and receiving messages.

**Weather flow**
<div align="center">
<img src="images/weather-time.png" alt="Dashboard" width="800">
</div>

**Dashboard flow and dashboard**
<div align="center">
<img src="images/dashboard-flow.png" alt="Dashboard flow" width="800">
</div>
<div align="center">
<img src="images/dashboard.png" alt="Dashboard page" width="600">
</div>

The default setup includes:

+ A send / receive messaging dashboard.
+ A BBC RSS news feed.
+ In inspirational (Zen) message.
+ Local weather.
+ Local time.
+ Severe weather warnings.
+ A help page.
+ An app info page.

Setting up a messaging system requires a Meshtastic node device to which there’s access to the serial (async) Tx and Rx pins, and a compute device with a serial port  (UART) that can run Linux and Node-RED.

**Important: Meshtastic firmware (currently) sends and receives TEXTMSG messages to / from the first (primary) Meshtastic channel, so if you want to use this application in a private group, that group MUST be set as the FIRST channel on the Meshtastic node hooked up for messaging. Further notes below.**

The setup used to develop this app comprises a Seeed studios Xiao ESP32 + SX1262 Meshtastiuc kit and a Raspberry Pi Zero 2W. This makes for an easy build because there is a script for the Pi that sets up most of Node-RED. This documentation is based on that setup.

<div align="center">
<img src="images/devboard-sm.jpg" alt="Devboard" width="640">
</div>

## How to Set up
The main setup steps when using a Raspberry Pi are:

1. Put an OS an SD card for the Raspberry pi (minimal OS install is required – no graphical interface).
2. Enable the async (UART) port on the Pi. Further notes below.
3. Run the official automated script to install Node-RED. When asked whether to install any Pi-specific nodes or functionality, reply Yes. Further notes below.
4. Install the needed Node-RED plugins – there’s a list below.
5. Enable TEXTMSG mode on your Meshtastic Node. (Note: You need to have physical access to the serial / async port pins on your node.) and make a note of its comms parameters, especially the speed, which is probably 19200 bits/second.
6. With the Pi and the Node shut down and powered off, connect a wire between the 0V/GND pins on the Pi and the Meshtastic node.
7. With everything still powered off, connect the Pi Rx and Tx async pins to the Node’s async pins – remembering that the wiring needs to be crossed over so, Rx on one device goes to Tx on the other and vice versa.
8. Download and import the flow code into Node-RED, which sets up all the tabs etc.
9. Check/setup the Async in and Async out nodes – further notes below and see the notes in the setup function block on the first Node-RED tab. Leave the Async node out disabled until all other config is done and tested.
10. Run through all the notes and settings in the setup function block at the top of the first tab in Node-RED*.
11. Run through the tabs in sequence, checking for a comments node, and checking all function nodes for additional setup help and info*.
12. Customise any other messages (either in the setup function or in separate function blocks) as needed.
13. Test all functions using the debug triggers.
14. Enable the Async out node and test again in a private channel. Further notes below.
15. Test and test again.
16. 'Go live'.

Most of the hard work is done inside Node-RED. As much of the setup as possible is done in a function node on the first tab and there are comprehensive notes in many of the function nodes – check them out to complete any setup before going live. Some tabs also have additional comments.

*The default setup uses a trigger of ‘l3k_’ for received messages so, for example, a Zen quote is sent out if the message /l3kz is received. You can change this by altering the *basePattern* global variable in the global variable function and adjusting the help test message on the Help tab.
If basePattern is not changed, or it’s left unset, the parser will use /l3k by default.

Some of the flows have rate limiting nodes – please respect the Mesh and leave these set to reasonable values to avoid letting people flood the channel with, for example, Zen quotes or ping requests. 

If you are testing your setup in a private channel, or with the async Tx node disabled, there is an inject node on the first tab to reset the rate limiters. Try this if you suddenly find that outbound messages are not being generated as expected.

By default: 
+ REDTastic sends an ident message every 24 hours.
+ Severe weather alerts are checked for and issued every hour.

**NB: There is currently (Sep 2025) a bug in the Meshtastic TEXTMSG code which means that a small string of garbage data from an uncleared buffer might be sent when the node is powered on (and sometimes off) – this is a Meshtastic thing so please do not report it as a REDTastic bug.**

## Other Setup Notes
### Device wiring
The wiring between the computer board and the Meshtastic NODE will depend on what devices are used. The development setup comprised a Raspberry Pi Zero 2W connected to a XIAO ESP32S3 & Wio-SX1262 Kit for Meshtastic & LoRa as follows:

<div align="center">
<img src="images/wiring.png" alt="Board wiring" width="800">
</div>

The pins on the Xiao board marked RX and TX weren’t used in this case because they were hooked up to a GPS module. This wiring meant that the serial setup in the Meshtastic was as follows:

<div align="center">
<img src="images/serial-config-meshtastic.png" alt="Serial config" width="400">
 <P></P>
</div>

Unless arranged differently by the setup, the Pi and Seeed boards still need their own, separate USB power supplies.

### Importing the REDTastic package into Node-RED
1. Download the REDTastic *.json package from this repo.
2. Open the Burger Menu in Node-RED (top right) and select Import.
3. Choose *select a file to import*.
4. Select the downloaded .json file.
5. Click *Import*.
6. Click Deploy.

#### Global Config Error
If you receive the message: *The workspace contains some unknown node types: global-config*, try the following:
1. Click Deploy.
2. Click Search for unknown nodes.
3. Select the global-config node – notice that it is displayed with a flashing border in a list on the right side of the screen.
4. Double-click the node with the flashing border.
5. Select Delete.
6. Select Deploy. The deploy should now work and it’s time to start configuring your flows.

### Async ports
How to enable the serial module (async port) and set TEXTMSG mode on set a meshtastic node 

https://meshtastic.org/docs/configuration/module/serial/

NB: Do NOT enable “Override Console Serial Port”.

How to enable the async port on a Raspberry Pi (needs config files changed)

https://www.raspberrypi.com/documentation/computers/configuration.html#configure-uarts

Example modified boot config file - /boot/firmware/config.txt:

Typically at the end of the file:

```
[all]
enable_uart=1
dtoverlay=uart0
dtoverlay=disable-bt
```
As might be deduced, enabling uart async requires Bluetooth to be disabled.

Note: Raspberry Pi configurations normally use UART0 for their async port, which usually (but not always) translates to ttyAMA0 in Linux-speak – hence the default setup of the Node-RED async out node:

<div align="center">
<img src="images/async_node_setup.png" alt="Async node setup" width="400">
</div>

### Installing Node-RED on a Raspberry PI
https://nodered.org/docs/getting-started/raspberrypi

### Node-RED plugins
Use the *Manage palette* menu to ensure the following plugins are installed before you import the REDTastic code. Some of these nodes will be installed by default. If you are NOT using a Raspberry Pi, you should identify whether any different plugins are needed to access the serial port on your device. Not all of these plugins may be being used in the current app.
NB: Make sure you install the Flowfuse dashboard (dashboard 2):

    • flowfuse/node-red-dashboard
    • node-red-contrib-astrodata
    • node-red-contrib-buffer-parser
    • node-red-contrib-feedparser-simple
    • node-red-node-email
    • node-red-node-ping
    • node-red-node-serialport
### Testing / Using in a Private Meshtastic Channel
**Reminder: Meshtastic firmware (currently) sends and receives TEXTMSG messages to / from the first (primary) Meshtastic channel, so if you want to use this application in a private group, that group MUST be set as the FIRST channel on the Meshtastic node hooked up for messaging.**

To do this is a three-step process in the Meshtastic phone app:
1. Set up a new channel as your private channel – by default, this is known as a ‘secondary’ channel, however we will move its order – see link below for how to set up a secondary (private) channel.
2. In the Radio Configuration for the node being used, drag the new private channel to be the first one in the list, then save and wait for the node to reboot. In other words the secondary (private) channel now becomes the primary channel on this node.
3. Configure another Meshtastic node to work with this private channel. Provided the config is done correctly, this can be left as a secondary channel – it does not have to be moved to be the first one in the channel list, but you must select it as the one for messaging.
Use this other Meshtastic node for testing - remembering to enable the Async out node on the first tab in Node-RED so that messages go out.

<div align="center">
<img src="images/Message-channels-annot.png" alt="Message channels" width="200">
</div>

### How to set up a Secondary (private) Channel.
https://meshtastic.org/docs/configuration/radio/channels/
Once all testing is done, there are two options:
1. Leave the channel order as it is on the messaging node and all the good things will happen in this private channel.
2. Use the Radio configuration menu to change the channel order back to the original order, which makes the Longfast (public) channel the default – which is the one to/from which all REDTastic messaging takes place.

In other words, if channel order is swapped back to the default, everything sent by REDTastic goes to the public (LongFast) messaging channel. 

***Make sure that’s what you want to do!***

REDTastic rate limits its sending so there is reduced risk of flooding the public channel, but you may not want to accidentally send any personal information to the wrong channel.
