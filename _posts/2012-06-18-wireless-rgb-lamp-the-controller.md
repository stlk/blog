---
layout: post
title: Wireless RGB lamp - The controller
---

In this part I will show you how to control [RGB lamp][1] using Netduino. The most important part such as [wiring XBee to Netduino and XBee setup][2] is described in my previous article, so I will highlight only most interesting bits.

Method `RequestReceived` handles incoming HTTP requests and decides which command send to XBee.

If URL is (your XBee's IP)/fade/1000 where 1000 is delay in milliseconds, following code is executed. Because highest number we can squeeze to one byte is 255 we need to use two bytes to achieve more suitable range 0 - 65535. We then need to split that number into two bytes as shown below.

    short fadeDelay = short.Parse(param[2]); // param[2] is delay in milliseconds
    _xBee.SendData(0x0013A200407A26AA, 0xFFFE,
                    new byte[]
                        {
                            20,
                            (byte) (fadeDelay >> 8),
                            (byte) (fadeDelay & 0x00FF)
                        }, null);

Following two commands are simple. Just parse number and send it.

    case "mode":
        if (param.Length == 3)
        {
            byte fadeMode = byte.Parse(param[2]);
            _xBee.SendData(0x0013A200407A26AA, 0xFFFE, new byte[] { 21, fadeMode }, null);
        }
        break;
    case "color":
        if (param.Length == 3)
        {
            byte color = byte.Parse(param[2]);
            _xBee.SendData(0x0013A200407A26AA, 0xFFFE, new byte[] { 22, color }, null);
        }
        break;

Command *colorx* takes 3 arguments. So URL will look like this (your XBee's IP)/colorx/(0-255 red)/(0-255 green)/(0-255 blue)

    byte colorR = byte.Parse(param[2]);
    byte colorG = byte.Parse(param[3]);
    byte colorB = byte.Parse(param[4]);
    _xBee.SendData(0x0013A200407A26AA, 0xFFFE, new byte[] { 23, colorR, colorG, colorB }, null);

Last command sends just one byte which requests current settings. When it's received method `XBeeFrameReceived` gets called. You can put breakpoint into the empty `case` statement or print incoming data. Format is described in table below.

    _xBee.SendData(0x0013A200407A26AA, 0xFFFE, new byte[] { 24 }, null);
    

###Commands

<table>
<thead>
<tr><th>ID</th><th>Name</th><th>Data</th><th>Description</th></tr>
</thead>
<tbody>
<tr>
<td>20</td>
<td>Fade delay</td>
<td>high byte, low byte</td>
<td>0 - 65535 milliseconds</td>
</tr>
<tr>
<td>21</td>
<td>Mode</td>
<td>mode id</td>
<td>0 - disabled, 1 - single color, 2 - fading, 3 - strobe</td>
</tr>
<tr>
<td>22</td>
<td>Color</td>
<td>color id</td>
<td>0 - Red, 1 - Green, 2 - Blue, 3 - Yellow, 4 - Cyan, 5 - Orange, 6 - Magenta</td>
</tr>
<tr>
<td>23</td>
<td>Color</td>
<td>red byte, green byte, blue byte</td>
<td>0 - 255 for each color</td>
</tr>
<tr>
<td>24</td>
<td>Status</td>
<td colspan="2"><em>returns</em> current settings in format: 25, mode, [fade delay] high, low, [color] R, G, B</td>
</tr>
</tbody>
</table>

###Action shot
<p></p>
<iframe src="http://player.vimeo.com/video/44331592" width="500" height="281" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>

###Source code

Full source code is available on [GitHub.com][3].


  [1]: http://rousek.name/blog/wireless-rgb-lamp "Link to my previous post"
  [2]: http://rousek.name/blog/control-lights-remotely-using-xbee-and-netduino "Link to another blog post"
  [3]: https://github.com/stlk/RGBLampGateway "Link to GitHub.com"