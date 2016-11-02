Fragments
===

fragmensは共通して使用されるfieldをグルーピングしたり、再利用する方法。

blogから幾人かのauthorを取得する例を考える。


```graphql
{
  arunoda: author(_id: "arunoda") {
    _id,
    name,
    twitterHandle
  },
  pahan: author(_id: "pahan") {
    _id,
    name,
    twitterHandle
  },
  indi: author(_id: "indi") {
    _id,
    name,
    twitterHandle
  }
}
```

↑のように`Author` typeを何度も書かなければいけない。これをさけるためにfragmentを使う。

次のように、`Author` typeにおけるfragmentを定義する。

```graphql
fragment authorInfo on Author {
  _id,
  name,
  twitterHandle
}
```

これを利用すると次のようなqueryになる。


```graphql
{
  arunoda: author(_id: "arunoda") {
    ...authorInfo
  },
  pahan: author(_id: "pahan") {
    ...authorInfo
  },
  indi: author(_id: "indi") {
    ...authorInfo
  }
}
```

fragmentの利用には、spread operator `...someFragment`を使う。

---

# Mixing fragments and fields

fragmentとfieldを組み合わせることもできる。

必須で欲しいfieldを集めたfragmentと必須ではないオプショナルで欲しいtwitterHandle fieldを組み合わせた例。

```graphql
fragment requiredAuthorInfo on Author {
  _id,
  name,
}
```

```graphql
{
  indi: author(_id: "indi") {
    ...requiredAuthorInfo
    twitterHandle
  }
}
```

fragmentで定義したfieldと、fragmentと組わせるfieldに同じfieldを定義した場合のqueryはどうなるか？

```graghql
{
  indi: author(_id: "indi") {
    ...requiredAuthorInfo,
    twitterHandle,
    _id # this is already defined in the fragment
  }
}

fragment requiredAuthorInfo on Author {
  _id,
  name,
}
```

結果は、問題なく実行され、結果のfieldが複数返ってくることもない。


```json
{
  "data": {
    "indi": {
      "_id": "indi",
      "name": "Kasun Indi",
      "twitterHandle": "@indi"
    }
  }
}
```

---

# Fragments with nested fields

fragmentでは、ネストしたfieldも定義できる。


```graphql
fragment postInfo on Post {
  title,
  content,
  author {
    name
  },
  comments {
    content
  }
}
```

結果は、

```json
{
  "data": {
    "post1": {
      "title": "New Feature: Tracking Error Status with Kadira",
      "content": "Here is a common feedback we received from our users:\n\n> Hi, I have a suggestion. It would be great if I could \"dismiss\" errors or mark them as resolved on my end. This way, I can keep track of which errors I have resolved.\n\nToday we are happy to announce new Kadira feature which allows you to track **status** to errors. With that, you can mark errors as \"Ignored\", \"Fixing\" or \"Fixed\".\n\nOnce you mark an error as \"Ignored\", it will be hidden. \n\nBut you can click on \"Show Ignored Errors\" checkbox or filter by \"ignored\" status to view them again.\n\n![show ignored errors](https://cldup.com/XvoJk9RGWf.gif)\n\nYou can also filter errors by status like this:\n\n![filtering errors with status](https://cldup.com/76JZ6wmbVb.gif)\n\nWe are rolling out this feature to all our paid users. [Give it a try](https://ui.kadira.io/apps/AUTO/errors/overview?metric=count).\n\n### What’s next?\n\nRight now we are planning to add few more feature related this. Could you help us on [prioritizing](https://arunoda.typeform.com/to/hyTwsy) them? Trust me, [it won't take a minute](https://arunoda.typeform.com/to/hyTwsy).",
      "author": {
        "name": "Pahan Sarathchandra"
      },
      "comments": [
        {
          "content": "This is a very good blog post"
        },
        {
          "content": "Keep up the good work"
        }
      ]
    },
    "post2": {
      "title": "Sharing the Meteor Login State Between Subdomains",
      "content": "Most developers and companies use two different apps for the marketing website and for the app itself. Thus, they can update each of the apps without affecting the other. [Stripe](https://stripe.com/), [Digital Ocean](https://www.digitalocean.com/) and many other companies follow this technique. Most Meteor apps also do the same.\n\nSo, in a scenario like this, sometimes we need to show the login state of the app on the landing page too. For an example, see our Kadira home page (<https://kadira.io>). If you are logged into the Kadira app (<https://ui.kadira.io>), we show a button with \"Open Kadira UI\" on the home page, which replaces the login button.\n\n[![Login State Example on Kadira](https://cldup.com/q9nKu_OIhQ.png)](https://kadira.io)\n\n## How Did We Do It?\n\nMeteor does not have a built-in way to share login states across multiple apps or subdomains. So, we have to find an alternative way to do so.\n\nAs a solution, we can use browser cookies to share the login state between multiple domains. That's exactly what we did. We wrapped this up into a Meteor package, which now you can also use.\n\nIn this guide, I'm going to explain how to share the login state between multiple domains using the [`kadira:login-state`](https://github.com/kadirahq/meteor-login-state) package.\n\n### On Meteor App\n\nFirst of all, install the `kadira:login-state` package in your Meteor app:\n\n~~~\nmeteor add kadira:login-state\n~~~\n\nThen, you need to add a new entry in the `public` object as the `loginState` in the `settings.json` file for your app. (If you haven't created the settings.json yet, you need to create it first.)\n\n~~~json\n{\n  \"public\": {\n    \"loginState\": {\n      \"domain\": \".your-domain-name.com\",\n      \"cookieName\": \"app-login-state-cookie-name\"\n    }\n  }\n}\n~~~\n\nThe `domain` field must be your main domain name, starting with a dot. It allows you to share the login state, which can be accessed from any of its subdomains. You can use any appropriate identifier, such as `cookieName`.\n\nNow, everything has been set up on the Meteor app.\n\n### On the Static App (the Landing Page)\n\nNow we have to show the login state of the app on the landing page. For this, we need to add support for the login state for the static app (or landing page).\n\nActually, there are three different ways to do this. Here I will show you how to do so by pasting a few lines of JavaScript code.\n\nYou need to create a JavaScript file in your js folder. I create it as `js/login_state.js`. After that, copy and paste the following code snippet into it:\n\n~~~javascript\nLoginState = {};\n\nLoginState.get = function(cookieName) {\n  var loginState = getCookie(cookieName);\n  if(loginState) {\n    return JSON.parse(decodeURIComponent(loginState));\n  } else {\n    return false;\n  }\n};\n\nfunction getCookie(cname) {\n  var name = cname + \"=\";\n  var ca = document.cookie.split(';');\n  for(var i=0; i < ca.length; i++) {\n      var c = ca[i];\n      while (c.charAt(0)==' ') c = c.substring(1);\n      if (c.indexOf(name) != -1) return c.substring(name.length,c.length);\n  }\n  return;\n}\n~~~\n\nInsert that file into the head section of your HTML document: \n\n`<script src=\"js/login-state.js\"></script>`\n\n> If you prefer, you can also use [Browserify](https://github.com/kadirahq/meteor-login-state#installing-via-browserify) or [Bower](https://github.com/kadirahq/meteor-login-state#installing-via-bower) to load the above JS file.\n> The package name for both Browserify and Bower is `meteor-login-state`.\n\nThen, use the following code to get the login state of your app. You need to provide the relevant `cookieName` to do so: \n\n~~~javascript\nvar loginState = LoginState.get(\"app-login-state-cookie-name\");\nif(loginState) {\n  // the user has loggedIn to the meteor app\n  // see the loginState Object for the addtional data\n  // (append your code here!)\n  console.log(loginState);\n} else {\n  // user has not loggedIn yet.\n  // (append your code here!) \n}\n~~~\n\nThe `loginState` object will be something like this:\n\n~~~json\n{\n  timestamp: 1435835751489,\n  username: \"username\",\n  userId: \"meteor-user-id\",\n  email: \"user@email.com\"\n  url: \"https://ui.kadira.io\"\n}\n~~~\n\nNow you can do whatever you need to do with the login state.\n\nGive it a try and let me know what you think.",
      "author": {
        "name": "Kasun Indi"
      },
      "comments": [
        {
          "content": "This is a very good blog post"
        },
        {
          "content": "Keep up the good work"
        }
      ]
    }
  }
}
```

---

# Fragments with nested fragments

Blogのauthorとcommentのauthorを取得したい例。

```graghql
fragment postInfo on Post {
  title,
  content,
  ...authorInfo
  comments {
    content,
    ...authorInfo
  }
}

fragment authorInfo on HasAuthor {
  author {
    _id, 
    name
  }
}
```

利用したqueryは以下の通り。


```graphql
{
  post1: post(_id: "03390abb5570ce03ae524397d215713b") {
    ...postInfo
  },
  post2: post(_id: "0176413761b289e6d64c2c14a758c1c7") {
    ...postInfo
  }
}
```

結果は、


```json
{
  "data": {
    "post1": {
      "title": "New Feature: Tracking Error Status with Kadira",
      "content": "Here is a common feedback we received from our users:\n\n> Hi, I have a suggestion. It would be great if I could \"dismiss\" errors or mark them as resolved on my end. This way, I can keep track of which errors I have resolved.\n\nToday we are happy to announce new Kadira feature which allows you to track **status** to errors. With that, you can mark errors as \"Ignored\", \"Fixing\" or \"Fixed\".\n\nOnce you mark an error as \"Ignored\", it will be hidden. \n\nBut you can click on \"Show Ignored Errors\" checkbox or filter by \"ignored\" status to view them again.\n\n![show ignored errors](https://cldup.com/XvoJk9RGWf.gif)\n\nYou can also filter errors by status like this:\n\n![filtering errors with status](https://cldup.com/76JZ6wmbVb.gif)\n\nWe are rolling out this feature to all our paid users. [Give it a try](https://ui.kadira.io/apps/AUTO/errors/overview?metric=count).\n\n### What’s next?\n\nRight now we are planning to add few more feature related this. Could you help us on [prioritizing](https://arunoda.typeform.com/to/hyTwsy) them? Trust me, [it won't take a minute](https://arunoda.typeform.com/to/hyTwsy).",
      "author": {
        "_id": "pahan",
        "name": "Pahan Sarathchandra"
      },
      "comments": [
        {
          "content": "This is a very good blog post",
          "author": {
            "_id": "pahan",
            "name": "Pahan Sarathchandra"
          }
        },
        {
          "content": "Keep up the good work",
          "author": {
            "_id": "indi",
            "name": "Kasun Indi"
          }
        }
      ]
    },
    "post2": {
      "title": "Sharing the Meteor Login State Between Subdomains",
      "content": "Most developers and companies use two different apps for the marketing website and for the app itself. Thus, they can update each of the apps without affecting the other. [Stripe](https://stripe.com/), [Digital Ocean](https://www.digitalocean.com/) and many other companies follow this technique. Most Meteor apps also do the same.\n\nSo, in a scenario like this, sometimes we need to show the login state of the app on the landing page too. For an example, see our Kadira home page (<https://kadira.io>). If you are logged into the Kadira app (<https://ui.kadira.io>), we show a button with \"Open Kadira UI\" on the home page, which replaces the login button.\n\n[![Login State Example on Kadira](https://cldup.com/q9nKu_OIhQ.png)](https://kadira.io)\n\n## How Did We Do It?\n\nMeteor does not have a built-in way to share login states across multiple apps or subdomains. So, we have to find an alternative way to do so.\n\nAs a solution, we can use browser cookies to share the login state between multiple domains. That's exactly what we did. We wrapped this up into a Meteor package, which now you can also use.\n\nIn this guide, I'm going to explain how to share the login state between multiple domains using the [`kadira:login-state`](https://github.com/kadirahq/meteor-login-state) package.\n\n### On Meteor App\n\nFirst of all, install the `kadira:login-state` package in your Meteor app:\n\n~~~\nmeteor add kadira:login-state\n~~~\n\nThen, you need to add a new entry in the `public` object as the `loginState` in the `settings.json` file for your app. (If you haven't created the settings.json yet, you need to create it first.)\n\n~~~json\n{\n  \"public\": {\n    \"loginState\": {\n      \"domain\": \".your-domain-name.com\",\n      \"cookieName\": \"app-login-state-cookie-name\"\n    }\n  }\n}\n~~~\n\nThe `domain` field must be your main domain name, starting with a dot. It allows you to share the login state, which can be accessed from any of its subdomains. You can use any appropriate identifier, such as `cookieName`.\n\nNow, everything has been set up on the Meteor app.\n\n### On the Static App (the Landing Page)\n\nNow we have to show the login state of the app on the landing page. For this, we need to add support for the login state for the static app (or landing page).\n\nActually, there are three different ways to do this. Here I will show you how to do so by pasting a few lines of JavaScript code.\n\nYou need to create a JavaScript file in your js folder. I create it as `js/login_state.js`. After that, copy and paste the following code snippet into it:\n\n~~~javascript\nLoginState = {};\n\nLoginState.get = function(cookieName) {\n  var loginState = getCookie(cookieName);\n  if(loginState) {\n    return JSON.parse(decodeURIComponent(loginState));\n  } else {\n    return false;\n  }\n};\n\nfunction getCookie(cname) {\n  var name = cname + \"=\";\n  var ca = document.cookie.split(';');\n  for(var i=0; i < ca.length; i++) {\n      var c = ca[i];\n      while (c.charAt(0)==' ') c = c.substring(1);\n      if (c.indexOf(name) != -1) return c.substring(name.length,c.length);\n  }\n  return;\n}\n~~~\n\nInsert that file into the head section of your HTML document: \n\n`<script src=\"js/login-state.js\"></script>`\n\n> If you prefer, you can also use [Browserify](https://github.com/kadirahq/meteor-login-state#installing-via-browserify) or [Bower](https://github.com/kadirahq/meteor-login-state#installing-via-bower) to load the above JS file.\n> The package name for both Browserify and Bower is `meteor-login-state`.\n\nThen, use the following code to get the login state of your app. You need to provide the relevant `cookieName` to do so: \n\n~~~javascript\nvar loginState = LoginState.get(\"app-login-state-cookie-name\");\nif(loginState) {\n  // the user has loggedIn to the meteor app\n  // see the loginState Object for the addtional data\n  // (append your code here!)\n  console.log(loginState);\n} else {\n  // user has not loggedIn yet.\n  // (append your code here!) \n}\n~~~\n\nThe `loginState` object will be something like this:\n\n~~~json\n{\n  timestamp: 1435835751489,\n  username: \"username\",\n  userId: \"meteor-user-id\",\n  email: \"user@email.com\"\n  url: \"https://ui.kadira.io\"\n}\n~~~\n\nNow you can do whatever you need to do with the login state.\n\nGive it a try and let me know what you think.",
      "author": {
        "_id": "indi",
        "name": "Kasun Indi"
      },
      "comments": [
        {
          "content": "This is a very good blog post",
          "author": {
            "_id": "pahan",
            "name": "Pahan Sarathchandra"
          }
        },
        {
          "content": "Keep up the good work",
          "author": {
            "_id": "indi",
            "name": "Kasun Indi"
          }
        }
      ]
    }
  }
}
```

---

# Finally

fragmentはmutationsでも使用することができる。

クライアントアプリにおけるGraphQLの使い方を議論する上で、fragmentsは重要な役割を果たす。詳細はまたあとで。
