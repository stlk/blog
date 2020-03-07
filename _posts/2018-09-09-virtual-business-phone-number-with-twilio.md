---
layout: post
title: Virtual support phone number with Twilio
---

For a long time I was looking for a simple and, well, cheap solution to isolate my support phone number from my personal number and I'm going to share my solution with you.

I like how Twilio makes things simple for developers so they were my first and only bet. What was I actually trying to achieve? I wanted to have a phone number that I could redirect to my phone and when the traffic gets high enough send caller to voicemail when needed. I also wanted to make a phone calls from that number using SIP phone.

First, create a Twilio account and get voice-enabled phone number. Now we need to connect the dots. Let's assume *+420111111111* is your number and *+420999999999* is the number you got from Twilio.

### Receiving calls

Twilio uses TwiML a markup language used to tell Twilio what to do when you receive an incoming call or SMS. The place where we will store our TwiML commands is called TwiML Bins. TwiML Bins allow you to write TwiML that Twilio will host for you - so we don't have to spin up a web server.

[Runtime > TwiML Bins](https://www.twilio.com/console/runtime/twiml-bins) is the place where we can manage our Bins.

Create a new bin called `Forward incoming calls to my phone` and paste following code.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Dial callerId="+420999999999">
    +420111111111
  </Dial>
</Response>
```

<p class="post__image-center">
  <img src="/public/twilio-incoming-call-bin.png" alt="" class="post__image post__image-full-width">
</p>

Then we need to associate this bin with a phone number. In [Phone Numbers > Manage Numbers](https://www.twilio.com/console/phone-numbers/incoming) section select your phone number and look for a field called *A call comes in*. Select *TwiML* and the select the bin we just created.

<p class="post__image-center">
  <img src="/public/twilio-number-management.png" alt="" class="post__image post__image-full-width">
</p>

That's it, you should now be able to accept incoming phone calls. Try calling your Twilio number.

### Making calls using your Twilio number

To make outgoing call using the Twilio number we will need to use VoIP application like [Zoiper](https://www.zoiper.com/en/products) which is available on Android and iOS. To use this application we need to set up a SIP domain. In [Programmable Voice > SIP Domains](https://www.twilio.com/console/voice/sip/endpoints) create a new domain.

<p class="post__image-center">
  <img src="/public/twilio-new-sip-domain.png" alt="" class="post__image post__image-full-width">
</p>

Make sure that you associate credentials for both *Voice Authentication* and *SIP Registration*. Then we create another TwiML Bin to handle outgoing calls.

```xml
{% raw %}
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Dial callerId="+420999999999">
    {{#e164}}{{To}}{{/e164}}
  </Dial>
</Response>
{% endraw %}
```

The interesting part of this command is `{% raw %}{{#e164}}{{To}}{{/e164}}{% endraw %}`. When using SIP Interface the `To` value that gets passed down is in following form `sip:+15125551212@freelance.sip.us1.twilio.com`. Function `e164` extracts the phone number from this text. After saving this bin we will copy its *URL* and use it as *Request URL* in the SIP Domain we created.

### Zoiper configuration

Now that Twilio is setup we can actually make an outgoing call. We will use our SIP credentials we created before. You might have to fill in domain with the region `us1` as domain without region or with any [other](https://www.twilio.com/docs/voice/api/sending-sip#localized-sip-uris) didn't work for me.

<p class="post__image-center">
  <img src="/public/twilio-zoiper-configuration.jpeg" alt="" class="post__image post__image-full-width">
</p>

I'm pretty happy with the way how this solution works and the calls and number itself are pretty cheap.
