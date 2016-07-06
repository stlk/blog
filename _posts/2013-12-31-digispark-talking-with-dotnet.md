---
layout: post
title: DigiSpark talking with .NET
---

With black Friday sales I decided to buy few DigiSparks to try them out and maybe spark some interest in electronics in my brother. Designer of the board did very good job, overall USB experience feels smooth and is easy to understand.

After some poking around I decided to try USB communication with C# on computer side. DigiSpark has DigiUSB library which is basically replacement of serial with much nicer handling of situations when disconnecting and connecting DigiSpark. I found working solution on DigiStump’s official forum so all credit for porting library to .NET goes to SmeeAgain.

Windows app is pretty simple. It waits for key to be pressed. If it’s “O” 1 is sent otherwise 0 is sent.

{% highlight cpp %}
if (Console.KeyAvailable) // If the key was pressed
{
  var data = (byte)(Console.ReadKey().Key == ConsoleKey.O ? 1 : 0); // If it was O send 1 else 0
  digiSpark.WriteBytes(new[] { data });
}
{% endhighlight %}

Print any character that we receive.

{% highlight cpp %}
byte[] value;
while (digiSpark.ReadByte(out value))
{
  Console.Write(Encoding.Default.GetString(value));
}
{% endhighlight %}

Application can detect when device is connected or disconnected using event `ArduinoUsbDeviceChangeNotifier`. There is one thing we need to be aware of. In order to receive events in console application we need to call `Application.DoEvents();` in each cycle. Because class `Application` is in namespace `System.Windows.Forms` we need to reference it.

There is nothing special on DigiSpark side too. We will control onboard LED. I’ve used breathing LED just to make it look nice. Sketch listens for incoming bytes, if its “1” `breathing` is set to true. In both cases we send LED’s status.

{% highlight cpp %}
if (DigiUSB.available() > 0) {
  breathing = DigiUSB.read() == 1;
  if(!breathing) {
    analogWrite(LED, 0);
  }
  if(breathing) {
    DigiUSB.write("1");
  } else {
    DigiUSB.write("0");
  }
}
{% endhighlight %}

## Source code
Full source code is available on [GitHub.com][1].


  [1]: https://github.com/stlk/DigiSparkDotNet "Link to GitHub.com"
