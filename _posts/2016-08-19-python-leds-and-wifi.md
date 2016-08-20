---
layout: post
title: "Python, LEDs and WiFi"
---

I lost interest in electrotechnics almost a year ago. I turned off my binary clock, temperature sensor and left it in a drawer. But before that I bought ESP8266, cheap WiFi microcontroller running lua. It was unreliable and hard to program.

Fast forward year later I was stuck in a bed on a sunny summer day. I remembered local PyLadies group is going to hold an ESP8266 workshop. Out of pure boredom I decided to give it another chance, this time using the MicroPython project. I was pretty lucky as it was only few days after they released [binaries](https://micropython.org/download/#esp8266) for ESP8266 to the public.

First you need to flash the board using following commands.

{% highlight bash %}
$ esptool.py --port /dev/tty.wchusbserial1420 erase_flash
esptool.py v1.0.2-dev
Connecting...
Erasing flash (this may take a while)...

$ esptool.py --port /dev/tty.wchusbserial1420 write_flash --flash_size=8m 0 esp8266-2016-07-31-v1.8.2-75-g4d22ade.bin
esptool.py v1.2-dev
Connecting...
Running Cesanta flasher stub...
Flash params set to 0x0020
Writing 540672 @ 0x0... 540672 (100 %)
Wrote 540672 bytes at 0x0 in 13.1 seconds (330.8 kbit/s)...
Leaving...
{% endhighlight %}

After restarting the device you can connect to python terminal using serial console of your choice. For me it is `screen /dev/tty.wchusbserial1420 115200`. You can then play around in REPL. I'm using [NodeMCU](https://github.com/nodemcu/nodemcu-devkit) so I have blue led on port 16 so the first thing I did was to turn it on.

{% highlight python %}
>>> import machine
>>> pin = machine.Pin(16, machine.Pin.OUT)
>>> pin.low() # pull the pin low to turn on the light
{% endhighlight %}

To develop more complex applications it would be great to have a way to upload files to the module. Luckily there's tool called [webrepl_cli](https://github.com/micropython/webrepl).

Lets create file called `light.py` with following content.

{% highlight python %}
import machine
pin = machine.Pin(16, machine.Pin.OUT)

def on():
    pin.low()

def off():
    pin.high()
{% endhighlight %}

And upload it to the device using following command.

{% highlight bash %}
$ python webrepl_cli.py light.py 192.168.1.251:/
put 192.168.1.251 8266
light.py -> /light.py
Password:
Sent 326 of 326 bytes
{% endhighlight %}

<blockquote>
There's a chance that the command fails. Thats because webrepl might not be enabled by default. To enable it run <code>import webrepl; webrepl.start();</code>.
</blockquote>

You can then use the file like every other module.

{% highlight python %}
>>> import light
>>> light.on()
>>> light.off()
{% endhighlight %}

Micropython supports various protocols. Eg. [onewire](http://docs.micropython.org/en/latest/esp8266/esp8266/quickref.html#onewire-driver), [neopixel](http://docs.micropython.org/en/latest/esp8266/esp8266/quickref.html#neopixel-driver) and [others](http://docs.micropython.org/en/latest/esp8266/esp8266_contents.html). I had some leftover DS18B20 temperature sensor and neopixel LED so I combined them together and created WiFi thermometer with RGB status LED.

Let's start with the temperature sensor.

{% highlight python %}
pin = machine.Pin(5) # the sensor is on GPIO14
ds = onewire.DS18B20(onewire.OneWire(pin))

roms = ds.scan() # scan for devices on the bus
print('found devices:', roms)
print('temperature:', end=' ')
ds.convert_temp() # tell the sensor to start reading temperature
time.sleep_ms(750) # sensor needs some time read the temperature
temp = ds.read_temp(roms[0]) # we know we only have one sensor connected
print(temp)
{% endhighlight %}

The NeoPixel is even simpler. I created [module](https://github.com/stlk/micropython/blob/master/temperature/status.py) that allows me to set the color using just one method call.

{% highlight bash %}
from machine import Pin
from neopixel import NeoPixel

pin = Pin(4, Pin.OUT) # the NeoPixel is on GPIO2
np = NeoPixel(pin, 1) # only one pixel is connected

np[0] = (255, 128, 0)
np.write()
{% endhighlight %}

With temperature reading and status notifications last part left to do is to send the data somewhere. At first I tried to send HTTP requests, but it wasn't working well for some reason.

My next shot was [MQTT](https://github.com/micropython/micropython-lib/tree/master/umqtt.simple), which is lightweight publish/subscribe messaging transport designed especially for small devices. MicroPython implementation implements only subset of the protocol, the one I missed the most is authentication, because I couldn't use hosted MQTT server and had to start one myself.

{% highlight python %}
# You can find the library on https://github.com/micropython/micropython-lib/tree/master/umqtt.simple
from umqtt.simple import MQTTClient

c = MQTTClient('umqtt_client', 'example.com')
c.connect()
c.publish(b'status', b'hi') # send message "hi" to channel "status"

c.publish(b'temp', b'%s' % (temp,)) # send temperature reading to channel "temp"
{% endhighlight %}

I really enjoyed diving back to electrotechnics especially with python as a language of choice. You can find the finished project on [Github](https://github.com/stlk/micropython/tree/master/temperature).
