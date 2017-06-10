# データベース接続

Vibe.dで簡単にあなたのバックエンドサービスのデータベースにアクセスできます。
**MongoDB**や**Redis**のサポートはvibe.dに直接あり、
他のデータベースアダプタは[code.dlang.org](https://code.dlang.org)
で見つけることができます。

### MongoDB

MongoDBへのアクセスはクラス[`MongoClient`](http://vibed.org/api/vibe.db.mongo.client/MongoClient)
で設計されています。実装は外部依存を持たず、vibe.dの非同期ソケットを使って実装されています。
接続にレイテンシがあった場合でもブロックは必要ありません。

    auto client = connectMongoDB("127.0.0.1");
    auto users = client.getCollection("users");
    users.insert(Bson("peter"));

### Redis

Redisサポートもvibe.dソケットで実装されており外部依存を持ちません。
実装の中枢はRedisサーバに対してコマンドを実行できる
[`RedisDatabase`](http://vibed.org/api/vibe.db.redis.redis/RedisDatabase)クラスです。
Redisに保存されているリストに透過的にアクセスできる
[`RedisList`](http://vibed.org/api/vibe.db.redis.types/RedisList)のような
便利なラッパもあります。

### MySQL

公式MySQLライブラリへの外部依存なしにMySQLのサポートが
[mysql-native](http://code.dlang.org/packages/mysql-native)プロジェクト
を使って追加できます。これもvibe.dのノンブロッキングソケットをサポートしています。

### Postgresql

フル機能のPostgresqlクライアントは公式**libpq**ライブラリを基にした外部モジュール
[dpq2](http://code.dlang.org/packages/dpq2)で実装されています。
これはvibe.dのイベントシステムを非同期的振る舞いを実装するのに使っています。

Postgresqlの他の代替品に、
vibe.dソケットで実装したPostgresqlクライアントで、
外部依存がない[ddb](http://code.dlang.org/packages/ddb)
があります。