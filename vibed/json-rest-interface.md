# JSON RESTインターフェース

Vibe.dによってJSONウェブサービスの素早い実装ができます。
`/api/v1/chapters`
へのHTTPリクエストに対する下記のJSON出力を実装したいときは:

    [
      {
        "title": "Hello",
        "id": 1,
        "sections": [
          {
            "title": "World",
            "id": 1
          }
        ]
      },
      {
        "title": "Advanced",
        "id": 2,
        "sections": []
      }
    ]

まず対応する関数を実装するインターフェースと、
**1対1**にシリアライズされるDの`struct`を定義します:

    interface IRest
    {
        struct Section {
            string title;
            int id;
        }
        struct Chapter {
            string title;
            int id;
            Section[] sections;
        }
        @path("/api/v1/chapters")
        Chapter[] getChapters();
    }

実際にデータ構造体を埋めるには、インターフェースから継承し、
ビジネスロジックを実装する必要があります:

    class Rest: IRest {
        Chapter[] getChapters() {
          // 埋める
        }
    }

`Rest`クラスのインスタンスを登録する`URLRouter`インスタンスで終わりです!

    auto router = new URLRouter;
    router.registerRestInterface(new Rest);

Vibe.d **RESTインターフェースジェネレータ**はpostされたJSONオブジェクトの子要素が
メンバ関数のパラメータにマップされたPOSTリクエストもサポートしています。

RESTインターフェースは定められたサーバに、バックエンド側で使うのと同じメンバ関数を使って、
透過的にJSONリクエストを送るRESTクライアントインスタンスを生成するのに使えます。
これはコードがクライアントとサーバーで共有される時に便利です。

    auto api = new RestInterfaceClient!IRest("http://127.0.0.1:8080/");
    // GET /api/v1/chapters を送りデシリアライズする
    // IRest.Chapter[] へのレスポンス
    auto chapters = api.getChapters();

## {SourceCode:disabled}

```d
import vibe.d;

interface IRest
{
    struct Section
    {
        string title;
        int id;
    }

    struct Chapter
    {
        string title;
        int id;
        Section[] sections;
    }

    @path("/api/v1/chapters")
    Chapter[] getChapters();

    /*
    このようなpostデータ:
        { "title": "D Language" }
    */
    @path("/api/v1/add-chapter")
    @method(HTTPMethod.POST)
    int addChapter(string title);
}

class Rest: IRest
{
    private Chapter[] chapters_;

    this()
    {
        chapters_ = [ Chapter("Hello", 1,
                [ Section("World", 1) ] ),
                 Chapter("Advanced", 2) ];
    }

    Chapter[] getChapters()
    {
        return chapters_;
    }

    int addChapter(string title)
    {
        import std.algorithm : map, max, reduce;
        // 次の最も大きいIDを生成
        auto newId = chapters_.map!(x => x.id)
                            .reduce!max + 1;
        chapters_ ~= Chapter(title, newId);
        return newId;
    }
}

shared static this()
{
    auto router = new URLRouter;
    router.registerRestInterface(new Rest);

    auto settings = new HTTPServerSettings;
    settings.port = 8080;
    listenHTTP(settings, router);
}
```
