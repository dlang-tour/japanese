# 統一関数呼び出し構文 (UFCS)

**UFCS**はDの重要な機能で、明確なカプセル化によりコードに
再利用性とスケーラビリティを与えます。

UFCSにより`a`に属していない関数の呼び出し`fun(a)`をメンバ関数の呼び出し
`a.fun()`として書くことができます。

コンパイラが`a.fun()`を見て、その型が`fun()`と呼ばれるメンバ関数を
持たない時、最初の引数が`a`の型にマッチするグローバル関数を探します。

この機能は複雑な関数呼び出しを連鎖させる時に特に便利です。こう書く代わりに

    foo(bar(a))

こう書くことができます。

    a.bar().foo()

加えて、Dでは引数のない関数に括弧を使う必要はなく、
これは**任意の**関数をプロパティのように使えるということを意味します:

    import std.uni : toLower;
    "D rocks".toLower; // "d rocks"

UFCSは複雑な操作を行うために複数のアルゴリズムを組み合わせて**レンジ**
に対応する時に特に重要で、明確で管理しやすいコードを実現します。

    import std.algorithm : group;
    import std.range : chain, retro, front, dropOne;
    [1, 2].chain([3, 4]).retro; // 4, 3, 2, 1
    [1, 1, 2, 2, 2].group.dropOne.front; // tuple(2, 3u)

### 掘り下げる

- [UFCS in _Programming in D_](http://ddili.org/ders/d.en/ufcs.html)
- [_Uniform Function Call Syntax_](http://www.drdobbs.com/cpp/uniform-function-call-syntax/232700394) by Walter Bright
- [`std.range`](http://dlang.org/phobos/std_range.html)

## {SourceCode}

```d
import std.stdio : writefln, writeln;
import std.algorithm.iteration : filter;
import std.range : iota;

void main()
{
    "Hello, %s".writefln("World");

    10.iota // 0から9までの数値を返す
      // 偶数をフィルタする
      .filter!(a => a % 2 == 0)
      .writeln(); // 標準出力に書き出す

    // 伝統的スタイル:
    writeln(filter!(a => a % 2 == 0)
                   (iota(10)));
}
```
