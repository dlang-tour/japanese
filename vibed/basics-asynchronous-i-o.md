# 基礎と非同期I/O

デフォルトのビルドパラメータを使って、特殊な`shared static this()`
モジュールコンストラクタによってvibe.dアプリケーションの
`main()`関数は指定されます:

    import vibe.d;
    shared static this() {
        // ここにVibe.dのコード
    }

モジュールコンストラクタは`main()`の前に一度だけ実行されます。
Vibe.dは独自の`main()`を提供してすべてのイベントループと
維持管理をユーザーコードから隠蔽します。

Vibe.dは**ファイバー**を非同期I/Oを実装するために利用しています:
例えば読み込むデータがないなどの理由でソケットの呼び出しがブロックされるたびに、
現在実行中のファイバーは現在の実行コンテキストを`yield`し、
別の操作のためにフィールドを離れます。データが利用可能になれば実行を再開します:

    // ブロックするかもしれませんが透過的です。
    // ソケットの準備ができているならvibe.dは
    // ここに戻るようにします。
    line = connection.readLine();
    // ここもブロックするかもしれません
    connection.write(line);

このコードは**同期的**で現在のスレッドをブロックするように見えますが、
そうではありません!
コードは綺麗で簡潔ですが、その上で、
1つのコアで何千もの接続を可能にする非同期I/Oの力を利用しています。

すべてのvibe.dの機能はファイバーを基にした非同期ソケット操作利用しており、
あなたはひとつの遅いMongoDBサーバーの接続がアプリケーション全体を
ブロックすることについて心配する必要はありません。

シンプルなTCPベースのechoサーバを実装する方法についての例を確認してください。

## {SourceCode:disabled}

```d
import vibe.d;

shared static this()
{
    // 8080ポート(TCP)をListenします。
    // 新しい接続が到着するたびに、
    // 2番めの引数で指定されたデリゲートによって
    // 処理されます。connはクライアントへの
    // TCP接続オブジェクトです。
    listenTCP(8080, (TCPConnection conn) {
        string line;
        conn.write("ECHO server says Hi!\r\n");
        conn.write("Type 'quit' to close.\r\n");
        while (line != "quit") {
            line = cast(string) conn.readLine();
            conn.write("ECHO: " ~ line
              ~ "\r\n");
        }

        // デリゲートがここを出るとそのクライアントの
        // 接続が閉じられます。
    });
}
```
