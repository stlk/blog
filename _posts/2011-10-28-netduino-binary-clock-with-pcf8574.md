---
layout: post
title: Netduino binary clock with PCF8574

excerpt: "Recently I found yet another binary clock and I realised that it might be nice way to try I<sup>2</sup>C expander **PCF8574**."
---

Recently I found yet another binary clock and I realised that it might be nice way to try I<sup>2</sup>C expander **PCF8574**. [PCF8574][1] is cheap and easy to use 8bit I<sup>2</sup>C expander. Schematic below shows the minute part of circuit. Hour part is exactly the same except you can leave out one LED.

<iframe width="600" height="350" frameborder="0" src="http://c.circuitbee.com/build/r/schematic-embed.html?id=0000000192"></iframe>

There are two points that I'd like to highlight. When controlling LEDs with this chip you dont need to use resistors. And secondly this chip is current sink which means we need to send 1 to turn LED off and 0 to turn it on.

So now we have the circuit, but how to feed it with data? Code below handles conversion of number to pseudo binary representation.

{% highlight cpp %}
    int digit2 = intVal % 10; // calculate 2nd digit

    byte byteVal = (byte)digit2; // convert to byte

    if (intVal != digit2) // if number is higher than 9
    {
        intVal /= 10; // divide by 10 to get single digit

        byteVal += (byte)(intVal << 4);
        // shift first digit by 4 bits(for example 00000001 becomes 00010000)
    }

    byte data = (byte)(~(int)byteVal); // invert value, explained earlier
{% endhighlight %}

With data to send we just need to find out how to send them. I<sup>2</sup>C communication examples can be found almost everywhere so I will explain only values. Adress is determinated by pins A<sub>0</sub>, A<sub>1</sub>, A<sub>2</sub>. Binary representation is 0100A<sub>2</sub>A<sub>1</sub>A<sub>0</sub>. With all pins connected to GND we get 0100000 which is 20 in hexadecimal.

{% highlight cpp %}
    // 0x20 is address in hexadecimal format, 100 is maximal frequency for this chip in kilohertz
    I2CDevice.Configuration configChip1 = new I2CDevice.Configuration(0x20, 100);
    I2CDevice i2cbus = new I2CDevice(configChip1);

    lock (i2cbus)
    {
        I2CDevice.I2CTransaction[] xact = new I2CDevice.I2CTransaction[]
        {                  
            I2CDevice.CreateWriteTransaction(new byte[] { data })
        };

        i2cbus.Execute(xact, 3000);
    }
{% endhighlight %}

Finally, here is [prototype][2] on breadboard and whole class I use in my binary clock.

{% highlight cpp %}
    public class BinaryClock
    {
        private I2CDevice.Configuration configChip1 = new I2CDevice.Configuration(0x20, 100);
        // all three connected to GND
        private I2CDevice.Configuration configChip2 = new I2CDevice.Configuration(0x21, 100);
        // 1st one connected to 5V, rest to GND

        private I2CDevice _i2cbus;

        public BinaryClock()
        {
            _i2cbus = new I2CDevice(configChip1);
        }

        public void Tick()
        {
            var dt = DateTime.Now;

            SendByte(IntToByte(dt.Hour), configChip1);
            SendByte(IntToByte(dt.Minute), configChip2);
        }

        private void SendByte(byte data, I2CDevice.Configuration config)
        {
            lock (_i2cbus)
            {
                _i2cbus.Config = config;

                I2CDevice.I2CTransaction[] xact = new I2CDevice.I2CTransaction[]
                {                  
                    I2CDevice.CreateWriteTransaction(new byte[] { data })
                };

                _i2cbus.Execute(xact, 3000);
            }
        }

        private byte IntToByte(int intVal)
        {
            int digit2 = intVal % 10;

            byte byteVal = (byte)digit2;

            if (intVal != digit2)
            {
                intVal /= 10;
                byteVal += (byte)(intVal << 4);
            }

            return (byte)(~(int)byteVal);
        }

    }
{% endhighlight %}

  [1]: http://www.gme.cz/i-o-obvody-multifunkcni-periferie/pcf8574p-p433-053/ "Link to czech shop"
  [2]: /public/binary_clock.jpg