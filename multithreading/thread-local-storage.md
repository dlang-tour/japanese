# スレッド局所記憶

ストレージクラス`static`は一度だけ初期化されるオブジェクトの宣言を
可能にします。次に同じ行が実行される時、初期化は省略されます。
各スレッドは自分の`static`オブジェクト(**TLS - スレッド局所記憶**)を得て、
他のスレッドの`static`オブジェクトを読みこんだり変更したりできなくなります - 変数名は
同じままで。従って`static`は**現在の**スレッドのグローバルの状態を保つ
オブジェクトの宣言を可能にします。

これは`static`が実際にアプリケーションでグローバルであることを意味する、
マルチスレッドアプリケーションでの同期の問題を有するC/C++やJavaなどの例とは異なります。

`static`変数に割り当てられる値はコンパイル時に計算可能でなければなりません。
実行時依存は持てません!これは`static`変数を実行時に構造体、クラス、モジュールの
`static this()`ワンタイムコンストラクタを使って初期化することを可能にします。

    static int b = 42;
    // bは一度だけ初期化されます!
    // 異なるスレッドで実行するとき各bは
    // 他のスレッドへのインターフェース
    // のない"自分の"bを見ます

さらにすべてのスレッドが閲覧と修正をできる"クラシックな"グローバル変数の宣言には、
Cの`static`と等価の`__gshared`ストレージクラスを使います。
醜い名前は頻繁に使わないためのフレンドリーな念押しです。

    __gshared int b = 50;
    // これも一度だけ初期化されます!
    // すべてのスレッドが読み込め、
    // それを危険にする変更ができる真にグローバルなbです。

### 掘り下げる

- [Thread-local storage on Wikipedia](https://en.wikipedia.org/wiki/Thread-local_storage)

## {SourceCode}

```d
import std.concurrency : spawn, thisTid;

void worker(bool firstTime)
{
    import std.stdio : writeln;
    // threadStateは現在のスレッドでのみグローバルです。
    // 他のスレッドはアクセスできません。
    // なお、これは最初にその行が実行された時
    // のみ初期化されます。
    static int threadState = 0;
    writeln("Thread ", thisTid,
        ": My state = ", threadState++);
    if (firstTime)
        worker(false);
}

void main()
{
    // それぞれworker(true,i)を呼ぶ
    // 5つのスレッドを作ります。
    for (size_t i = 0; i < 5; ++i) {
        spawn(&worker, true);
    }
}
```
