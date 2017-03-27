---
layout: post
title: On writing and publishing Shopify application
---

Recently I got interested in small and hopefully profitable projects, sometimes called Micropreneurship. Later I found that one of the most "simple" ways to start a SAAS is Shopify application.

Shopify applications are standalone websites integrated with Shopify using their API and doing something that store owners find useful and would be willing to pay for it. The part that makes it easier is that your app will be published on [Shopify app store](https://apps.shopify.com). There's already pretty strong competition but there are still places offering opportunity.

Trying to find a Django project I could work on I realized there is no [Fakturoid.cz](https://www.fakturoid.cz) integration. That was exactly it! Simple integration exposing webhook on one side and creating invoice on the other side.

<p class="post__image-center"><a href="https://apps.shopify.com/fakturoid-invoices?utm_source=blog&utm_medium=article&utm_campaign=writing_app" target="_blank"><img src="/public/shopify-appstore-banner.png" alt="" class="post__image"></a></p>

But how to start? Luckily [Gavin Ballard](http://gavinballard.com) shares quite a lot of his Shopify expertise and even published two packages I found very helpful.

[Django Shopify Auth](https://github.com/discolabs/django-shopify-auth) takes care of all the steps required to retrieve, store and use sellers credentials. Second package is [Django Shopify Webhook](https://github.com/discolabs/django-shopify-webhook) which handles verification of request's authenticity. I [helped](https://github.com/discolabs/django-shopify-webhook/pull/3) to make this package work on Python 3.6.

Shopify maintains [Python client](https://github.com/Shopify/shopify_python_api) for their API. Using these three packages I was able to create basic skeleton pretty fast. There were two roadblocks on the way. I already mentioned the first one. Second one was just overall understanding of how do applications integrate with Shopify. Especially what Embedded app is and how it differs from original integrations.

Embedded apps are supposed to make user feel like he has never left Shopify. There is a great CSS framework called [Shopify Embedded App Frontend Framework](https://github.com/MONEI/Shopify-Embedded-App-Frontend-Framework) which I've used.

It surprised me how everyone I've encountered was happy to help and willing to share their knowledge. From Gavin whose packages simplify app development by a great deal to Philip from Shopify Apps Team who was always kind to me during the approval process.

Are you thinking about building your own Shopify app or want me to build you one? Let me know either way! I will be happy to talk to you. The one I build helps creating invoices needed in Czech republic called [Send invoices to Fakturoid.cz](https://apps.shopify.com/fakturoid-invoices).
