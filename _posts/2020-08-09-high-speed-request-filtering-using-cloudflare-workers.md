---
layout: post
title: Hi-speed request filtering using Cloudflare Workers
---

Recently I was facing very interesting challenge. We distribute our app's javascript using Cloudflare. Sometimes the `<script>` tag stays on the site after the app is uninstalled resulting in unnecessary load on our server. This means that eventually we'll have to quickly determine whenever should the script run or not.

Quite a few CDN providers now support running our code on CDN edge. I'm using it to return empty JS file for clients that uninstalled our app. Based on URL `https://my-awesome-app.com/static/main.js?shop=example-client.com` I can use Cloudflare Workers and KV store to make this decision in around 1ms.

I wrote following worker and assigned it to `my-awesome-app.com/static/main.js*`.

```js
async function getBlockedHostname(myshopify_domain) {
  if (!myshopify_domain) {
    return
  }

  return await KV_NAMESPACE.get(myshopify_domain);
}

async function handleRequest(request) {
  let url = new URL(request.url)
  let myshopify_domain = url.searchParams.get("shop")

  let blocked_hostname = await getBlockedHostname(myshopify_domain)
  // if KV contains key myshopify_domain we'll return empty response instead 
  if (blocked_hostname) {
    return new Response("",
      {
        headers: {
          'content-type': 'application/javascript',
        }
      })
  }

  // else return regular response
  return fetch(request)
}

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})
```