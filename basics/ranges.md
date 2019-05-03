# レンジ

コンパイラが`foreach`に遭遇した時、

```
foreach (element; range)
{
    // Loop body...
}
```

内部では下のものと同じように書き換えられます:

```
for (auto __rangeCopy = range;
     !__rangeCopy.empty;
     __rangeCopy.popFront())
 {
    auto element = __rangeCopy.front;
    // ループ本文...
}
```

下記のインターフェースを満たす任意のオブジェクトは**レンジ**(もしくはより明確に`InputRange`)
と呼ばれ、これは反復処理のできる型です:

```
    interface InputRange(E)
    {
        bool empty();
        E front();
        void popFront();
    }
```
右の例を見てインプットレンジの実装と使用法をより詳しく調べてみましょう。

## 遅延評価

レンジは**遅延評価**をします。
必要になるまで評価は行われません。
そのため、無限レンジからレンジを得ることができます:

```d
42.repeat.take(3).writeln; // [42, 42, 42]
```

## 値型と参照型

レンジオブジェクトが値型のとき、
レンジはコピーされてコピーだけが消費されます:

```d
auto r = 5.iota;
r.drop(5).writeln; // []
r.writeln; // [0, 1, 2, 3, 4]
```

レンジオブジェクトが参照型
(たとえば`class`や[`std.range.refRange`](https://dlang.org/phobos/std_range.html#refRange))
のとき、レンジは消費されて元に戻りません:

```d
auto r = 5.iota;
auto r2 = refRange(&r);
r2.drop(5).writeln; // []
r2.writeln; // []
```

### 複製可能`InputRange`は`ForwardRange`

標準ライブラリに存在するレンジの殆どは構造体であり、
`foreach`による反復は保証はされないもののふつう非破壊的です。
それが保証されていることが重要なら、
`InputRange`の特殊化であり`.save`メソッドを持つ**フォワード**レンジが利用できます:

```
interface ForwardRange(E) : InputRange!E
{
    typeof(this) save();
}
```

```d
// 値 (構造体)
auto r = 5.iota;
auto r2 = refRange(&r);
r2.save.drop(5).writeln; // []
r2.writeln; // [0, 1, 2, 3, 4]
```

### `ForwardRanges`はバイディレクショナルレンジとランダムアクセスレンジに拡張可能

複製可能な`ForwardRange`には2つの拡張があります。
1つ目はバイディレクショナルレンジ、2つ目はランダムアクセスレンジです。
バイディレクショナルレンジは後ろからの反復が可能です:

```d
interface BidirectionalRange(E) : ForwardRange!E
{
     E back();
     void popBack();
}
```

```d
5.iota.retro.writeln; // [4, 3, 2, 1, 0]
```

ランダムアクセスレンジは`length`を持ち、各要素に直接アクセスできます。

```d
interface RandomAccessRange(E) : ForwardRange!E
{
     E opIndex(size_t i);
     size_t length();
}
```

Dの配列は最もよく知られたランダムアクセスレンジです:

```d
auto r = [4, 5, 6];
r[1].writeln; // 5
```

### 遅延レンジアルゴリズム

[`std.range`](http://dlang.org/phobos/std_range.html)や
[`std.algorithm`](http://dlang.org/phobos/std_algorithm.html)
の関数はこのインターフェースを利用する素材を提供します。
レンジによって容易に反復操作のできるオブジェクトの背後に複雑なアルゴリズムの構築が可能となります。
さらに、レンジは反復の中でそれが本当に必要になったとき、
例えばレンジの次の要素がアクセスされるときにのみ計算を行う**遅延**オブジェクトを作ることができます。
後の[応用](gems/range-algorithms)セクションでは特殊なレンジアルゴリズムを紹介しましょう。

### 掘り下げる

- [`std.algorithm`](http://dlang.org/phobos/std_algorithm.html)
- [`std.range`](http://dlang.org/phobos/std_range.html)

## {SourceCode}

```d
import std.stdio : writeln;

struct FibonacciRange
{
    // フィボナッチ数ジェネレータの状態
    int a = 1, b = 1;

    // フィボナッチ数は終わりません
    enum empty = false;

    // 先頭の要素を取得
    int front() const @property
    {
        return a;
    }

    // 先頭の要素を削除
    void popFront()
    {
        auto t = a;
        a = b;
        b = t + b;
    }
}

void main()
{
    FibonacciRange fib;

    import std.range : drop, generate, take;
    import std.algorithm.iteration :
        filter, sum;

    // フィボナッチ数最初の10項を選択
    auto fib10 = fib.take(10);
    writeln("Fib 10: ", fib10);

    // 最初の5項を除く
    auto fib5 = fib10.drop(5);
    writeln("Fib 5: ", fib5);

    // 偶数の部分集合を選択Select the even subset
    auto fibEven = fib5.filter!(x => x % 2);
    writeln("FibEven : ", fibEven);

    // 全要素を合計
    writeln("Sum of FibEven: ", fibEven.sum);

    // 通常このようにまとめられます:
    fib.take(10)
         .drop(5)
         .filter!(x => x % 2)
         .sum
         .writeln;
}
```
