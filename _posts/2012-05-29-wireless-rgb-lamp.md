---
layout: post
title: Wireless RGB lamp

excerpt: "For a long time I wanted to have a wirelessly controlled RGB light in my kitchen. This idea got more specific shape when I saw hacked IKEA dioder light where original PIC microcontroller was removed and ATTiny took its place."
---

For a long time I wanted to have a wirelessly controlled RGB light in my kitchen. This idea got more specific shape when I saw hacked IKEA dioder light where original PIC microcontroller was removed and ATTiny took its place. This wasn't the one I ended up using, but I think it's worth mentioning. I have decided to use [5,1W RGB LED][1] along with ATMega328P with Arduino bootloader and XBee for the wireless connectivity.<a href="http://www.flickr.com/photos/stalker_cz/7181211930/" title="Link to Flickr"><img src="http://farm6.staticflickr.com/5348/7181211930_369958da68_n.jpg" class="right" /></a>

Arduino is running on 3,3V so it can safely communicate with XBee which means the only special circuit needed is LED controller. Controller consists of three [BD139-16][2] transistors and three high power resistors. Base current is limited by 470 ohm resistors. Thing which scared me a bit when I was testing this circuit is that high power resistors were getting pretty hot. But values I measured were correct and yeah, it's good idea to check datasheet. I found that high temperature is normal. There is another thing worth mentioning. Don't forget to connect AVCC pin to VCC, because it is required when running on 3,3V with <abbr title="Brown Out Detector">BOD</abbr> enabled. You will save yourself few hours wondering why the hell it doesn't work on 3,3V.

XBee I'm using is XBee Series 2 with API firmware, because it allows me to easily create mesh of devices. My current setup is XBee Controller connected to [Netduino Plus][3] and two end devices. [Wireless LCD][4] and RGB Lamp. Netduino serves as gateway to the internet (for temperature sensors) and LAN (for [controlling lamp][5]). Netduino runs lightweight webserver and when request is received it sends ZigBee packet to the lamp. This way I can change mode by visiting *http://192.168.1.95/mode/mode_id*. I can do the same on Wireless LCD so I don't have to run to my PC every time I want to control the lamp. I will describe both ways in future posts.

Firmware is pretty straightforward. It listens for commands, stores settings on EEPROM and fades the led. Commands are received in form of ZigBee Receive packet where first byte specifies command id and following bytes are different for each command. Commands are described in table below.

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

###Schematic

<iframe width='600' height='350' frameborder='0' src='http://c.circuitbee.com/build/r/schematic-embed.html?id=0000000249'></iframe>

###Part list

<table>
<thead>
<tr><th>ID</th><th>Description</th><th>Count</th></tr>
</thead>
<tbody>
<tr>
<td>R4, R5, R6</td>
<td>470Ω</td>
<td>3x</td>
</tr>
<tr>
<td>R2, E3</td>
<td>4,7Ω</td>
<td>2x</td>
</tr>
<tr>
<td>R1</td>
<td>8,2Ω</td>
<td>1x</td>
</tr>
<tr>
<td>T1, T2, T3</td>
<td>BD139-16</td>
<td>3x</td>
</tr>
<tr>
<td>RES1</td>
<td>Ceramic Resonator 16MHz</td>
<td>1x</td>
</tr>
<tr>
<td>D1, D2, D3</td>
<td><a href="http://www.gme.cz/vykonove-led-nad-1w/l-lxhl-hprgb-p511-558/" title="Link to Czech shop GM electronic">L-LXHL-HPRGB</a></td>
<td>1x</td>
</tr>
<tr>
<td>IC1</td>
<td>ATMega 328P</td>
<td>1x</td>
</tr>
<tr>
<td>IC2</td>
<td>XBee Series 2</td>
<td>1x</td>
</tr>
</tbody>
</table>

###Source code

Source code is available on [Github][6].


  [1]: http://www.gme.cz/vykonove-led-nad-1w/l-lxhl-hprgb-p511-558/ "Link to Czech shop GM electronic"
  [2]: http://www.gme.cz/bipolarni-tranzistory-npn/bd139-16-p211-010/ "Link to Czech shop GM electronic"
  [3]: http://www.flickr.com/photos/stalker_cz/6368627583/ "Link to Flickr"
  [4]: http://www.flickr.com/photos/stalker_cz/7145245471/ "Link to Flickr"
  [5]: http://rousek.name/blog/wireless-rgb-lamp-the-controller "Link to my blog post"
  [6]: https://github.com/stlk/Arduino/blob/master/stlk_lamp/stlk_lamp.ino "Source code on Github"