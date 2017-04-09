# GraphQL Combine Resolvers

A library to simplify composition of GraphQL [resolvers](http://graphql.org/learn/execution/).

![Build status](https://travis-ci.org/lucasconstantino/graphql-combine-resolvers.svg?branch=master)

## Installation

This package is available on [npm](https://www.npmjs.com/package/graphql-combine-resolvers) as: *graphql-combine-resolvers*

```
npm install graphql-combine-resolvers
```

> You should consider using [yarn](https://yarnpkg.com/), though.

## Motivation

Many times we end-up repeating lots of logic on our resolvers. Access control, for instance, is something that can be done in the resolver level but just tends to end up with repeated code, even when creating services for such a task. This package solves it in a very simple and functional style.

## How to use it

This library consists solely of a helper function for combining functions in a first-result-returns manner. GraphQL resolvers is just one kind of thing to benefit from this helper. Here is an example usage with [resolver maps](http://dev.apollodata.com/tools/graphql-tools/resolvers.html):

```
import { graphql } from 'graphql'
import { makeExecutableSchema } from 'graphql-tools'
import { skip, combineResolvers } from 'graphql-combine-resolvers'

const typeDefs = `
  type Query {
    sensitive: String!
  }

  schema {
    query: Query
  }
`

/**
 * Sample resolver which returns an error in case no user
 * is available in the provided context.
 */
const isAuthenticated = (root, args, { user }) => user ? skip : new Error('Not authenticated')

/**
 * Sample resolver which returns an error in case user
 * is not admin.
 */
const isAdmin = combineResolvers(
  isAuthenticated,
  (root, args, { user: { role } }) => role === 'admin' ? skip : new Error('Not authorized')
)

/**
 * Sample sensitive information resolver, for admins only.
 */
const sensitive = combineResolvers(
  isAdmin,
  (root, args, { user: { name } }) => 'shhhh!'
)

// Resolver map
const resolvers = { Query: { sensitive } }

const schema = makeExecutableSchema({ typeDefs, resolvers })

// Resolves with a "Non authenticated" error.
graphql(schema, '{ sensitive }', null, { }).then(console.log)

// Resolves with a "Not authorized" error.
graphql(schema, '{ sensitive }', null, { user: { role: 'some-role' } }).then(console.log)

// Resolves with a sensitive field containing "shhhh!".
graphql(schema, '{ sensitive }', null, { user: { role: 'admin' } }).then(console.log)
```
