# ウェブサーバ

Vibe.dはHTTP(S)ウェブサーバを一瞬のうちに書くことを可能にします:

    auto settings = new HTTPServerSettings;
    settings.port = 8080;
    listenHTTP(settings, &foo);

これはすべてのリクエストが`foo`関数によりハンドルされるウェブサーバをポート8080で開始します:

    void foo(HTTPServerRequest req,
        HTTPServerResponse res) { ... }

典型的な使用パターンと、異なるパスの設定を簡単にするために、
`URLRouter`クラスは`GET`、`POST`などを登録できる`.get("path", handler)`
や`.post("path", handler)`メンバ関数を使う、
またはメンバ関数としてウェブサーバーパスを実装したカスタム**ウェブインターフェース**
クラスを登録するハンドラを提供します:

    auto router = new URLRouter;
    router.registerWebInterface(new WebService);
    listenHTTP(settings, router);

カスタムウェブインターフェース`WebService`
のメンバ関数のパスはシンプルな仕組みを用いて自動的に決定されます:
* `index()`は`/index`にハンドルされます
* `getName()`は`GET`リクエスト`/name`にハンドルされます
* `postUsername()`は`/username`への`POST`リクエストにハンドルされます

メンバ関数に`@path("/hello/world")`
属性を属性付けすることによってカスタムパスを設定できます。
`POST`リクエストのパラメータは`_`プレフィックスのついた変数名を使い、
関数で利用できるようになります。そのパスから直接パラメータを明記することもできます:

    @path("/my/api/:id")
    void foo(int _id)

各関数にパラメータとして`HTTPServerResponse`や`HTTPServerRequest`を渡す必要はありません。
Vibe.dはそれが関数のパラメータリストにあるかどうか静的にチェックし、必要なら渡します。

## {SourceCode:disabled}

```d
import vibe.d;

class WebService
{
    /*
    このようにセッション変数を使うと、個々のユーザに関連する情報は
    各ユーザのセッションの間すべてのリクエストを通して持続します。
    */
    private SessionVar!(string, "username")
        username_;

    /*
    デフォルトではルートパス("/")へのリクエストはindexメソッドにルーティングされます。
    */
    void index(HTTPServerResponse res)
    {
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <form action="/username" method="POST">
        Your name:
        <input type="text" name="username">
        <input type="submit" value="Submit">
        </form>
        </body>
        </html>};

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }

    /*
    @path属性はurlルーティングのカスタマイズに使えます。
    こちらの"/name"へのリクエストはgetNameメソッドへとマップされます.
    */
    @path("/name")
    void getName(HTTPServerRequest req,
            HTTPServerResponse res)
    {
        import std.string : format;

        // ヘッダ情報<li>タグをリクエストの
        // ヘッダのプロパティを調べて生成します
        string[] headers;
        foreach(key, value; req.headers) {
            headers ~=
                "<li>%s: %s</li>"
                .format(key, value);
        }
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <h1>Your name: %s</h1>
        <h2>Headers</h2>
        <ul>
        %s
        </ul>
        </body>
        </html>}.format(username_,
                headers.join("\n"));

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }

    void postUsername(string username,
            HTTPServerResponse res)
    {
        username_ = username;
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <h1>Your name: %s</h1>
        </body>
        </html>}.format(username_);

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }
}

void helloWorld(HTTPServerRequest req,
        HTTPServerResponse res)
{
    res.writeBody("Hello");
}

shared static this()
{
    auto router = new URLRouter;
    router.registerWebInterface(new WebService);
    router.get("/hello", &helloWorld);

    auto settings = new HTTPServerSettings;
    // SessionVarを使うのに必要です
    settings.sessionStore =
        new MemorySessionStore;
    settings.port = 8080;
    listenHTTP(settings, router);
}
```
