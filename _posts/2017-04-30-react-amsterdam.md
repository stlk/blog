---
layout: post
title: React Amsterdam 2017
---

Last Friday I attended my first React conference. It took place in Amsterdam. So once again I got the opportunity to visit Amsterdam because of a conference. I want to thank organizers of [PragueJS](https://www.meetup.com/praguejs/) meetup for giving me a ticket!

This was my biggest conference I ever attended and I must say the organization was pretty good. Location and technical production were perfect and overall I feel pretty positive about the conference.

It started on Thursday when [Tomislav Tenodi](https://twitter.com/TomislavTenodi) organized unofficial warmup party. I love those small events when I'm forced to talk to people. For me it's so much easier to meet people and it gets me into networking mood for a conference.

<p class="post__image-center"><img src="/public/react-amsterdam.jpg" alt="" class="post__image"></p>

## Talks

### Coding Mobile with the Pros
[Gant Laborde](https://twitter.com/GantLaborde) started the React Native track. His company Infinite Red maintains some pretty interesting project and he came to introduce them.

[Reactotron](https://github.com/infinitered/reactotron) - Headlined a desktop app for inspecting your React Native projects. You can inspect and change the apps state from nice and simple UI.

[Ignite](https://infinite.red/ignite) - Great tool to simplify library usage. It links a library into your app and generates examples. Will definitely try it.

### A Real-World GraphQL Application in Production
[Stefano Masini](https://twitter.com/stefanomasini) presented advanced how they build GraphQL server at scale. He presented some interesting ways to introduce caching and organize mutations. Architecture they used is called [Hexagonal Architecture](http://fideloper.com/hexagonal-architecture). He was also discussing GraphQL Subscriptions. They faced the problem of correctly subscribing to events. The approach they ended up using was to send all events happening on a page to a single channel.

### Test Like It's 2017
[Michele Bertoli](https://twitter.com/MicheleBertoli) is testing obsessed front end engineer working at Facebook. He presented few tools that will help us with testing.

[Jest utilities for Styled Components](https://github.com/styled-components/jest-styled-components) - Includes styled-components generated css in snapshots.

[Snapguidist](https://github.com/styleguidist/snapguidist) do snapshot-based testing inside your [React Styleguidist](https://github.com/styleguidist/react-styleguidist) styleguide.

[StoryShots](https://github.com/storybooks/storybook/tree/master/packages/storyshots) exactly same for [React Storybook](https://github.com/storybooks/storybook).

[React Fix It](https://github.com/MicheleBertoli/react-fix-it) is a great tool which generates test when exceptions occurs. Why should we manually reproduce test case when the browser has all the info to generate it.

### Navigating React Native Navigation
[Kurtis Kemple](https://twitter.com/kurtiskemple) and his team invested quite some time into investigating which router would suit them the most. He decided to share their learnings with us.

[Native Navigation](https://github.com/airbnb/native-navigation) promising approach based on building thin layer over native components.

[React Navigation](https://github.com/react-community/react-navigation) is aiming to share navigation logic between mobile apps, web apps, and server rendering.

I personally have used [React Native Router](https://github.com/aksonov/react-native-router-flux) and I'm quite happy with it.

### Complexity: Divide and Conquer!
[Michel Weststrate](https://twitter.com/mweststrate) author of MobX shared his thoughts on managing complexity.

What does *reactive* mean? Separate *when* and *how*. What does it mean? Well that you can separate your app login from view layer(React).

I also learned one new MobX function called [when](https://mobx.js.org/refguide/when.html).

### Flow Typing a React Codebase
[Forbes Lindesay](https://twitter.com/ForbesLindesay) author of [Cabbie](https://github.com/ForbesLindesay/cabbie) a webdriver client for node.js and front end engineer at Facebook has shared his tips on [flow](https://flow.org).

- Mark files for typechecking using `//@flow`.
- Use `any` type only for experiments.
- `mixed` type is fine, it means that return value doesn't matter.
- Use [Exact object types](https://flow.org/en/docs/types/objects/#toc-exact-object-types) it helps you catch some typos.
- Use [read-only properties ](https://flow.org/en/docs/types/interfaces/#toc-covariant-read-only-properties-on-interfaces) by default.

### Party warmup
[Vincent Riemer](https://twitter.com/vincentriemer) author of fully recreated web-based [TR-808](https://io808.com) drum machine and [Bruce Lane](https://twitter.com/batchass) have prepared audiovisual party warmup talk about sound in React.

And at the afterparty I finally managed to meet [Max Stoiber](https://twitter.com/mxstbr). He was mentor of a [Rails Girls SoC team](https://railsgirlssummerofcode.org/blog/2016-10-01-goodbye-xyz) I was coaching.

<p class="post__image-center"><img src="/public/react-amsterdam-party.jpg" alt="" class="post__image"></p>

## Amsterdam and around

This time I was staying at Haarlem. City about 15 minutes by train away from Amsterdam Centraal. The day after conference I decided to take a hike through Nationaal Park Zuid-Kennemerland.

<p class="post__image-center"><img src="/public/react-amsterdam-park.jpg" alt="" class="post__image"></p>

Then I really needed coffee so went back to Screaming Beans coffee. It's lovely small café with tasty coffee.

<p class="post__image-center"><img src="/public/react-amsterdam-screaming-beans.jpg" alt="" class="post__image"></p>
