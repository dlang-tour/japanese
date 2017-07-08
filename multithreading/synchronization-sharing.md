# 同期と共有

`immutable`データを信頼して、メッセージパッシングを使ってスレッドを同期するのが
Dでのマルチスレッディングの好ましいやり方ですが、この言語は型システムサポートと同じように、
複数のスレッドからアクセスされるオブジェクトにマークする`shared`による
プリミティブな**同期**の組み込みサポートを持ちます。

`shared`型識別子は変数が異なるスレッド間で共有されるとマークします:

    shared(int)* p = new int;
    int* t = p; // エラー

たとえば`std.concurrency.send`は`immutable`または`shared`のデータ
のみを送り、送られるメッセージをコピーします。`shared`は推移的で、従って`class`または`struct`
が`shared`マークされた時、そのすべてのメンバもマークされます。`static`変数は**スレッド局所記憶**
(TLS)を使って実装されており、各スレッドは自分専用の変数を得るためデフォルトでは
`shared`ではないことに注意してください。

`synchronized`ブロックでコンパイラに、一度に1つのスレッドのみが入れる
クリティカルセクションを作るよう指示できます。

    synchronized {
        importStuff();
    }

`class`メンバ関数の中でこれらのブロックは競合を減らすために`synchronized(member1, member2)`
によって異なるメンバオブジェクト**ミューテックス**に制限されることがあります。Dコンパイラは自動的に
**クリティカルセクション**を挿入します。クラス全体も同じように`synchronized`としてマークされ、
コンパイラは一度に1スレッドのみがその実際のインスタンスにアクセスするようにします。

`shared`変数の不可分操作は`core.atomic.atomicOp`ヘルパを使って行うことができます:

    shared int test = 5;
    test.atomicOp!"+="(4);

### 掘り下げる

- [Data Sharing Concurrency in _Programming in D_](http://ddili.org/ders/d.en/concurrency_shared.html)
- [`shared` type qualifier](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=11)
- [Lock-Based Synchronization with `synchronized`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=13)
- [Deadlocks and `synchronized`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=15)
- [`synchronized` specification](https://dlang.org/spec/statement.html#SynchronizedStatement)
- [Implicit conversions with `shared` data types](https://dlang.org/spec/const3.html#implicit_conversions)

## {SourceCode}

```d
import std.concurrency : receiveOnly, send,
    spawn, Tid, thisTid;
import core.atomic : atomicOp, atomicLoad;

/*
異なるスレッド間で安全に使えるキューです。
インスタンスへのすべてのアクセスは
synchronizedキーワードにより自動的に
ロックされます。
*/
synchronized class SafeQueue(T)
{
    // 注意: synchronizedクラス内では
    // privateでなければDは文句を言います。
    private T[] elements;

    void push(T value) {
        elements ~= value;
    }

    /// キューが空ならT.initを返します
    T pop() {
        import std.array : empty;
        T value;
        if (elements.empty)
            return value;
        value = elements[0];
        elements = elements[1 .. $];
        return value;
    }
}

/*
並行するスレッドの数に関係なく安全にメッセージを
プリントします。可変引数がargsとして使われていることに
注意してください!argsは0 .. N個の引数になります。
*/
void safePrint(T...)(T args)
{
    // 同時に1つしか実行されません
    synchronized {
        import std.stdio : writeln;
        writeln(args);
    }
}

void threadProducer(shared(SafeQueue!int) queue,
    shared(int)* queueCounter)
{
    import std.range : iota;
    // 1から11の値をプッシュ
    foreach (i; iota(1,11)) {
        queue.push(i);
        safePrint("Pushed ", i);
        atomicOp!"+="(*queueCounter, 1);
    }
}

void threadConsumer(Tid owner,
    shared(SafeQueue!int) queue,
    shared(int)* queueCounter)
{
    int popped = 0;
    while (popped != 10) {
        auto i = queue.pop();
        if (i == int.init)
            continue;
        ++popped;
        // queueCounterの現在の値をatomicLoadを
        // 使って安全に取得します
        safePrint("Popped ", i,
            " (Consumer pushed ",
            atomicLoad(*queueCounter), ")");
    }

    // 終わりです!
    owner.send(true);
}

void main()
{
    auto queue = new shared(SafeQueue!int);
    shared int counter = 0;
    spawn(&threadProducer, queue, &counter);
    auto consumer = spawn(&threadConsumer,
                    thisTid, queue, &counter);
    auto stopped = receiveOnly!bool;
    assert(stopped);
}
```
