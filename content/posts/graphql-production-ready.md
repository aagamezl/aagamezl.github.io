---
layout: post
title: GraphQL, A Production Ready Tutorial
date: September 30, 2021
author: Álvaro José Agámez Licha
overview: At its core, GraphQL is a language for querying databases from client-side applications. On the backend, GraphQL specifies to the API how to present the data to the client. GraphQL redefines developers’ work with APIs offering more flexibility and speed to market; it improves client-server interactions by enabling the former to make precise data requests and obtain no more and no less, but exactly what they need.
permalink: graphql-production-ready.html
tags:
- Node.js
- GraphQL
- Software Development
- API
---

At its core, GraphQL is a language for querying databases from client-side applications. On the backend, GraphQL specifies to the API how to present the data to the client. GraphQL redefines developers’ work with APIs offering more flexibility and speed to market; it improves client-server interactions by enabling the former to make precise data requests and obtain no more and no less, but exactly what they need.

GraphQL is a query language created by Facebook in 2012 and open-sourced in 2015. It aims to provide an alternative to traditional REST API architecture. At its core, GraphQL provides a syntax that describes how to ask for data (called a Schema). GraphQL is database agnostic and works by creating a single endpoint responsible for accepting queries, rather than relying on the REST API approach of having separate endpoints for each service.

# Advantages of GraphQL

GraphQL offers many benefits over REST APIs. One of the main benefits is clients have the ability to dictate exactly what they need from the server, and receive that data in a predictable way.

For example, imagine you have the following schema:

```json
type Query {
  me: User
}

type User {
  id: ID
  name: String
  city: String
  state: String
  friends: [User]
}
```

Now let’s say that you just needed to get a `User`‘s name. If you had a REST API, you would typically receive the entire `User` object back in your response. This leads to overfetching and can result in performance issues. With GraphQL, you only receive the data you explicitly request.

Here’s what our `User` name query would look like:

```json
{
  me {
    name
  }
}
```

And here’s what our response would look like:

```json
{
  "me": {
    "name": "John Doe"
  }
}
```

# Disadvantages of GraphQL

While GraphQL has many advantages over traditional REST APIs, there are several key disadvantages as well.

One disadvantage is that queries always return a HTTP status code of 200, regardless of whether or not that query was successful. If your query is unsuccessful, your response JSON will have a top-level errors key with associated error messages and stacktrace. This can make it much more difficult to do error handling and can lead to additional complexity for things like monitoring.

Another disadvantage is the lack of built-in caching support. Because REST APIs have multiple endpoints, they can leverage native HTTP caching to avoid refetching resources. With GraphQL, you will need to setup your own caching support which means relying on another library, or setting up something like globally unique IDs for your backend.

This leads us to the final disadvantage: complexity. If you have a simple REST API and deal with data that is relatively consistent over time, you would be better off sticking with your REST API. For companies that deal with rapidly-changing data, and have the engineering resources to devote to rearchitecting their API platforms, GraphQL can solve many of the pain points experienced with REST APIs.

GraphQL provides an interesting solution to common hurdles faced when using REST APIs. While using GraphQL has several downsides, if you find yourself working with rapidly-changing data at scale, it could be a great solution for your business. To learn more about, check out the docs. And if you’re looking for a more fully-fledged GraphQL solution, Apollo provides an easy way to get started on both the client and server-side.