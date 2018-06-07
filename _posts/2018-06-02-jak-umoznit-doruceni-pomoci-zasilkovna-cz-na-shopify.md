---
layout: post
title: Jak umožnit doručení pomocí Zásilkovna.cz na Shopify
---

Klient mě oslovil se záměrem poskytovat doručování pomocí Zásilkovna.cz na Shopify. Vzhledem k omezeným možnostem ovlivnit checkout proces u levnějších Shopify variant se to ukázalo jako cvičení tvořivosti.

> Úpravu checkout procesu umožňuje pouze enterprise varianta [Shopify Plus](https://www.shopify.com/plus).

Jak je vlastně možné ovlivnit checkout proces na Shopify? Poslední místo, kde je možné zasahovat do HTML je v košíku. V checkout procesu je možné měnit pouze texty. Dalším pomocníkem je [CarrierService](https://help.shopify.com/api/reference/shipping_and_fulfillment/carrierservice). Ta umožňuje ovlivnit nabídku způsobu doručení na základě obsahu košíku.

Ustálené řešení jak nabídnout výběr z velkého množství výdejních míst je následující.

1. V košíku umožnit výběr výdejního místa.
2. Doručovací adresu použít jako adresu vyzvednutí.
3. Pokud bylo vybráno výdejní místo, v seznamu způsobů doručení se zobrazí pouze Zásilkovna v opačném případě bežné způsoby doručení.

Ukázku checkout procesu s výběrem výdejního místa můžete vidět na následujícím videu.

<p class="post__image-center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/ZX39HoaZcrM?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</p>

Chcete také nabízet doručení na výdejní místa Zásilkovna.cz na svém e-shopu? Nebo chcete přesně vědět jako to funguje? Neváhejte mě kontaktovat.

### Zajímá vás Shopify?

Přihlašte se a budete dostávat tipy a novinky související s provozováním e-shopu na Shopify.

<form action="https://www.getdrip.com/forms/572224451/submissions" method="post" data-drip-embedded-form="572224451">
  <div>
      <label for="drip-email">E-mail</label><br />
      <input type="email" id="drip-email" name="fields[email]" value="" />
  </div>
  <input type="submit" value="Přihlásit k odběru" data-drip-attribute="sign-up-button" />
</form>
