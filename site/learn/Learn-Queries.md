---
title: 查询和修改
layout: ../_core/DocsLayout
category: Learn
permalink: /learn/queries/
next: /learn/schema/
sublinks: 字段,参数,别名,片段,变量,操作名,指令,转变,行内片段
---

在这一章节您将会学习如何向 GraphQL 服务器请求查询.

## 字段

在 GraphQL 上最简单查询对象字段的fangfa. 我们先来看一下这个简单的查询和运行得到的结果:

```graphql
# { "graphiql": true }
{
  hero {
    name
  }
}
```

你可以立即看到查询与结果的形式完全相同. 这对于 GraphQL 是非常重要的, 因为你总是希望获得你所期待的内容, 并且服务器端明确知道客户端请求的具体字段.

`name`  字段返回 `String` 类型, 在这个示例里,是星球大战的英雄的名字 `"R2-D2"`.

> 对了,还有一件事: 上面的查询是*互动的*. 意味着你可以随意修改它并看到更新的结果. 尝试在 `hero` 对象中加入 `appearsIn` 字段的查询,并查看更新的结果.

在上面的例子里, 我们只是要求返回字符串格式的英雄名字, 字段也可以直接饮用对象. 在这样的情况下, 你可以用 *子选择* 来约束该对象的字段. GraphQL 查询可以遍历相关对象及其字段, 让客户端在单一请求里获取更多的相关数据, 从而替代传统 REST 架构那样经过多次的服务器往返.

```graphql
# { "graphiql": true }
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

请注意,在这个示例中, `friends` 字段返回的是一个数据的数组. GraphQL 查询对于单个项目或者项目列表看起来相同, 然而我们可以根据结构里指出的内容知道预期的结果.

## 参数

If the only thing we could do was traverse objects and their fields, GraphQL would already be a very useful language for data fetching. But when you add the ability to pass arguments to fields, things get much more interesting.

```graphql
# { "graphiql": true }
{
  human(id: "1000") {
    name
    height
  }
}
```

In a system like REST, you can only pass a single set of arguments - the query parameters and URL segments in your request. But in GraphQL, every field and nested object can get its own set of arguments, making GraphQL a complete replacement for making multiple API fetches. You can even pass arguments into scalar fields, to implement data transformations once on the server, instead of on every client separately.

```graphql
# { "graphiql": true }
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

Arguments can be of many different types. In the above example, we have used an Enumeration type, which represents one of a finite set of options (in this case, units of length, either `METER` or `FOOT`). GraphQL comes with a default set of types, but a GraphQL server can also declare its own custom types, as long as they can be serialized into your transport format.

[Read more about the GraphQL type system here.](/learn/schema)


## 别名

If you have a sharp eye, you may have noticed that, since the result object fields match the name of the field in the query but don't include arguments, you can't directly query for the same field with different arguments. That's why you need _aliases_ - they let you rename the result of a field to anything you want.

```graphql
# { "graphiql": true }
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

In the above example, the two `hero` fields would have conflicted, but since we can alias them to different names, we can get both results in one request.


## 片段

Let's say we had a relatively complicated page in our app, which let us look at two heroes side by side, along with their friends. You can imagine that such a query could quickly get complicated, because we would need to repeat the fields at least twice - one for each side of the comparison.

That's why GraphQL includes reusable units called _fragments_. Fragments let you construct sets of fields, and then include them in queries where you need to. Here's an example of how you could solve the above situation using fragments:

```graphql
# { "graphiql": true }
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

You can see how the above query would be pretty repetitive if the fields were repeated. The concept of fragments is frequently used to split complicated application data requirements into smaller chunks, especially when you need to combine lots of UI components with different fragments into one initial data fetch.


## 变量

So far, we have been writing all of our arguments inside the query string. But in most applications, the arguments to fields will be dynamic: For example, there might be a dropdown that lets you select which Star Wars episode you are interested in, or a search field, or a set of filters.

It wouldn't be a good idea to pass these dynamic arguments directly in the query string, because then our client-side code would need to dynamically manipulate the query string at runtime, and serialize it into a GraphQL-specific format. Instead, GraphQL has a first-class way to factor dynamic values out of the query, and pass them as a separate dictionary. These values are called _variables_.

When we start working with variables, we need to do three things:

1. Replace the static value in the query with `$variableName`
2. Declare `$variableName` as one of the variables accepted by the query
3. Pass `variableName: value` in the separate, transport-specific (usually JSON) variables dictionary

Here's what it looks like all together:

```graphql
# { "graphiql": true, "variables": { "episode": "JEDI" } }
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

Now, in our client code, we can simply pass a different variable rather than needing to construct an entirely new query. This is also in general a good practice for denoting which arguments in our query are expected to be dynamic - we should never be doing string interpolation to construct queries from user-supplied values.


### Variable definitions

The variable definitions are the part that looks like `($episode: Episode)` in the query above. It works just like the argument definitions for a function in a typed language. It lists all of the variables, prefixed by `$`, followed by their type, in this case `Episode`.

All declared variables must be either scalars, enums, or input object types. So if you want to pass a complex object into a field, you need to know what input type that matches on the server. Learn more about input object types on the Schema page.

Variable definitions can be optional or required. In the case above, since there isn't an `!` next to the `Episode` type, it's optional. But if the field you are passing the variable into requires a non-null argument, then the variable has to be required as well.

To learn more about the syntax for these variable definitions, it's useful to learn the GraphQL schema language. The schema language is explained in detail on the Schema page.


### Default variables

Default values can also be assigned to the variables in the query by adding the default value after the type declaration. 

```graphql
query HeroNameAndFriends($episode: Episode = "JEDI") {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

When default values are provided for all variables, you can call the query without passing any variables. If any variables are passed as part of the variables dictionary, they will override the defaults. 

## 操作名

One thing we also saw in the example above is that our query has acquired an _operation name_. Up until now, we have been using a shorthand syntax where we omit both the `query` keyword and the query name, but in production apps it's useful to use these to make our code less ambiguous.

Think of this just like a function name in your favorite programming language. For example, in JavaScript we can easily work only with anonymous functions, but when we give a function a name, it's easier to track it down, debug our code, and log when it's called. In the same way, GraphQL query and mutation names, along with fragment names, can be a useful debugging tool on the server side to identify different GraphQL requests.


## 指令

We discussed above how variables enable us to avoid doing manual string interpolation to construct dynamic queries. Passing variables in arguments solves a pretty big class of these problems, but we might also need a way to dynamically change the structure and shape of our queries using variables. For example, we can imagine a UI component that has a summarized and detailed view, where one includes more fields than the other.

Let's construct a query for such a component:

```graphql
# { "graphiql": true, "variables": { "episode": "JEDI", "withFriends": false } }
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

Try editing the variables above to instead pass `true` for `withFriends`, and see how the result changes.

We needed to use a new feature in GraphQL called a _directive_. A directive can be attached to a field or fragment inclusion, and can affect execution of the query in any way the server desires. The core GraphQL specification includes exactly two directives, which must be supported by any spec-compliant GraphQL server implementation:

- `@include(if: Boolean)` Only include this field in the result if the argument is `true`.
- `@skip(if: Boolean)` Skip this field if the argument is `true`.

Directives can be useful to get out of situations where you otherwise would need to do string manipulation to add and remove fields in your query. Server implementations may also add experimental features by defining completely new directives.


## 转变

Most discussions of GraphQL focus on data fetching, but any complete data platform needs a way to modify server-side data as well.

In REST, any request might end up causing some side-effects on the server, but by convention it's suggested that one doesn't use `GET` requests to modify data. GraphQL is similar - technically any query could be implemented to cause a data write. However, it's useful to establish a convention that any operations that cause writes should be sent explicitly via a mutation.

Just like in queries, if the mutation field returns an object type, you can ask for nested fields. This can be useful for fetching the new state of an object after an update. Let's look at a simple example mutation:

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI", "review": { "stars": 5, "commentary": "This is a great movie!" } } }
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

Note how `createReview` field returns the `stars` and `commentary` fields of the newly created review. This is especially useful when mutating existing data, for example, when incrementing a field, since we can mutate and query the new value of the field with one request.

You might also notice that, in this example, the `review` variable we passed in is not a scalar. It's an _input object type_, a special kind of object type that can be passed in as an argument. Learn more about input types on the Schema page.

### Multiple fields in mutations

A mutation can contain multiple fields, just like a query. There's one important distinction between queries and mutations, other than the name:

**While query fields are executed in parallel, mutation fields run in series, one after the other.**

This means that if we send two `incrementCredits` mutations in one request, the first is guaranteed to finish before the second begins, ensuring that we don't end up with a race condition with ourselves.


## 行内片段

Like many other type systems, GraphQL schemas include the ability to define interfaces and union types. [Learn about them in the schema guide.](/learn/schema/#interfaces)

If you are querying a field that returns an interface or a union type, you will need to use *inline fragments* to access data on the underlying concrete type. It's easiest to see with an example:

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI" } }
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}
```

In this query, the `hero` field returns the type `Character`, which might be either a `Human` or a `Droid` depending on the `episode` argument. In the direct selection, you can only ask for fields that exist on the `Character` interface, such as `name`.

To ask for a field on the concrete type, you need to use an _inline fragment_ with a type condition. Because the first fragment is labeled as `... on Droid`, the `primaryFunction` field will only be executed if the `Character` returned from `hero` is of the `Droid` type. Similarly for the `height` field for the `Human` type.

Named fragments can also be used in the same way, since a named fragment always has a type attached.


### Meta fields

Given that there are some situations where you don't know what type you'll get back from the GraphQL service, you need some way to determine how to handle that data on the client. GraphQL allows you to request `__typename`, a meta field, at any point in a query to get the name of the object type at that point.

```graphql
# { "graphiql": true}
{
  search(text: "an") {
    __typename
    ... on Human {
      name
    }
    ... on Droid {
      name
    }
    ... on Starship {
      name
    }
  }
}
```

In the above query, `search` returns a union type that can be one of three options. It would be impossible to tell apart the different types from the client without the `__typename` field.

GraphQL services provide a few meta fields, the rest of which are used to expose the [Introspection](../introspection/) system.

