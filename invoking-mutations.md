Invoking Mutations
===

mutationsはGraphQLの背後にあるdatasetに副作用や変更を与えることを想定している。

multi field queryは並列に処理されることを学んだが、multiple mutationsは1つずつ逐次実行される。

---

# First mutation

blogに新しいauthorを作成する例。

```graphql
mutation {
  createAuthor(
    _id: "john",
    name: "John Carter",
    twitterHandle: "@john"
  ) {
    _id
    name
  }
}
```

結果は、

```json
{
  "data": {
    "createAuthor": {
      "_id": "john",
      "name": "John Carter"
    }
  }
}
```
`createAuthor`がmutationの実行の中身。`_id`,`name`はmutationの実行結果として取得したいプロパティが指定されている。

`mutation` keywordで始めるとqueryではなくmutationを表す。

ちなみに、引数のrequiredチェックもできる。

```graphql
mutation {
  createAuthor(
    _id: "Hoge",
    twitterHandle: "@blahblah"
  ) {
    _id
    name
    twitterHandle
  }
}
```

結果は、

```json
{
  "errors": [
    {
      "message": "Field \"createAuthor\" argument \"name\" of type \"String!\" is required but not provided."
    }
  ]
}
```

# Required arguments

必須な引数は、例えば`name`引数が文字列で必須の場合は、`name: String!`のように`!`が付けられる。

---

# Multiple mutations

queryと同様に複数のmutationを呼び出すこともできる。

新たな2人の`author`を作る例。

```graphql
mutation {
  sam: createAuthor(
    _id: "sam",
    name: "Sam Hautom"
    twitterHandle: "@sam"
  ) {
    _id
    name
  },

  chris: createAuthor(
    _id: "chris",
    name: "Chris Mather"
    twitterHandle: "@chris"
  ) {
    _id
    name
  } 
}
```

結果は、

```json
{
  "data": {
    "sam": {
      "_id": "sam",
      "name": "Sam Hautom"
    },
    "chris": {
      "_id": "chris",
      "name": "Chris Mather"
    }
  }
}
```

multiple mutationsにおけるmutation間の依存関係を見れる例。
一つ目のmutationで登録した`author`は2つ目のmutationではユニーク制約により失敗する。


```graphql
mutation {
  carter: createAuthor(
    _id: "carter",
    name: "Carter Boom"
    twitterHandle: "@carter"
  ) {
    _id
  },

  carter2: createAuthor(
    _id: "carter",
    name: "Carter Boom"
    twitterHandle: "@carter"
  ) {
    _id
  }
}
```

結果は、

```json
{
  "data": {
    "carter": {
      "_id": "carter"
    },
    "carter2": null
  },
  "errors": [
    {
      "message": "Author already exists: carter"
    }
  ]
}
```

---

# Mutations executed as a sequence

GraphQLでは、mutationsはシーケンシャルに実行される、そうでなければ、同じauthorをユニーク管理して追加するようなエラーを防ぐことが困難。

> mutationsの処理パラダイムは、GraphQLサーバの実装に依存する。

---

# Finally

既存の`author`データと紐付けてmutationを行う例。

```graphql
mutation {
  createPost(
    _id: "awesome-graphql",
    title: "awesome-graphql",
    content: "hogehogeblablah",
    category: OTHER,
    author: "sam"
  ) {
    title
    content
    summary
    category
    author {
      _id
      name
      twitterHandle
    }
  }
}
```

結果は、

```json
{
  "data": {
    "createPost": {
      "title": "awesome-graphql",
      "content": "hogehogeblablah",
      "summary": "hogehogeblablah",
      "category": "OTHER",
      "author": {
        "_id": "sam",
        "name": "Sam Hautom",
        "twitterHandle": "@sam"
      }
    }
  }
}
```

