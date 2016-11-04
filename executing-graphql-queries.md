Executing GraphQL Queries
===

schemaを定義して、そのschemaに対しsandboxのUIからではなく、JavaScriptでqueriesを実行する方法を見ていく。

node.jsにおいて必要なライブラリは、`graphql` module。

---

# Setting up

```bash
$ mkdir hello-graphql
$ cd hello-graphql

$ npm init -f
$ npm i -D babel-cli babel-preset-es2015

$ npm i --save graphql
```

---

# Defining the schema

---

# Assigning query variables

