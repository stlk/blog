---
layout: post
title: Statically generated e-commerce using Shopify and Netlify
---

I recently fell in love with Netlify because of its simplicity and speed. Nelify allows you to build and host your website in one place with generous pricing levels. I wanted to explore what are possibilities when it comes to e-commerce so I decided to create a simple statically generated website where I could sell some products.

As a Shopify fan, I will try to use Shopify to manage my products and orders. Shopify offers a pricing plan called [Shopify Lite](https://www.shopify.com/lite) for $9 per month. Most notable features of this plan are Buy Button and Facebook Messenger sales channels.

I will use their GraphQL API to retrieve the products. With data from Shopify, I will generate static website containing product list and [BuyButton.js](https://github.com/Shopify/buy-button-js) code. Let's see how it fits together.

## Setting up

Let's start by creating Store in Shopify. Then add new sales channel called Buy Button. Now you need to find API token so you can use the API. To get there select a product and then click *Generate code*. In the generated script find and copy `domain` and `apiKey`.

<a href="/public/shopify_buy_button_code.png" target="_blank">
  <img src="/public/shopify_buy_button_code.png" alt="" class="post__image post__image-full-width">
</a>

With the credentials from the script, we can focus on Netlify. First fork my project from [github.com/stlk/shopify-buy-button](https://github.com/stlk/shopify-buy-button) so you can add it to your Netlify account. In the Netlify *site Settings* go to *Build & deploy* and the to *Build environment variables* section. While in there fill in `SHOPIFY_SHOP_DOMAIN` (only the part before `.myshopify.com`) and `SHOPIFY_STOREFRONT_TOKEN`.

<a href="/public/netlify_env_variables.png" target="_blank">
  <img src="/public/netlify_env_variables.png" alt="" class="post__image post__image-full-width">
</a>

You can now check Deploys section to see if the build succeeded. If not, try rebuilding using *Trigger deploy*. After the build is published you can visit your e-shop!

<p class="post__image-center">
  <img src="/public/checkout.gif" alt="" class="post__image post__image-full-width">
</p>

But do not rejoice! You still have one last step to make Netlify rebuild the site after you change or create the product in Shopify. 

This is just the right job for webhooks. In our favourite section *Build & deploy* in Netlify's Settings create a new *Build hook*. Keep the branch setting on master. This creates a unique URL that will cause your site to rebuild when POST request hits. We will use this URL in Shopify.

<a href="/public/netlify_webhooks.png" target="_blank">
  <img src="/public/netlify_webhooks.png" alt="" class="post__image post__image-full-width">
</a>

In your Shopify Store administration go to *Settings* > *Notifications* and scroll down to *Webhooks*. In there create three webhooks for `Product creation`, `Product update` and `Product deletion` point all three to the *Build hook* we just created. Don't the format setting as we aren't reading any data from it.

<a href="/public/shopify_webhooks.png" target="_blank">
  <img src="/public/shopify_webhooks.png" alt="" class="post__image post__image-full-width">
</a>

Hope this helped you explore unusual take on building e-commerce site. If you want to find out more about internals, keep reading.

## How did we get the data from Shopify

The script uses [Storefront API GraphQL](https://help.shopify.com/api/storefront-api/getting-started#accessing-the-storefront-api-graphql-endpoint) which lets us to specify exactly what data we need. You can play around with data from example shop using [GraphiQL](https://help.shopify.com/api/storefront-api/graphql-explorer/graphiql).

```
{
  shop {
    name
    products(first: 10) {
      edges {
        node {
          id
          title
          descriptionHtml
          images(first: 1) {
            edges {
              node {
                originalSrc
              }
            }
          }
        }
      }
    }
  }
}
```

You can see it in action in `build.py` on [GitHub](https://github.com/stlk/shopify-buy-button/blob/master/build.py#L25).