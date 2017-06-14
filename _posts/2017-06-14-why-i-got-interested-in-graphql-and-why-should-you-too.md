---
layout: post
title: Why I got interested in GraphQL and why should you too
---

GraphQL is query language created by developers in Facebook to make their app development manageable. There are quite a few reasons why should we be interested in it even at small scale.

### Avoid overfetching and underfetching
Let client decide what data it wants. I will use GitHub's GraphQL api as an example. Lets say I want to show number of issues for repositories I recently contributed to.

```graphql
{
  viewer {
    contributedRepositories(first: 10, privacy: PUBLIC, orderBy: {field: UPDATED_AT, direction: DESC}) {
      nodes {
        name
        issues {
          totalCount
        }
      }
    }
  }
}
```

GraphQL looks like JSON without values and GraphQL server fills in the values in shape we requested.

### Formalize writes

Writes and all other changes to the store are called mutations. They are queries with side-effects.

First lets retrieve ID of repository I want to star.
```graphql
query {
  repository(owner: "django", name: "django") {
    id
    name
  }
}
```

Then use the ID as parameter. Plus we can specify what data offered by the mutation we want to receive.

```graphql
mutation {
  addStar(input: {starrableId: "MDEwOlJlcG9zaXRvcnk0MTY0NDgy"}) {
    starrable {
      id
      __typename
    }
  }
}
```

### Avoid implementing data fetching logic

GraphQL has variety of client libraries. Most notable ones are Relay and Apollo. They will help you with linking data from server directly to React components. You can find out more in [Reducing our Redux code with React Apollo](https://dev-blog.apollodata.com/reducing-our-redux-code-with-react-apollo-5091b9de9c2a).

### Great language support

Long gone are the times when GraphQL worked only in Javascript. There are great [libraries](https://github.com/chentsulin/awesome-graphql#lib) in [Python](http://graphene-python.org/), Ruby, .NET and even Haskell.

Want to learn more? Visit [graphql.org/learn/](http://graphql.org/learn/) or checkout my [GraphQL workshop](https://react.coffee/#graphql-a-apollo).
