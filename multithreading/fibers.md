# ファイバー

**ファイバー**は**協調的**なやりかたで並行性を実装する方法です。`Fiber`クラスはモジュール
[`core.thread`](https://dlang.org/phobos/core_thread.html)に定義されています。

ファイバーにやることがないとき、または入力を待っているときに、`Fiber.yield()`
を呼び出すことで積極的に命令を実行できるようにするというのが基本理念です。
親コンテキストがコントロールを得ますがファイバーの状態 - 
スタック上のすべての変数 - は保持されます。ファイバーは再開することができ、
`Fiber.yield()`が呼ばれた直後からの命令を継続します。魔法でしょうか?そのとおり。

    void foo() {
        writeln("Hello");
        Fiber.yield();
        writeln("World");
    }
    // ...
    auto f = new Fiber(&foo);
    f.call(); // Hello と表示
    f.call(); // World と表示

この機能は1つのコアを複数のファイバーが協調的に共有する並行処理を
実装するのに使うことができます。スレッドと比べてファイバーが有利なのは、
コンテキストスイッチングの必要がないためリソース使用量が少ないところです。

この技術の、よりクリーンなコードに繋がると言う面でのファイバーの非常に良い使用例を、
ノンブロッキング(または非同期)I/O操作を実装している
[vibe.d フレームワーク](http://vibed.org)に見ることができます。

### 掘り下げる

- [Fibers in _Programming in D_](http://ddili.org/ders/d.en/fibers.html)
- [Documentation of core.thread.Fiber](https://dlang.org/library/core/thread/fiber.html)

## {SourceCode}

```d
import core.thread : Fiber;
import std.stdio : write;
import std.range : iota;

/**
`range`を反復処理し、関数`Fnc`を各要素xに
適用し、`result`に返します。Fiberは各適用後に譲ります。
*/
void fiberedRange(alias Fnc, R, T)(
    R range,
    ref T result)
{
    for(; !range.empty; range.popFront) {
        result = Fnc(range.front);
        Fiber.yield();
    }
}

void main()
{
    int squareResult, cubeResult;
    // 平方レンジを生成するデリゲートで初期化された
    // ファイバーを生成します。
    auto squareFiber = new Fiber({
        fiberedRange!(x => x*x)(
            iota(1,11), squareResult);
    });
    // ……そしてこちらは立方を生成します!
    auto cubeFiber = new Fiber({
        fiberedRange!(x => x*x*x)(
            iota(1,9), cubeResult);
    });

    // 状態がTERMならファイバーは結び付けられた関数の
    // 実行を終了します。
    squareFiber.call();
    cubeFiber.call();
    while (squareFiber.state
        != Fiber.State.TERM && cubeFiber.state
        != Fiber.State.TERM)
    {
        write(squareResult, " ", cubeResult);
        squareFiber.call();
        cubeFiber.call();
        write("\n");
    }

    // squareFiberはまだ終了していないため実行することができます!
}
```
