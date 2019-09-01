# メッセージパッシング

スレッドとやり取りをして自分で同期をするかわりに、Dは複数コアを活用する優れた手段として
**メッセージパッシング**が使えます。スレッドは仕事を分配しそれら自身を同期させるために、
任意の値であるメッセージを使って連絡を取り合います。スレッドたちはマルチスレッディングの
よくある問題を避けるために設計によってデータを共有しません。

Dでメッセージパッシングを実装するすべての関数は[`std.concurrency`](https://dlang.org/phobos/std_concurrency.html)モジュールにあります。
`spawn`はユーザ定義関数をもとにした新しい**スレッド**を作ります:

    auto threadId = spawn(&foo, thisTid);

`thisTid`は`std.concurrency`組み込みでメッセージパッシングに必要な
現在のスレッドを参照するものです。`spawn`は最初の引数としての関数と、
その関数に向けた追加の引数を引数にとります。

    void foo(Tid parentTid) {
        receive(
          (int i) { writeln("An ", i, " was sent!"); }
        );
        
        send(parentTid, "Done");
    }

`receive`関数は`switch`-`case`のように、他のスレッドから受け取った
値を、受け取った値の型により、渡された`delegates`に送ります。

特定のスレッドにメッセージを送るのには関数`send`とそのスレッドのidを使います:

    send(threadId, 42);

`receiveOnly`を特定の型のみを受け取るのに使えます:

    string text = receiveOnly!string();
    assert(text == "Done");

`receive`ファミリー関数は何かがスレッドのメールボックスに送られてくるまでブロックします。


### 掘り下げる

- [Exchanging Messages between Threads](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=5)
- [Messaging passing concurrency](http://ddili.org/ders/d.en/concurrency.html)
- [Pattern Matching with receive](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=6)
- [Multi-threaded file copying](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=7)
- [Thread Termination](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=8)
- [Out-of-Band Communication](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=9)
- [Mailbox crowding](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=10)

## {SourceCode}

```d
import std.stdio : writeln;
import std.concurrency : receive, receiveOnly,
    send, spawn, thisTid, Tid;

/*
小さなスレッド軍隊用のメッセージとして使われる
カスタム構造体です。
*/
struct NumberMessage {
    int number;
    this(int i) {
        this.number = i;
    }
}

/*
他のスレッドのストップサインとして使われるメッセージです
*/
struct CancelMessage {
}

/// CancelMessageに応答します
struct CancelAckMessage {
}

/*
引数として渡された親idを得る
スレッドワーカーメイン関数。
*/
void worker(Tid parentId)
{
    bool canceled = false;
    writeln("Starting ", thisTid, "...");

    while (!canceled) {
      receive(
        (NumberMessage m) {
          writeln("Received int: ", m.number);
        },
        (string text) {
          writeln("Received string: ", text);
        },
        (CancelMessage m) {
          writeln("Stopping ", thisTid, "...");
          send(parentId, CancelAckMessage());
          canceled = true;
        }
      );
    }
}

void main()
{
    Tid[] threads;
    // 小さな10個のワーカースレッドを生成。
    for (size_t i = 0; i < 10; ++i) {
        threads ~= spawn(&worker, thisTid);
    }

    // 奇数のスレッドは数値を、偶数のスレッドは文字列を取得します!
    foreach(idx, ref tid; threads) {
        import std.string : format;
        if (idx % 2)
            send(tid, NumberMessage(cast(int)idx));
        else
            send(tid, format("T=%d", idx));
    }

    // そしてすべてのスレッドはキャンセルメッセージを受け取ります!
    foreach(ref tid; threads) {
        send(tid, CancelMessage());
    }

    // そしてすべてのスレッドが停止リクエストに応答するのを待ちます
    foreach(ref tid; threads) {
        receiveOnly!CancelAckMessage;
        writeln("Received CancelAckMessage!");
    }
}
```
