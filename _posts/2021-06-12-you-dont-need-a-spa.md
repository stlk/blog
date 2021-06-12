---
layout: post
title: You don't need a SPA
---

I'm building Shopify apps using Django and React. When I first created [Candy Rack](https://apps.shopify.com/candyrack) it was server rendered HTML with single or multiple React components on a page. Since then the web ecosystem evolved. The most important change are the restrictions around 3rd party cookies. To address this problem Shopify came with an alternative authentication method that doesn't rely on cookies called [session tokens](https://shopify.dev/tutorials/authenticate-your-app-using-session-tokens). Instead of cookies Shopify provides a Javascript function that returns a token. As a result all authenticated requests must be made from Javascript. While SPA is fine for larger apps and teams, we have some smaller apps and rewriting them to SPA isn't viable.

Thankfully a whole new approach to interactive apps is starting to get a widespread attention.

The one I have some experience with is [Hotwire](https://hotwire.dev/), successor of Turbolinks. I created an [example Shopify app](https://github.com/digismoothie/django-session-token-auth-demo) based on Shopify's tutorial [Authenticate server-side rendered embedded apps using Rails and Turbolinks](https://shopify.dev/tutorials/authenticate-server-side-rendered-embedded-apps-using-rails-and-turbolinks). Hotwire works well, gets the job done. But there are two other tools I would love to try.

[htmx](https://htmx.org/) is similar to Hotwire and I'm seeing more of it in Python community.

[Inertia.js](https://inertiajs.com/) is taking slightly different approach. It serves as client-side router for React, Vue and Svelte apps. The most interesting feature is that Inertia uses a [protocol](https://inertiajs.com/the-protocol) that defines how are data passed from the server and based on this data it renders a component. If you would like to learn more I recommend Episode [291: All Things Inertia.js with Jonathan Reinink](https://www.bikeshed.fm/291) of The Bike Shed podcast.

I'm very excited about the direction these tools are going and will keep writing about my experience with them.


