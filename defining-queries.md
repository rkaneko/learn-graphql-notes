Defining Queries
===

これまでは、すでに定義されたschemaに対してqueryを呼び出すことを見てきた。
今度は、query languagesを理解してGraphQLのschemaを書いてみる。

---

# Setting up

```bash
$ git clone https://github.com/kadirahq/graphql-blog-schema.git
$ cd graphql-blog-schema

$ git checkout graphql-blog-schema

# install deps
$ npm install

# start server.js
$ npm start
```

---

# Inspecting the schema

`BlogSchema`というSchemaを定義する。

```js
const Query = new GraphQLObjectType({
  name: 'BlogSchema',
  description: "Root of the Blog Schema",
  fields: () => ({
    echo: {
      type: GraphQLString,
      description: "Echo what you enter",
      args: {
        message: {type: GraphQLString}
      },
      resolve: function(root, {message}) {
        return `recieved ${message}`;
      }
    }
  })
});

const Schema = new GraphQLSchema({
  query: Query
});
```

---

# Defining the post type

root query fieldに実用に応用できる型を定義してみる。

Post型を定義する。

```js
const Post = new GraphQLObjectType({
  name: "Post",
  description: "This represent a Post",
  fields: () => ({
    _id: {type: new GraphQLNonNull(GraphQLString)},
    title: {type: new GraphQLNonNull(GraphQLString)},
    content: {type: GraphQLString}
  })
});
```

root queryに`posts` fieldを追加する。

```js
const Query = new GraphQLObjectType({
  name: 'BlogSchema',
  description: "Root of the Blog Schema",
  fields: () => ({
    posts: {
      type: new GraphQLList(Post),
      resolve: function() {
        return PostsList;
      }
    }
  })
});
```

`resolve`で返している`PostsList`の実態はPost ObjectのArrayをJSで定義したもの。実際のアプリでは、ここにDatastoreから取得したデータを詰めることになる。

このqueryを使用してみる。

```graphql
{
  posts: posts {
    _id
    title
    content
  }
}
```

結果は、

```json
{
  "data": {
    "posts": [
      {
        "_id": "0176413761b289e6d64c2c14a758c1c7",
        "title": "Sharing the Meteor Login State Between Subdomains",
        "content": "blahblah"
      },
      {
        "_id": "0176413761b289e6d64c2c14a758c1c8",
        "title": "blahblah",
        "content": "blahblah"
      }
    ]
  }
}
```

次のように`resolve`で`Post`の`_id`のみを返すObjectのArrayを返すような変更をして、

```js
resolve: function() {
  return PostsList.map(post => ({_id: post._id})
                  .filter((post, index) => index === 0);
}
```

次のように、`title`が`NonNull`指定されているにもかかわらず、`title`を要求するqueryを投げると次のような結果が返る。

```json
{
  "data": {
    "posts": [
      null
    ]
  },
  "errors": [
    {
      "message": "Cannot return null for non-nullable field Post.title."
    }
  ]
}
```

---

# Defining default values

NonNull指定されている戻り値のfieldとしてnullが返されてエラーが出ると、クライアントサイドからすると戻り値に一貫性がない。

これを避けるためには、

- Nullableにする
- default valueを使用する

という2手法が取れる。

NonNull指定のままdefault valueを使用する方法を見る。

```js
const Post = new GraphQLObjectType({                                               
  name: "Post",
  description: "this represent a Post",
  fields: () => ({
    _id: {type: new GraphQLNonNull(GraphQLString)},
    title: {
      type: new GraphQLNonNull(GraphQLString),
      resolve: function(post) {
        return post.title || "[default value]Does not exist";
      }   
    },  
    content: {type: GraphQLString}
  })
});
```

fieldに`resolve`関数を定義し、default valueを返すようにする。

結果は、

```json
{
  "data": {
    "posts": [
      {
        "title": "[default value]Does not exist"
      }
    ]
  }
}
```

---

# Defining nested fields

`Post`型に`Author`型の値をfieldに追加する。

```js
const Author = new GraphQLObjectType({
  name: "Author",
  description: "This represent an author",
  fields: () => ({
    _id: {type: new GraphQLNonNull(GraphQLString)},
    name: {type: GraphQLString}
  })
});

const Post = new GraphQLObjectType({
  name: "Post",
  description: "this represent a Post",
  fields: () => ({
    _id: {type: new GraphQLNonNull(GraphQLString)},
    title: {
      type: new GraphQLNonNull(GraphQLString),
      resolve: function(post) {
        return post.title || "[default value]Does not exist";
      }
    },
    content: {type: GraphQLString},
    author: {
      type: Author,
      resolve: function(post) {
        return AuthorsList.find(a => a._id === post.author);                       
      }
    }
  })
});
```

queryは

```graphql
{
  posts: posts {
    title,
    author {
      name
    }
  }
}
```

結果は、

```json
{
  "data": {
    "posts": [
      {
        "title": "Sharing the Meteor Login State Between Subdomains",
        "author": {
          "name": "Kasun Indi"
        }
      }
    ]
  }
}
```

---

# Finally

GraphQLのschemaの書き方について、ネストしたfieldの定義の仕方等を学んで、グラフのようにdatasetを作れる。
