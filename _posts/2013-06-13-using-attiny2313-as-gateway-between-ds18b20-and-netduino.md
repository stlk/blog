---
layout: post
title: Using ATTiny2313 as gateway between DS18B20 and Netduino

excerpt: "For some time I was happy user of Stanislav \"CW2\" Šimíček's modified firmware which brought OneWire capability to Netduino, but new version(4.2) of .NET MicroFramework came and I realized there are other possibilities."
---

For some time I was happy user of [Stanislav "CW2" Šimíček's][1] modified firmware which brought OneWire capability to Netduino, but new version(4.2) of .NET MicroFramework came and I realized there are other possibilities. More specifically I realized that this is good opportunity to make something useful with C on Atmel chips.

I decided to build device which reads temperature and sends it back to Netduino using UART. For this task I've chosen ATTiny2313 which was cheapest chip with UART at the time of buying.

<a href="http://www.flickr.com/photos/stalker_cz/9033527511/" title="ATTiny2313 by JosefRousek, on Flickr"><img src="http://farm4.staticflickr.com/3738/9033527511_cb41ab499e_n.jpg" width="320" height="218" alt="ATTiny2313" class="right"></a>

As you can see the circuit is terribly simple. I've used external crystal because I couldn't make UART work with internal one. But there might be something else which was causing my problems.

The program is a bit more interesting but its fairly simple. It waits for character "t" followed by "1" or "2" and replies with it's identificator(t1 or t2) followed by LSB and MSB of temperature. I then convert temperature to float using following statement.

``float temp = ((short)((MSB << 8) | LSB)) / 16F;``

I'm using minimal implementation of OneWire protocol originally written by [Walda][2] with added CRC verification. This implementation requires you to use separate pin for each sensor. But it isn't an issue for me. Almost each line of source code is commented so I will rather focus on describing my configuration.

My ATTiny is running on 16MHz, this is set on beginning of both *makefile* and *main.c*. Another noteworthy thing is programmer options described on line [270 of makefile][3]. I'm using Arduino as programmer which is disguised as stk500v1, it is connected to port COM4 and programming speed is 19200bps.

###Netduino side

Here is my overly complicated way of handling UART communication on .NETMF where temperature is converted to readable format on line 64.

<script src="https://gist.github.com/stlk/5791735.js"></script>

###Source code

Full source code is available on [GitHub.com][4]. Source code is licenced under [CC BY 3.0][5].


  [1]: http://forums.netduino.com/index.php?/topic/230-onewire-alpha/ "Link to Netduino.com forums"
  [2]: http://walda.starhill.org/elektronika-avr-gcc-stripky.html "Walda's website"
  [3]: https://github.com/stlk/avr/blob/master/attiny2313_ds18b20/makefile#L270 "Link to line 270 of makefile on GitHub.com"
  [4]: https://github.com/stlk/avr/blob/master/attiny2313_ds18b20 "Link to GitHub.com"
  [5]: http://creativecommons.org/licenses/by/3.0/