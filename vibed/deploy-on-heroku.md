# Herokuにデプロイ

### 事前要件

- Heroku[アカウント](https://signup.heroku.com/login)
- [Git](https://git-scm.com/)がインストールされていること
- アプリケーションがdmd(`dub build`)を使ってエラー無くコンパイルできること

### 1 : Appのセットアップ

Herokuはデプロイされたアプリケーションと通信する方法を知る必要があります。
従ってアプリケーションに注入され、ポートにバインドしリッスンする必要があるグローバル`PORT`環境変数が提供されます。
開発のためにデフォルトのポート(ここで__8080__)を設定する必要があります:

```d
shared static this() {
  // ...
  auto settings = new HTTPServerSettings;
  // $PORT変数が設定されていない場合デフォルトのポートを提供します。
  settings.port = environment.get("PORT", "8080").to!ushort;
  listenHTTP(settings, router);
}
```

さらに、Appを開始する時にどのコマンドを実行するかを明示的に宣言する、アプリケーションのルートディレクトリのテキストファイルである`Procfile`を作成しましょう。

このサンプルの`Procfile`は以下のようになります:

```
web: ./hello-world
```

### 2 : Appの準備

話をすすめる前に[Heroku Toolbelt](https://toolbelt.heroku.com/standalone)を使ってHerokuコマンドラインにログインしましょう。

これはアプリケーションやアドオンの管理やスケーリングができる、Herokuコマンドラインインターフェース(CLI)。

Toolbeltのインストール後以下を実行してください:

```
$ heroku login
```

### 3 : Appの作成

[heroku dashboard](https://dashboard.heroku.com)へ行き新しいAppを作成しましょう。
その後作ったAppの名前を覚えておいてください、後で役に立ちます。

またはこのようにコマンドラインを使いましょう:

```
$ heroku create
Creating app... done, ⬢ rocky-hamlet-67506
https://rocky-hamlet-67506.herokuapp.com/ | https://git.heroku.com/rocky-hamlet-67506.git
```

ここではAppの名前は*rocky-hamlet-67506*です。

### gitを使ってデプロイ

`git`コマンドを使って直接Appをデプロイします。新しいリリースをプッシュできる異なるgitリモートエンドポイントを追加しなければなりません。
従って前のチャプターで新しく作ったアプリケーションの名前を引数として渡す必要があります。ここでは*rocky-hamlet-67506*です。

```
$ heroku git:remote -a rocky-hamlet-67506
```

リモートエンドポイントはgitコンフィグに追加されることを知っておいてください:

```
$ git remote -v
heroku	https://git.heroku.com/rocky-hamlet-67506.git (fetch)
heroku	https://git.heroku.com/rocky-hamlet-67506.git (push)
```

### ビルドパックの追加

ビルドパックはアセットやコンパイルされたコードの生成を請け負います。

詳しい情報は[Heroku documentation](https://devcenter.heroku.com/articles/buildpacks)を閲覧してください。

デプロイメントには[Vibe.dビルドパック](https://github.com/MartinNowak/heroku-buildpack-d)が使えます:

```
$ heroku buildpacks:set https://github.com/MartinNowak/heroku-buildpack-d
```
デフォルトではこのビルドパックは最新の`dmd`コンパイラを使います。
`.d-compiler`ファイルをプロジェクトに追加することで、GDCやLDCを使い、特定のコンパイラのバージョンを選ぶことができます。

`dmd`、`ldc`、`gdc`を使い最新を選択するか`dmd-2.0xxx`、`ldc-1.0xxx`、`gdc-4.9xxx`を使いコンパイラの特定のバージョンを選択します。

### コードのデプロイ

あなたのgitの習慣で作業を進め、素晴らしいコードを書いてください。

新しいバージョンのリリースには、ただ最新のバージョンをHerokuエンドポイントにプッシュしてください。

```
$ git add .
$ git commit -am "My first vibe.d release"
$ git push heroku master
Counting objects: 9, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 997 bytes, done.
Total 9 (delta 0), reused 0 (delta 0)

-----> Fetching custom git buildpack... done
-----> D (dub package manager) app detected
-----> Building libevent
-----> Building libev
-----> Downloading DMD
-----> Downloading dub package manager
-----> Setting PATH:
-----> Initializing toolchain
-----> Building app
       Running dub build ...
Building configuration "application", build type release
Running dmd (compile)...
Compiling diet template 'index.dt' (compat)...
Linking...
       Build was successful
-----> Discovering process types
       Procfile declares types -> web
-----> Compiled slug size: 3.5MB
-----> Launching... done, v4
       https://rocky-hamlet-67506.herokuapp.com/ deployed to Heroku
To git@heroku.com:rocky-hamlet-67506.git
 * [new branch]      master -> master
```

以下のコマンドによってブラウザでAppを開きましょう。

```
$ heroku open
```

### dynoコンテナのスケーリング

デプロイ後、Appはweb dyno上で動作します。
dynoのことはProcfileで指定したコマンドが実行される軽量のコンテナだと考えてください。

`ps`コマンドを使っていくつのdynoが動作中かチェックできます:

```
$ heroku ps
Free dyno hours quota remaining this month: 550h 0m (100%)
For more information on dyno sleeping and how to upgrade, see:
https://devcenter.heroku.com/articles/dyno-sleeping

No dynos on ⬢ rocky-hamlet-67506
```

デフォルトでは、Appはデフォルトではリクエストを受け入れないフリーdynoにデプロイされます。
フリーdynoは30分活動がないとスリープします。これによって起動時の最初のリクエストが数秒遅延します。

以下を実行してdynoを開始してください:

```
$ heroku ps:scale web=1
```

### ログを見る

Herokuはログを時間順のすべてのAppとHerokuコンポーネンツの出力ストリームから集約されたイベントストリームとして扱い、
すべてのイベントの単一のチャンネルを提供します。

```
$ heroku logs --tail
```

## さらなる情報

HerokuへのAppのデプロイ後に、アドオンを使ってさらに素晴らしくできます。たとえば:

- [Postgresql](https://elements.heroku.com/addons/heroku-postgresql)
- [MongoDb](https://elements.heroku.com/addons/mongohq)
- [Logging](https://elements.heroku.com/addons#logging)
- [Caching](https://elements.heroku.com/addons#caching)
- [Error and exceptions](https://elements.heroku.com/addons#errors-exceptions)
