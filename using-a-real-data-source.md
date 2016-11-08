Using A Real Data Source
===

これまではインメモリのデータ・ソースをバックエンドに使用してきたが、リアルワールドではRDBMSやKVSといったものがバックエンドになるのが普通。

この章では、MongoDBをバックエンドとして利用していく。

---

# Setting up

- [How to Install MongoDB on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-16-04)

---

# Async / promises

リアルワールドのデータ・ソースにアクセスする場合には、ときに非同期処理で実行をしなければならない場面もある。

ここでは、[ES2015 promises](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)を使って実現する。

## Accessing MongoDB

[promised-mongo](https://www.npmjs.com/package/promised-mongo)を利用してMongoDBにアクセスする。

- [Mongo shell](https://docs.mongodb.com/v3.2/mongo/#start-the-mongo-shell)

---

# Implementing a mutation


