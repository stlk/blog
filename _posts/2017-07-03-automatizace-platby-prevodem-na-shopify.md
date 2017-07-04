---
layout: post
title: Automatizace platby převodem na Shopify pomocí Fakturoidu
---

Můj klient potřeboval automatizovat platbu převodem v obchodě na platformě Shopify. Vymysleli jsme řešení, jak pomocí proformy využít schopnosti [Fakturoid.cz](https://www.fakturoid.cz). Fakturoid při zaplacení proformy vytvoří fakturu a následně informuje o jejím vytvoření.

#### Ukážeme si jak automaticky:

- poslat zákazníkovi platební informace
- po zaplacení odeslat e-mail s fakturou
- označit objednávku jako zaplacenou

## Jak takový proces nastavit?

### Základní propojení

Do vašeho obchodu si nainstalujte aplikaci <a href="https://apps.shopify.com/fakturoid-invoices?utm_source=blog&utm_medium=article&utm_campaign=proforma_guide" target="_blank">Send invoices to Fakturoid.cz</a>. V sekci *Apps* se objeví aplikace *Send invoices to Fakturoid.cz*.

Nejprve je nutné vyplnit přihlašovací údaje pro Fakturoid v sekci *Fakturoid credentials*. [<a href="https://vimeo.com/224252284" target="_blank">videonávod</a>] A následně klinutím na *Enable proforma invoices* v sekci *Proforma invoices* zapnout vytváření proforma faktur při vytvoření objednávky.

<a href="/public/enable-proforma-invoices.png" target="_blank">
  <img src="/public/enable-proforma-invoices.png" alt="" class="post__image post__image-full-width">
</a>

Po zapnutí je ještě nutné nastavit *webhook* ve Fakturoidu. [<a href="https://vimeo.com/224252723" target="_blank">videonávod</a>] Webhook je způsob, jakým Fakturoid informuje aplikaci o zaplacené faktuře.

### Platební údaje v souhrnu objednávky

Zbývá najít způsob, jak dostat platební údaje k zákazníkovi. K tomuto účelu je možné využít notifikační e-mail *Order confirmation*. Proformy mají jako variabilní symbol nastavené číslo objednávky.

> Návod předpokládá, že máte v sekci *Settings > Payments > Manual payments* zapnutou platbu s názvem *Bank Deposit*.

<a href="/public/order-confirmation.png" target="_blank">
  <img src="/public/order-confirmation.png" alt="" class="post__image post__image-full-width">
</a>

V šabloně najděte sekci `<h4>Payment method</h4>`, konkrétně `{% raw %}{{ transaction.gateway | replace: "_", " "...{% endraw %}`. Za `{% raw %}{% endif %}{% endraw %}` po tomto výrazu vložte kód uvozený `<!-- Platební informace -->` a `<!-- / Platební informace -->` dle příkladu.

{% highlight liquid %}
{% raw %}
<td class="customer-info__item">
  <h4>Payment method</h4>
  ...
    {% else %}
      {{ transaction.gateway | replace: "_", " " | capitalize }} — <strong>{{ transaction.amount | money }}</strong>
    {% endif %}
    <!-- Platební informace -->
    {% if transaction.gateway == "Bank Deposit" %}
      <p class="customer-info__item-content">
        <strong>Číslo účtu:</strong> 123457/0000
      </p>
      <p class="customer-info__item-content">
        <strong>Variabilní symbol:</strong> {{ name }}
      </p>
    {% endif %}
    <!-- / Platební informace -->
{% endraw %}
{% endhighlight %}

### Doručení faktury zákazníkovi

Fakturoid umožňuje automatické odeslání faktury zákazníkovi po zaplacení proformy. V sekci *Nastavení > Emaily > Texty a způsob odesílání* je záložka *Zaplacení proformy*. Tuto záložku nastavte dle obrázku.

<a href="/public/fakturoid-nastaveni-emailu.png" target="_blank">
  <img src="/public/fakturoid-nastaveni-emailu.png" alt="" class="post__image post__image-full-width">
</a>

Potřebujete pomoct s nastavením? Neváhejte mě kontaktovat na e-mailu josef@rousek.name.
