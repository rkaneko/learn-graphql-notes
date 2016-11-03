Query Variables
===

queryを組み立てるとき、引数をよく使う。

``` graphql
{
  recentPosts(count: 10) {
    title
  }
}
```

引数を変更したい場合(例えばページングサイズを変えたい等)、実世界のアプリで管理が大変になりうる。

また、サーバサイドから取得する`userId`のようなものはそもそもqueryの引数として使用できない。

GraphQLでは、このようなケースにおいて**query variables**の利用を検討する。

---

# Named queries

これまでは、queryの簡易表現で現してきたが、フルで書くと以下のようになる。


```graphql
query getFewPosts {
  recentPosts(count: 10) {
    title
  }
}
```

必要なら、queryに名前を与えることもできる。

---

# Using query variables

`recentPosts`の引数としての`count`にquery variableを利用することを見る。

```graphql
query getFewPosts($postCount: Int!) {
  recentPosts(count: $postCount) {
    title
  }
}
```

## How to input query variables

```graphql
{
  "postCount": 2
}
```

上のような設定をサーバでもたせることができる。

learn GraphQLのsandboxではQUERY VARIABLESタブで設定する。

もし、`postCount`を設定していなければ、

```json
{
  "errors": [
    {
      "message": "Variable \"$postCount\" of required type \"Int!\" was not provided."
    }
  ]
}
```

のようなレスポンスが返る。

---

# Use query variables anywhere

一度query variablesを定義すればどこでもその値を使用することができる。


query

```graphql
query getFewPosts($postCount: Int!, $commentCount: Int) {
  recentPosts(count: $postCount) {
    title,
    ...comments
  }
}

fragment comments on Post {
  comments(limit: $commentCount) {
    content
  }
}
```

query variales

```json
{
  "postCount": 2,
  "commentCount": 3
}
```

結果は、

```json
{
  "data": {
    "recentPosts": [
      {
        "title": "New Feature: Tracking Error Status with Kadira",
        "comments": [
          {
            "content": "This is a very good blog post"
          },
          {
            "content": "Keep up the good work"
          }
        ]
      },
      {
        "title": "Understanding Mean, Histogram and Percentiles",
        "comments": [
          {
            "content": "This is a very good blog post"
          },
          {
            "content": "Keep up the good work"
          }
        ]
      }
    ]
  }
}{
  "data": {
    "recentPosts": [
      {
        "title": "New Feature: Tracking Error Status with Kadira",
        "comments": [
          {
            "content": "This is a very good blog post"
          },
          {
            "content": "Keep up the good work"
          }
        ]
      },
      {
        "title": "Understanding Mean, Histogram and Percentiles",
        "comments": [
          {
            "content": "This is a very good blog post"
          },
          {
            "content": "Keep up the good work"
          }
        ]
      }
    ]
  }
}
```

---

# Input types

GraphQL schemaにおいて、いくつかのbuild-inの型(e.g. `Int`, `String`)と、カスタム型(e.g. `Post`, `Author`)がある。

**query variablesの型として使用できるのは、次の型のみ。**

- `Int`, `String`, `Float`, `Boolean`といったScalers。
- Enum
- ScalerおよびEnumのArray

型の使用の詳細は
https://facebook.github.io/graphql/#sec-Input-Types

`Category` Enumを使用した例。

query

```graphql
query getFewPosts($category: Category) {
  posts(category: $category) {
    title
  }
}
```

query variablesとして、categoryにCategory.PRODUCTを指定する。

```json
{
  "category": "PRODUCT"
}
```

結果は、

```json
{
  "data": {
    "posts": [
      {
        "title": "New Feature: Tracking Error Status with Kadira"
      },
      {
        "title": "Introducing Kadira Debug, Version 2"
      },
      {
        "title": "Awesome Error Tracking Solution for Meteor Apps with Kadira"
      },
      {
        "title": "What Should Kadira Build Next?"
      },
      {
        "title": "Tracking Meteor CPU Usage with Kadira"
      }
    ]
  }
}
```

---

# At the end

query variablesは、client sideでGraphQLを利用する場合はとても有用。

