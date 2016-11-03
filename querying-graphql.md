Querying GraphQL
===

GraphQL query languageはGraphQLの大部分を占める。
この章を終えたら、GraphQLの良さとGraphQLのqueryの書き方について理解しているだろう。

---

# Hello GraphQL

blogの`latest post`の`title`と`summary`を取得するqueryを書く。

```graphql
{
    lastestPost {
        title,
        summary
    }
}
```

結果は、

```json
{
  "data": {
    "latestPost": {
      "title": "New Feature: Tracking Error Status with Kadira",
      "summary": "Lot of users asked us to add a feature to set status for errors in the Kadira Error Manager. Now, we've that functionality."
    }
  }
}
```

`latestPost`の部分はGraphQLにおいて**root query fields**と呼ばれる。


# Get authors

```graphql
{
    authors {
        name
    }
}
```

結果は、

```json
{
  "data": {
    "authors": [
      {
        "name": "Arunoda Susiripala"
      },
      {
        "name": "Pahan Sarathchandra"
      },
      {
        "name": "Kasun Indi"
      }
    ]
  }
}
```

# Nested querying

例えば、一つのGraphQL queryで全てのBlog postとpostに対するコメントを一緒に取得することができる。

```graphql
{
  posts {
    title,
    author {
      name
    },
    summary,
    comments {
      content
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
        "title": "New Feature: Tracking Error Status with Kadira",
        "author": {
          "name": "Pahan Sarathchandra"
        },
        "summary": "Lot of users asked us to add a feature to set status for errors in the Kadira Error Manager. Now, we've that functionality.",
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
        "author": {
          "name": "Arunoda Susiripala"
        },
        "summary": "A short guide to means, histograms and percentiles and how we can use them in a real situation.",
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
        "title": "Introducing Kadira Debug, Version 2",
        "author": {
          "name": "Arunoda Susiripala"
        },
        "summary": "Today, we are introducing a new version of Kadira Debug. It comes with many UI improvements and support for CPU profiling.",
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
        "title": "Sharing the Meteor Login State Between Subdomains",
        "author": {
          "name": "Kasun Indi"
        },
        "summary": "In this blog we'll show you how we shared login state between our static web app and our Meteor app Kadira UI.",
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
        "title": "Meteor Server Side Rendering Support with FlowRouter and React",
        "author": {
          "name": "Arunoda Susiripala"
        },
        "summary": "This is an experiment arunoda did to implement Server Side Renderng(SSR) using Flow Router and React.",
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
        "title": "Awesome Error Tracking Solution for Meteor Apps with Kadira",
        "author": {
          "name": "Arunoda Susiripala"
        },
        "summary": "Error tracking is so much important and goes side by side with performance issues. This is the public beta announcement of Kadira's error tracking solution.",
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
        "title": "What Should Kadira Build Next?",
        "author": {
          "name": "Arunoda Susiripala"
        },
        "summary": "We are working on the next few major feature releases for Kadira. We would like to know your preference. Pre-order the feature you would most like to see in the next major release (scheduled for August 1).",
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
        "title": "Tracking Meteor CPU Usage with Kadira",
        "author": {
          "name": "Arunoda Susiripala"
        },
        "summary": "We've replaced EventLoop Utilization chart with actual CPU Usage. See why?",
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
        "title": "How Brent is using Kadira with his development workflow",
        "author": {
          "name": "Arunoda Susiripala"
        },
        "summary": "Denis has been using Kadira from the initial beta release and helped us a lot on identifying issues with Kadira. This is how he is using Kadira.",
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

# Arguments

data全体ではなく、subsetを取得したいときがある。argumentsはそのためのもの。

ページングの例。

```graphql
{
  recentPosts(count: 3) {
    title
  }
}
```

結果は、

```json
{
  "data": {
    "recentPosts": [
      {
        "title": "New Feature: Tracking Error Status with Kadira"
      },
      {
        "title": "Understanding Mean, Histogram and Percentiles"
      },
      {
        "title": "Introducing Kadira Debug, Version 2"
      }
    ]
  }
}
```

## Arugments for nested fields

top-level fieldsのargumentsと同様に、nested fieldsに対してもargmentsをつけることができる。

Blogのcommentを件数指定して取得する例。

```graphql
{
  recentPosts(count: 2) {
    title,
    comments(limit: 1) {
      content
    }
  }
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
          }
        ]
      },
      {
        "title": "Understanding Mean, Histogram and Percentiles",
        "comments": [
          {
            "content": "This is a very good blog post"
          }
        ]
      }
    ]
  }
}
```

# Multiple fields

一つのGraphQL queryに複数のroot query fieldsを指定することができる。サーバは並列に処理され、結果は一つにまとめて取得できる。

latestPOstとauthorsを取得する例。


```graphql
{
  latestPost {
    title
  },

  authors {
    name
  }
}
```

結果は、


```json
{
  "data": {
    "latestPost": {
      "title": "New Feature: Tracking Error Status with Kadira"
    },
    "authors": [
      {
        "name": "Arunoda Susiripala"
      },
      {
        "name": "Pahan Sarathchandra"
      },
      {
        "name": "Kasun Indi"
      }
    ]
  }
}
```

# Assigning a result to a variable

一つのqueryにおいて、同じroot query fieldを複数回使いたい場合がある。
例えば、author `_id`と`name`をそれぞれroot query fieldsとして取得したい。

書き方は少し変わって以下のようになる。

```graphql
{
  latestPost: latestPost {
    title
  },

  authorNames: authors {
    name
  },

  authorIds: authors {
    _id
  }
}
```

結果は、

```json
{
  "data": {
    "latestPost": {
      "title": "New Feature: Tracking Error Status with Kadira"
    },
    "authorNames": [
      {
        "name": "Arunoda Susiripala"
      },
      {
        "name": "Pahan Sarathchandra"
      },
      {
        "name": "Kasun Indi"
      }
    ],
    "authorIds": [
      {
        "_id": "arunoda"
      },
      {
        "_id": "pahan"
      },
      {
        "_id": "indi"
      }
    ]
  }
}
```

