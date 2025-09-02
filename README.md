# Beta Testers wanted � I WANT YOU!
(skip this section if not interested :-))

I'm looking for a small group of passionate beta testers to help shape a new GPIB adapter that packs serious capabilities into a tiny form factor. If you're into instrumentation, protocol-level debugging, or just love pushing hardware to its limits�this might be for you. Breaking something is highly valued by me, such that I can fix it :-)

- **USB High-Speed (480 MBit/s)** with USBTMC class support  
  Supports multiple GPIB devices on a single adapter
- **Tiny formfactor**
- **GPIB compliant IO characteristics** including termination
- **GPIB HS488 mode**  
  Up to **8 MBytes/s** peak speed (if your instrument supports HS488 mode, but normal mode is on par with commercial adapters!)  
- **100MBit/s Ethernet with VXI-11 and PoE**
  Also works with USB power and non-PoE Ethernet
- **GPIB Device Mode Support**
- **Fully GPIB-compliant I/O characteristics**

Here a rendering - my actual prototype is too ugly to show :-D

<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/NewHighSpeedEthernetUsbVersion.png" width="50%"/>
(Unfortunately such a nice transparent housing will be far too expensive to produce due to high NRE)

## What You Get as Betatester

- Free hardware shipped directly to you  
- Beta testing starts **January 2026**  
- Final hardware expected **late November to mid-December 2026**
- Software already strongly progressed now

## Who I�m Looking For

You don�t need to tick every box�but here�s what would be amazing:

- Willingness to debug remotely and help zoom into issues  
  - I won�t nag for daily updates�good things take time  
  - The adapter includes built-in debug tools like trace generation  
- Ability to run tracing scripts and share trace files + issue descriptions  
  - I�ll send updates, you flash and retry�simple loop  
- Access to a variety of GPIB instruments from different eras
- **MacOSX testers** especially welcome (I can only test on Linux and Windows)  

## Logistics

- Preference for **European testers** (faster shipping and less costs for me, no customs hassle), but don't let yourself stop you from applying if you're outside Europe.
- Limited to **3 testers max** in this early phase, depending on coverage areas (otherwise I won't be able to handle all interactions) 
- Communication via forum or email -> your choice

## Interested?

Write me at: **Xyphro@gmail.com**  
Let�s build something awesome together.

---

# UsbGpib
Versatile, cheap, portable and robust USB to GPIB converter (USBTMC class based).

You'll find many projects like this, but this one is special (ok, everybody will claim this) :-)

V1 Hardware: 
<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/UsbGPIB.jpg" width="50%"/>

V2 Hardware: 
<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/Upcoming_Rev2.png" width="50%"/>

If you have a lot of test equipment at home, you might know the issues: Lots of devices only have GPIB as interface and the  GPIB adapters and GPIB cables on the market are very expensive and some of them even have many issues, when run under Windows 10 (device driver does not work). Or they e.g. are not able to be operated with VISA, because they are UART based, need special command sequences, ...

<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/SizeComparison.jpg" width="40%" align="right"/>
The adapters are also typically very long, such that they extend the overall length of your test equipment by at least 10cm (~4 inches).

Apart of the 2 very big manufacturers, other GPIB adapters, e.g. with Ethernet or also USB interface are not recognized my normal VISA providers or PyVisa, making the measurement control implementation specific for your GPIB adapter.

I've got frustrated and tried to turn it into something positive - Here a video showing the final device in action - click to view:

[![](https://img.youtube.com/vi/pZp1QZCXrF8/0.jpg)](https://www.youtube.com/watch?v=pZp1QZCXrF8) 


Some goals of the project were:
- Work based on the standard USBTMC protocol. This allows the GPIB test equipment to look like a normal USB based measurement device and work flawless with e.g. NI VISA, Labview, Matlab or PyVisa.
- Have a small length - otherwise my equipment has the risk of falling from the shelf :-) Also the USB cable should connect 90 degree angled, to make it very short.
- It should be cheap but still versatile (you can build a single one of these for only 14 USD!)
- It should support ALL my test equipment, from many different GPIB implementation generations and different GPIB flavors
- The Firmware should be upgradeable over USB
- It should be rock-solid (!) I don't want to end up in a very long measurement being interrupted because of a software issue of my USB GPIB converter.
- It should support additional features like serial poll, remote enabling/disabling
- If there is no GPIB device connected to the USBGpib converter, or the GPIB device is powered down, there should be no USB device visible on the PC.

All those goals are met.

# 21st April '25 update

Today it's time for another release - version V2.0. 

While writing this and looking at this readme.md file, I certainly think that I should clean it up in future - it is not well structured for new users. Sorry for this, but I have this on my radar and will improve. Whenever I have more time I will also extend it with usage examples / small "trainings", especially for those who are completely new to GPIB and controlling instruments.

I had btw. some occasions where users had issues flashing the device. Many of them were caused by the fact that the .hex and .bin files were saved over the GitHub web interface by rightclicking and selecting "Save as". This will result in a HTML page being saved which will not flash well :-)
The safest option is to do a git clone or to download this whole github repository as .zip file.

Here the release V2.0 details:
- **Bugfix:** the options autoid slower and autoid slowest did not work. This is addressed and I test those settings now also properly in release testing, promised :-) The issue was caused in my super slim parser implementation for custom internal commands and "autoid slow" and "autoid slower" start with the same letters causing hickups.
The Windows GUI is also updated to expose those 2 autoid settings - they were not included before.
As general good recommendation, I still recommend to turn autoid feature off, as it limits the measurement instruments which are used - Instruments HAVE to support *IDN? query for it to work well, which is for many old equipment not the case and for such old equipment errors will be flagged after the instrument is turned on.
- **New feature:** I enabled a new method to set the READ termination. One that is fully compliant with usbtmc standard and does not need a workaround with pulse indicator requests. I don't know why I did not spot this earlier, but a discussion with the usbtmc linux kernel mode driver maintainer exposed this to me. When using direct access to usbtmc kernel mode driver you can use the <a href="https://github.com/dpenkler/linux-usbtmc?tab=readme-ov-file#ioctl-to-control-setting-eom-bit" target="_blank">USBTMC_IOCTL_EOM_ENABLE</a> IOCTL to set the read termination. When using a VISA layer, you can set the read termination by setting the VI_ATTR_TERMCHAR_EN and VI_ATTR_TERMCHAR attributes (see below for more details). 
The previous method of setting read termination is not modified in any way, this just gives an additional method to set the readtermination.
- **Bugfix / Improvement:** The pulse indicator request function did use a blocking delay to pulse the LED. This delayed USB side handling. In some cases this could trigger timeouts. The LED blinking is now handled fully asynchronous and those timeouts will not occur anymore.

# 6th April '25 update

Time to release a bugfix (reported by rapgenic #80): V1.9 of the firmware fixes an issue where the "autoid slow" setting was not applied properly.

# 12th January '25 update

## New Gui available! (Windows)

I implemented a small GUI to change the non-volatile settings of the GpibUSB adapter. Sorry - it's windows only :-)
It is a single .exe file without external dependencies. You can just download it, run it and immediately change settings.

<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/UsbGpibGUI.png" width="80%"/>

Please download it from this folder: [SW/UsbGpibGUI](SW/UsbGpibGUI)

The source code is included in the src sub-directory.

## Python GUI (Linux)

UsbGpib user Jim Houston created an impressive pure Python GUI similar to mine - and I believe he began working on it even before I released mine. His GUI stands out for its versatility and platform independence, requiring only Python to run. It can be executed directly with Python 3 and should run on most platforms.

For details about it, have a look into this discussion thread: [https://github.com/xyphro/UsbGpib/discussions/55](https://github.com/xyphro/UsbGpib/discussions/55)

The GUI can be found here including documentation: [SW/NiceGUI](SW/NiceGUI)

## New Firmware

I release also herewith Version 1.8 of the firmware. 
Mainly a bug-fix was done, that the string setting now also returns short, when it is actually set to short. (issue report #59).

# 13th January '24 update

New year, new update :-)

I realized a user suggestion, which is a better human readable interface to the internal instrument settings.
This also exposes new functionality, e.g. shortening of the visa resource string, adjustable delayed application of AutoID featured, ...
The read termination setting can now be set in a volatile way and also be read back.

A slight issue sneaked in also and got fixed: After an instrument clear the next GPIB transfer timed out. Nobody found it so far though :-) But I fixed it.

Have fun using this!
(would anybody volunteers to make a simple cross platform gui for the non volatile settings :-) ?)

# 19th December '23 update

Finally I made it - the new HW design is on this page. As I had to restructure the repository a bit it took quite a while.
The software image is always the same and runs on both HW revisions.

<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/Upcoming_Rev2.png" width="40%"/>

I also have put a binary for a new Firmware image in there. I am still busy optimizing certain things but want to give you a try before doing a next release.
Please if you test it: Feed it back to me - does it work / does it break anything -> sharing is caring. 

Mail me at xyphro@gmail.com or raise an issue under the issue section.

[FW image location: SW/binaries](SW/binaries)

- 488.2 support further rolled out. This means Status bytes are automatically read out and reported via interrupt transfers to PC as foreseen in the standard.
- FASTER (!) 7 x write speed improvement and 4.x times read speed improvement. As reference: On an FSW I get 310kBytes/s as write transfer speed and 240kBytes/s as read transfer speed.
- a fix for read status byte readout leading to issues on my CMU200
- several smaller race conditions identified and fixed. A lot of work went into stress testing 488.2.

As said, I feel this is the best Firmware image so far, but I changed so much that I want to go first for this small beta test phase asking you explicitly for positive and negative feedback.

# Hardware

## Microcontroller choice

Although I typically would prefer nowadays an ARM Cortex M0/3/4/7 controller, there is an issue with it. Available devices support only max. 3.3V supply voltages, such that there would be a requirement for a level-shifter towards the GPIB Bus.
GPIB is based on 5V (not exactly true, but a first iteration).

This limited the microcontroller choice to e.g. AVR or PIC controllers. Because of very good availability I ended up in ATMEGA32U4 controllers.
Apart of the device supporting 5V I/O voltages, it also does not require a regulator to be part of the application - it has an internal 3.3V regulator. This minimizes the full application schematic and BOM.

Apart from that, there is an excellent USB stack available [http://www.fourwalledcubicle.com/LUFA.php](http://www.fourwalledcubicle.com/LUFA.php).

The GPIB side of the schematic can be directly connected to the ATMega32U4 IO pins. The IO pins from the microcontroller side are only set to 2 different states: Tri-state (input) or output LOW, to talk over GPIB.

## Component sourcing

All components are easy to source, so I only specify the potential critical ones:

- 16 MHz Crystal: Farnell 2853867 - MCSJK-7E-16.00-8-30-100-B-30 
- REV 1 GPIB connector: Farnell 2421095 - NORCOMP 112-024-113R001. For REV 2 use a straight 24P male solder type connector e.g. from AliExpress.
- USB connector: Farnell 2668483 - Amphenol ICC 61729-1011BLF

## PCB

The PCB can be ordered at nearly any PCB pool production service (e.g. 10 PCBs for 2 USD + shipping). The gerber files are included in the "HW/Gerber files" subdirectory.

## Mounting the PCB & Flashing the firmware

The PCB is available in 2 revisions.
- [REV 1](HW/REV1/README.md) is the most popularly used right now due to age. It has a USB Type-B connector and an L-shaped housing visible on a few photos of this page.
- [REV 2](HW/REV2/README.md) has some improvements like being smaller, better fit and USB Type-C connector.

Choose whatever you prefer. The software images, but also the external behavior is the same.

# Housings

## REV 1
<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/housing.png" width="33%"/><img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/housing_snap.png" width="50%"/>

I created a sophisticated 3D printable housing for this adapter. The design was made with Fusion 360. The project file + the STL files are included in the "Housing" sub directory.

The PCB fits perfectly into it. Optionally it can be fixed with 2 mounting screws (the GPIB connector has 2 threads, use 2 times 4-40 UNC x 3/8) and the TOP cover snaps onto the housing base.

I printed this using an Ender 5 3D printer with black PLA, 0.15mm layer height, 1mm wall thickness, no support.
Take care, that you rotate the TOP part of the housing by 180 degrees, so that the flat side is located on the printer bed.
Printing works fine, several iterations of the design were made to ensure good printability.
I printed so far 15 housings, without a single fail.

More information on this REV 1 can be found here: [REV 1](HW/REV1/README.md).
Note, that also the programming/build instructions moved to this location.

## REV 2

<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/Upcoming_Rev2.png" width="40%"/>

The REV 2 housing is a lot smaller, but requires 2 screws.
The housing is quite important to be able to connect and disconnect the board without breaking anything. It is key for mechanical stability of the adapter.
When operating the device without housing, take very well care when plugging in and out the board in case the GPIB connector has a very tight fit.

Information on how to build it can be found in the HW/REV 2 folder: [REV 2](HW/REV2/README.md).
Note, that also the programming/build instructions moved to this location.

# Software

## Source code

The source code of the Boot loader (slightly modified LUFA MassStorage Boot loader) and the main USBGPIB converter are located in the "SW" subdirectory.
At the time of publication LUFA 170418 release was used, with GCC as compiler.

Note: The Software is compatible with any HW revision in this repository. For REV 1 and REV 2 hardware you don't need different SW images.

## Binaries

For those, that just want to create their own device, I've included the binary output in the "SW/binaries" subdirectory.

# Using the device

## USB enumeration

You might be surprised initially, that the device does not show up in your device manager (or lsusb), when you connect only the USB side. This is a feature, not a bug (really!).
Only, if a GPIB device is connected, you can see the device on your PC too.

The reason behind the feature is simple: Instead of having a standard GPIB wiring, where you have a single GPIB controller and lots of GPIB devices interconnected, USBGPIB supports only a direct connection of the USBGPIB device to your measurement device. If you have like me e.g. 14 Instruments you don't want all to show up in the device manager, if the measurement device itself is powered down - you won't anyway be able to communicate with a powered down device.

When USB and the GPIB side is connected, the device enumerates. The USBGPIB device reads out the ID of the instrument and constructs a unique USB Serial number out of it. It is thus easily possible to assiate multiple connected USBGPIB devices with the measurement instrument.

The VISA ressource name is constructed from this USB Serial number. You can identify easily e.g. in NiMax, which device is connected:

<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/NiMaxExample.png" width="90%"/>

If you connect your USBGPIB device afterwards to another GPIB measurement device, it will disconnect and connect with a new serial number string, matching the other GPIB device *IDN? response again.

## GPIB settings on your measurement device

GPIBUSB does probe all GPIB primary addresses (and secondary address 0) for presence of a GPIB Talker/listener. It is thus not required to set a specific GPIB address - GPIBUSB will find it itself.

The only importance setting on the measurement device is, that the GPIB interface is enabled, which is typically the case.

## LED indicator

The LED indicates different states:

LED blinking: The USBGPIB converter is connected to a measurement instrument, it is powered off or its GPIB port is disabled. In this state, the device is also not connected to USB and will not show up in the device manager or lsusb.
LED on: The device is connected to a measurement device and GPIB communication possible. It is also accessible over USB
LED off: The device is not connected over USB, or the PC powered off :-)

## Controlling GPIB devices
<img src="https://raw.githubusercontent.com/xyphro/UsbGpib/master/pictures/towerOfGpib.jpg" width="30%" align="right"/>


As this converter implements the standard USBTMC Test and measurement class, you can control your instrument from ANY of the normal VISA tools. I tried so far R&S VISA and NI Visa, using Python and Matlab to talk to the devices.

# Testing status

## Sucessfully tested measurement equipment:
- R&S FSEM30
- R&S SMIQ03
- R&S CMU200
- R&S SMW200A
- R&S FSW
- R&S FSV
- Keithley 199 multimeter
- HP 34401 DMM from different generations / with different FW versions
- HP 3325A synthesizer/frequency generator
- HP 3457A multimeter
- Agilent E4406A VSA transmitter tester
- Tektronix TDS7104 Digital Phosphor Oscilloscope
- HP 8596A spectrum analyzer
- Agilent E3648A dual power supply

## Scenarious

- It works under all popular operating systems like Windows 7 and 10, 11, Linux and MacOSX
- USB1.1, USB2.0 and USB3.x ports tested, with and without USB HUB in between.
- The connection stays responsive, when power cycling the PC, or hibernating/sleeping it
- Different connection cycles (GPIB side connected first, USB side connected first, swapping GPIB side equipment, ...)
- Extensive testing of timeout scenarious. E.g. making an illegal query and testing, if the USBTMC handles the timeouts properly. This was a very tricky part to get right.
- Tested special transfer modes. E.g. capturing screenshots from different equipment is usually something, which will drive other GPIB adapters to the limits, because binary data of unknown length needs to be transported successfully.

# Setting Parameters

The firmware version from 13th January 2024 onwards has the ability for human readable text base configuration of several parameters.
The previous methods are still supported, but won't be further documented. You can look them back in the history of this file.

The command parser is quite simple. For that reason follow the exact syntax as shown below. 
Don't add extra spaces or make other modifications or concatenate commands.

## Read termination method

While most GPIB interfaces use the hardware signal EOI to signal the end of a message, not all old equipment supports it. Some older instruments even don't have the EOI pin hardware wise wired and use \r or \n termination.

The USB-TMC standard allows to set the read termination. While in firmware versions before < 2.0 I did not enable that method, it is now finally supported with standard compliance.

I document here the older method (using pulse indicator request), but I add also the newer methods. You can choose which ones to use, but in some cases pulse indicator requests can be difficult to issue, for which reason it is likely better to use the standard compliant method.


### Set read termination to CR (\r):
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!term cr')
```
Above setting is volatile. To make this a permanent setting call the below mentioned "!term store" command.

Alternatively you can set the termination directly with visa means, e.g. using PyVisa:
```
dev.set_visa_attribute(visa.constants.VI_ATTR_TERMCHAR_EN, True)
dev.set_visa_attribute(visa.constants.VI_ATTR_TERMCHAR, ord('\r') )
```

### Set read termination to LF (\n):
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!term lf')
```
Above setting is volatile. To make this a permanent setting call the below mentioned "!term store" command.

Alternatively you can set the termination directly with visa means, e.g. using PyVisa:
```
dev.set_visa_attribute(visa.constants.VI_ATTR_TERMCHAR_EN, True)
dev.set_visa_attribute(visa.constants.VI_ATTR_TERMCHAR, ord('\n') )
```

### Set read termination to EOI only (default setting):
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!term eoi')
```
Above setting is volatile. To make this a permanent setting call the below mentioned "!term store" command.

Alternatively you can set the termination directly with visa means, e.g. using PyVisa:
```
dev.set_visa_attribute(visa.constants.VI_ATTR_TERMCHAR_EN, True)
dev.set_visa_attribute(visa.constants.VI_ATTR_TERMCHAR, ord('\0') )
```

### Save readtermination setting in eeprom (make them non-volatile)
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!term store')
```

### Query current termination setting from device
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
print(dev.query('!term?'))
```
This returns a text string containing "lf", "cr" or "eoi"


## AutoID setting

Default wise the GPIB adapter tries during power on of the instrument to query using *IDN? or ID? commands the instrument name automatically.
This is used to build the USB serial number, which finally gets part of the VISA ressource string.

Not all instruments support this *IDN / ID? query. For this reason this feature can be turned off.
The serial number will then be built based on the GPIB address of the instrument.

### Turn AutoID feature off
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!autoid off')
```
This setting is stored in eeprom = non volatile memory, so will survive a power cycle

### turn AutoID feature on
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!autoid on')
```
This setting is stored in eeprom = non volatile memory, so will survive a power cycle

### turn auto ID on with delay

Some instruments need after turn on some seconds before GPIB is responsive.

With below 3 settings you can set a delay which is applied before the instrument ID is queried after power on.
Note that it will take then also more time, before the USB device is recognized by the PC.

Also this setting is non-volatile.

Delay 5 seconds:
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!autoid slow')
```

Delay 15 seconds:
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!autoid slower')
```

Delay 30 seconds:
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!autoid slowest')
```

### read autoid setting

```
dev.control_in(0xa1, 0x40, 0, 0, 1); # 
print(dev.query('!autoid?'))
```
Returns as text string either: "off", "on", "slow", "slower" or "slowest".

## Firmware version

Finally I implemented a command to query the USB adapters firmware version :-)

```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
print(dev.query('!ver?'))
```

## Shorten ressource strings (Matlab)

A user discovered that Matlab has a limitation in the VISA ressource string length and shared a pull request to reduce the length.
I expose this now first time in the baseline firmware with the following options.

This setting is stored in eeprom = non volatile.

### Limit the USB serial number to a length of 20 characters
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!string short')
```

### disable limitation of USB serial number length (default behavior)
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
dev.write('!string normal')
```

### query the string length setting.
```
dev.control_in(0xa1, 0x40, 0, 0, 1); # USBTMC pulse indicator request (enables internal command processing)
print(dev.query('!string?'))
```

This returns as text string either "normal" or "short".

## reset the adapter
```
dev.control_in(0xa1, 0x40, 0, 0, 1); 
dev.write('!reset')
```

Do a reset of the adapter. Note that due to the reset you have to close the visa session and start a new one.


# If you got here and you want to support me, here is your chance :-)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/J3J7WPWSQ)
