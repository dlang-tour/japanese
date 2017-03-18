# レンジ

コンパイラが`foreach`に遭遇した時、

    foreach(element; range) {

内部で下のものと同じように書き換えられます:

    for (; !range.empty; range.popFront()) {
        auto element = range.front;
        ...

上記のインターフェースを満たすオブジェクトは**レンジ**と呼ばれ、反復処理ができる型です:

    struct Range {
        @property empty() const;
        void popFront();
        T front();
    }

`std.range`と`std.algorithm`の関数はこのインターフェースに使う構成要素を提供します。
レンジは簡単に反復処理可能なオブジェクトを背景とした複雑なアルゴリズムを
組み合わせることを可能にします。さらにレンジは例えば次の要素にアクセスされた時など、
反復処理内で本当に必要なときだけ計算を行う**遅延**オブジェクトを作ります。
特殊なレンジのアルゴリズムは後ほど[D's Gems](gems/range-algorithms)
セクションで提示されます。

### エクササイズ

[フィボナッチ数列](https://en.wikipedia.org/wiki/Fibonacci_number)
の数を返す`FibonacciRange`を作りソースコードを完成させましょう。
`assert`を削除してごまかさないでくださいよ!

### 掘り下げる

- [`std.algorithm`](http://dlang.org/phobos/std_algorithm.html)
- [`std.range`](http://dlang.org/phobos/std_range.html)

## {SourceCode:incomplete}

```d
import std.stdio : writeln;

struct FibonacciRange
{
    bool empty() const @property
    {
        // フィボナッチ数列が終わる時がありますか?!
    }

    void popFront()
    {
    }

    int front() @property
    {
    }
}

void main() {
    import std.range : take;
    import std.array : array;

    FibonacciRange fib;

    // `take`は最大N個の要素を返す別のレンジを作ります。
    // このレンジは_遅延評価_され、本当に必要な時しか
    // オリジナルのレンジにさわりません
    auto fib10 = take(fib, 10);

    // しかし我々はすべての要素に触れて
    // レンジを整数の配列に変換したいです。
    int[] the10Fibs = array(fib10);

    writeln("The 10 first Fibonacci numbers: ",
        the10Fibs);
    assert(the10Fibs ==
        [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]);
}
```
