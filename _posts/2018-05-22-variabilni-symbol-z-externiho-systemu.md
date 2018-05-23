---
layout: post
title: "Shopify - Fakturoid: Variabilní symbol z externího systému"
excerpt: Máte systém na zpracování objednávek generující vlastní čísla objednávek a potřebujete použít toto číslo jako variabilní symbol?
image: http://blog.rousek.name/public/variable-symbol-settings.png
---

> Tento článek se vztahuje k integraci propojující [Shopify a Fakturoid.cz](https://apps.shopify.com/fakturoid-invoices).

Máte systém na zpracování objednávek generující vlastní čísla objednávek a potřebujete použít toto číslo jako variabilní symbol? Nově je možné použít atributy uložené v `note_attributes` jako variabilní symbol na faktuře. Tento atribut má několik názvů v závislosti na místě použití.

Celý proces začíná v nákupním košíku, kde je možné pomocí skrytého `input` elementu nastavit [Cart attribute][1].

{% highlight html %}
{% raw %}
<input type="hidden" name="attributes[shopify_fakturoid_variable_symbol]" value="{{ cart.attributes["shopify_fakturoid_variable_symbol"] }}">
{% endraw %}
{% endhighlight %}

Pro složitější aplikace je také možné použít [Shopify Ajax API][2].

{% highlight js %}
jQuery.post('/cart/update.js', {
  attributes: {
    shopify_fakturoid_variable_symbol: '123456789'
  }
})
{% endhighlight %}

Tento atribut se zobrazí v sekci **Additional details** v detailu objednávky v administraci Shopify.

<p class="post__image-center">
  <img src="/public/variable-symbol-aditional-details.png" alt="" class="post__image post__image-full-width">
</p>

Posledním krokem je zobrazení instrukcí k platbě. K tomuto účelu je možné využít notifikační e-mail *Order confirmation*.

> Návod předpokládá, že máte v sekci *Settings > Payments > Manual payments* zapnutou platbu s názvem *Bank Deposit*.

<a href="/public/order-confirmation.png" target="_blank">
  <img src="/public/order-confirmation.png" alt="" class="post__image post__image-full-width">
</a>

V šabloně najděte sekci `<h4>Payment method</h4>`, konkrétně `{% raw %}{{ transaction.gateway | replace: "_", " "...{% endraw %}`. Za `{% raw %}{% endif %}{% endraw %}` po tomto výrazu vložte kód uvozený `<!-- Platební informace -->` a `<!-- / Platební informace -->` dle příkladu.

{% highlight liquid %}
{% raw %}
<!-- Platební informace -->
{% if transaction.gateway == "Bank Deposit" %}
<p class="customer-info__item-content">
    <strong>Číslo účtu:</strong> 123457/0000
</p>
<p class="customer-info__item-content">
    <strong>Variabilní symbol:</strong>
    {% if order.attributes %}
        {% for attribute in order.attributes %}
            {% if attribute | first == "shopify_fakturoid_variable_symbol" %}
                {{ attribute | last }}
            {% endif %}
        {% endfor %}
    {% endif %}
</p>
{% endif %}
<!-- / Platební informace -->
{% endraw %}
{% endhighlight %}

### Nastavení aplikace

Pro aktivaci této funkce stačí nastavit název atributu v sekci **Custom Variable symbol**.

<p class="post__image-center">
  <img src="/public/variable-symbol-settings.png" alt="" class="post__image post__image-full-width">
</p>

Více o automatizaci platby převodem se dozvíte v článku [Automatizace platby převodem na Shopify pomocí Fakturoidu][4].

[1]: https://help.shopify.com/themes/customization/cart/get-more-information-with-cart-attributes#sectioned-themes
[2]: https://help.shopify.com/themes/development/getting-started/using-ajax-api#update-cart
[3]: https://help.shopify.com/themes/liquid/objects/order#order-attributes
[4]: {% post_url 2017-07-03-automatizace-platby-prevodem-na-shopify %}