---
layout: post
title: Local state management pomocí Apollo
---

Apollo je sada nástrojů pro práci s GraphQL od tvůrců Meteor.js. Apollo Client slouží k napojení UI na data, Apollo Engine na infrastrukturu a tooling a nakonec Apollo Server pro zpřístupnění REST API pomocí GraphQL.

V tomto článku bych se rád zaměřil na velmi užitečnou část zvanou [apollo-link-state](https://github.com/apollographql/apollo-link-state). Když používám [Apollo Client](https://www.apollographql.com/docs/react/essentials/get-started.html#apollo-boost), tak mám vyřešenou práci s API. V aplikaci je ale vždy nějaký lokální stav, s jehož správou mi Apollo Client může pomoci.

Běžně se pro uchování lokálního stavu používá Redux nebo MobX. S pomocí `apollo-link-state` je nově možné použít cache Apollo Clienta jako úložiště pro všechna naše data. Pro čtení nebo zápis lokálního stavu použijeme queries a mutations jako bychom pracovali s daty na serveru.

Začít ukládat stav pomocí `apollo-link-state` je velmi jednoduché. Jako základ používám [apollo-boost](https://www.apollographql.com/docs/react/essentials/get-started.html#apollo-boost). `apollo-boost` je balíček apollo knihoven, který zahrnuje i `apollo-link-state`. Při inicializaci klienta nastavím `clientState` a vyplním výchozí hodnoty.

{% highlight jsx %}
import ApolloClient from "apollo-boost";

export const client = new ApolloClient({
  uri: "",
  clientState: {
    defaults: { expanded: false },
    resolvers: {}
  }
});
{% endhighlight %}

`clientState` přijímá i parametr `typeDefs`. Toto schéma se nepoužívá pro validaci jako na serveru, protože knihovna `graphql-js` by výrazně zvedla velikost výsledného balíku. Client-side schéma se používá v Apollo DevTools, kde je možné schéma prozkoumat.

Pro čtení dat pak stačí použít GraphQL query s direktivou `@client`. Na stejném principu fungují ostatní Apollo Link knihovny, např. `apollo-link-rest`. Ta používá `@rest` direktivu pro označení polí, která by měla být načtena z REST endpointu. Direktivy `@client` a `@rest` nemění podobu dat, jen určují odkud se data načítají.

{% highlight jsx %}
import { Query } from "react-apollo";
import { gql } from "apollo-boost";

const SECTION_QUERY = gql`
  {
    expanded @client
  }
`;

export const Expander = () => (
  <Query query={SECTION_QUERY}>
    {({ data }) => (data.expanded ? "expanded" : "collapsed")}
  </Query>
);
{% endhighlight %}

Zbýva vyřešit zápis dat. Zde jsou na výběr dvě možnosti. První je zápis přímo do cache použitím `client.writeData` v komponentách `ApolloConsumer`, nebo `Query`. Přímý zápis je vhodný pro jednorázové zápisy nezávislé na datech uložených v cache. Druhá možnost je Mutation, volající client-side resolver. Resolver může pracovat s daty v cache, hodí se tedy např. na přidávání položek do seznamu nebo přepínání hodnoty. Přímý zápis bych přirovnal k `setState`, zatímco resolver připomíná Redux.

{% highlight jsx %}
import { ApolloConsumer } from "react-apollo";

const expand = client =>
  client.writeData({
    data: { expanded: true }
  });

export default = () => (
  <ApolloConsumer>
    {client => <button onClick={() => expand(client)}>See FAQs</button>}
  </ApolloConsumer>
);
{% endhighlight %}

Pokud bych potřeboval použít mutation, musím nejdříve do `ApolloClient` přidat [resolver](https://www.apollographql.com/docs/react/essentials/local-state.html#resolvers).

{% highlight jsx %}
export const client = new ApolloClient({
  uri: "",
  clientState: {
    defaults: {
      expanded: false
    },
    resolvers: {
      Mutation: {
        expand: (_, d, { cache }) => {
          cache.writeData({ data: { expanded: true } });
        }
      }
    }
  }
});
{% endhighlight %}

Potom přidám komponentu `Mutation` a předám jí mutation s direktivou `@client`. To řekne `apollo-link-state`, aby volal lokální `expand` mutation resolver pro zpracování mutation.

{% highlight jsx %}
import { Mutation } from "react-apollo";
const FAQ_SECTION_EXPAND = gql`
  mutation expand {
    expand @client
  }
`;

<Mutation mutation={FAQ_SECTION_EXPAND}>
  {expand => <Button onClick={expand}>See FAQs</Button>}
</Mutation>
{% endhighlight %}

Toto bylo moje základní seznámení s `apollo-link-state`. V jednom z budoucích článků popíšu jak na kombinování lokálního stavu se stavem ze serveru a složitější mutations. Aplikaci používající [přímý zápis](https://codesandbox.io/s/2jvjvllr9y) i [mutations](https://codesandbox.io/s/l4mkk62k39) najdete na CodeSandbox.

### Školení GraphQL a Apollo

Vedu [GraphQL a Apollo workshop](https://rousek.name/skoleni/#graphql-a-apollo), kde si vyzkoušíme Apollo Client v praxi.
