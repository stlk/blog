---
layout: post
title: Next.js - server side rendering bez práce
excerpt: Next.js je minimalistický framework pro tvorbu React applikací s důrazem na jednoduchost server-side renderingu. Ukážeme si, jak nám pomůže vytvořit aplikaci, která bude přátelská jak pro uživatele, tak i pro vyhledávače.
image: http://blog.rousek.name/public/nextjs-workshop.png
---

Skvělý framework [Next.js](https://github.com/zeit/next.js) se dočkal už čtvrté verze. Oproti verzi 2, kterou jsem zmiňoval minule je výrazně rychlejší a má několik velmi zajímavých novinek.

## Co je Next.js

Next.js je minimalistický framework pro tvorbu React applikací s důrazem na jednoduchost server-side renderingu. Bere na starost pouze server-side rendering, který zvládá dokonale a zbytek je opět na nás.

Jak Next.js zapadá do současného trendu vývoje? Pro mě byl vždy problém nedosažitelnost server-side renderingu na začínajících projektech a s tím související absence SEO. Místo toho, abych aplikaci poskládal v komponentách, musel jsem pracovat s dalším šablonovým jazykem a globálními styly v Django nebo Rails aplikaci.

Frameworky jako Next.js a služby poskytující databázi s GraphQL API umožnily nový způsob prototypování nebo nastartování projektu. Vytvořit velmi jednoduchou aplikaci a napojit ji na [Graph.cool](https://www.graph.cool/). Na GraphQL se ale podíváme později.

Základní nastavení je popsáno v původním článku [Next.js - server side rendering for masses](http://blog.rousek.name/2017/06/19/nextjs-server-side-rendering-for-masses/), teď se budu soustředit pouze na novinky.

### Dynamic Import

[TC39 dynamic import](https://github.com/tc39/proposal-dynamic-import) je návrh způsobu jak odsunout načtení modulu na pozdější chvíli. V následujícím příkladu Next.js automaticky oddělí knihovnu moment.js do samostatného souboru, který načte až po uplynutí `setTimeout`. Díky tomu, můžeme zmenšit velikost hlavní aplikace.

{% highlight js %}
const moment = import('moment')
setTimeout(function() {
  moment.then(moment => {
    // Do something with moment
  })
}, 15000)
{% endhighlight %}

### Dynamic React Components

Dynamické importy umožňují importovat i komponenty. To nám umožní vyrenderovat na serveru jen důležitý obsah a obsah, který není zásadní pro vyhledávače, jako formuláře nebo infografiku stáhnout později.

{% highlight js %}
import dynamic from 'next/dynamic'

const DynamicForm = dynamic(import('../components/Form'))

export default () =>
  <div>
    <Header />
    <p>Content is here!</p>
    <DynamicForm />
  </div>
{% endhighlight %}

## Školení Next.js

Pořádám [Next.js workshop](https://react.coffee/#react-na-serveru), kde si vyzkoušíme Next.js v praxi.
