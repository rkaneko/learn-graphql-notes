Defining Mutations
===

mutationsを使うとclientはdatasetに変更を加えることができる。

mutationsの定義はqueryの定義と似ていてい、内部ロジックが異なるだけ。

---

# Setting up

```bash
$ git co defining-mutations
```

---

# Hello mutations

blog postを作成するmutationの例。


```js
const Mutation = new GraphQLObjectType({
  name: "BlogMutations",
  description: "Mutations of our blog",
  fields: () => ({
    createPost: {
      type: Post,
      args: {
        title: {type: new GraphQLNonNull(GraphQLString)},
        content: {type: new GraphQLNonNull(GraphQLString)}
      },
      resolve: function(source, args) {
        let post = Object.assign({}, args);
        // Generate the _id
        post._id = `${Date.now()}::${Math.ceil(Math.random() * 9999999)}`;
        // Assign a user
        post.author = "arunoda";

        // Add the Post to the data store
        PostsList.push(post);

        // return the new post.
        return post;
      }
    }
  })
});
```

mutationは、

```graphql
mutation insertFirstPost {
  post: createPost(
    title: "GraphQL is Awesome",
    content: "Yep, it's purely awesome."
  ) {
    _id,
    title
  }
}
```

結果は、

```json
{
  "data": {
    "post": {
      "_id": "1478253161276::4472557",
      "title": "GraphQL is Awesome"
    }
  }
}
```

当然、`content`を指定しないmutationを投げると、

```graphql
mutation insertFirstPost {
  post: createPost(
    title: "GraphQL is Awesome",
  ) {
    _id,
    title
  }
}
```

NonNull制約に引っかかり、エラーになる。

```json
{
  "errors": [
    {
      "message": "Field \"createPost\" argument \"content\" of type \"String!\" is required but not provided."
    }
  ]
}
```

---

# Finally

mutationが起こったあとは、修正されたdocumentを返すことが重要。
