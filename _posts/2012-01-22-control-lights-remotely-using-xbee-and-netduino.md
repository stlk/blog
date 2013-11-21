---
layout: post
title: Control lights remotely using XBee and Netduino
---

I'm moving to my own appartment and I want to control lamp and mood light wirelessly. Because I want to extend mood light across whole appartment in future. I have decided to use XBee. Specifically XBee Series 2.

To start with XBees you need atleast 2 [XBees][1], [XBee shield][2] or [breakout boards][3] and some sort of USB adapter. For example [XBee Explorer][4] or [FT232RL USB to Serial breakout board][5].

**My setup is**

- 2 XBees
- 2 breakout boards
- FT232RL USB to Serial breakout board
- NetduinoPlus

One of nice features of XBee is availability of 10 I/O pins. This allows XBee to sample sensor data without any other microcontroller. XBee can also send sample only when selected digital input changes. This chapter is not well covered online, so I've decided to share my experience.

First of my XBees is running API Controller firmware, its connected to Netduino via serial port. API firmware allows you to receive I/O data from other XBees, which is exactly what I want. Second one is standalone, only two buttons and one lamp are connected. Its running end device API firmware.

Lets configure our XBees using Digi's configuration utility X-CTU.

**Controller configuration**<br />
ID - 70 // PAN ID - ID of your network<br />

**End device configuration**<br />
ID - 70 // PAN ID - ID of your network<br />
DH - 13A200 // High part of destination(coordinator) address<br />
DL - 407A26A8 // Low part of destination(coordinator) address<br />
D0 - 3 // Port set to digital input<br />
D1 - 3 // Port set to digital input<br />
D2 - 4 // Port set to digital output, low<br />
IC - 3 // Digital I/O change detection, enabled for pins D0 and D1<br />

Next step is wiring circuit for controller.

<iframe width='600' height='350' frameborder='0' src='http://c.circuitbee.com/build/r/schematic-embed.html?id=0000000214'></iframe>

And end device.

<iframe width='600' height='350' frameborder='0' src='http://c.circuitbee.com/build/r/schematic-embed.html?id=0000000215'></iframe>

Only obstacle on our way to sucess is code to learn Netduino how to talk with xBee. As I already mentioned I have decided to use API mode.

I'm using modified Asynchronus API XBee library from [Grommet][6]. My version is hosted on [GitHub][7], this version is modified to work with limited rescources of Netduino and more importantly to fulfill my needs. Because I have only one end device its address is hardcoded in source code. This will change sooner or later.

To setup the library open XBee.cs and modify *ID of recipient* as shown in example below. Serial number of my xBee end device is 0013A200407126AA. (**EDIT:** This is no longer needed)

    private readonly byte[] _txDataBase = new byte[] {
        0x0, // frame type
        0x1, // frame id, set to zero for no reply
        // ID of recipient, or use 0xFFFF for broadcast 
        0x0,
        0x13,
        0xA2,
        0x0,
        0x40,
        0x7A,
        0x26,
        0xAA,
        // 16 bit of recipient or 0xFFFE if unknown
        0xFF,
        0xFE
    };

Last thing I have prepared for you are some [examples][8].


  [1]: http://www.sparkfun.com/products/10414
  [2]: http://www.sparkfun.com/products/9976
  [3]: http://www.sparkfun.com/products/8276
  [4]: http://www.sparkfun.com/products/8687
  [5]: http://www.sparkfun.com/products/718
  [6]: http://grommet.codeplex.com/ "Link to Codeplex"
  [7]: https://github.com/stlk/XBee "Link to GitHub"
  [8]: https://github.com/stlk/XBee/blob/master/Examples/Example.cs