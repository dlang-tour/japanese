# デリゲート

### 引数としての関数

関数は他の関数の引数にもなれます:

    void doSomething(int function(int, int) doer) {
        // 渡された関数を呼ぶ
        doer(5,5);
    }

    doSomething(add); // ここでグローバル関数`add`を使う
                      // addは2つのintの引数をもつ必要があります

そして`doer`は他の普通の関数のように呼びだされます。

### 文脈のあるローカル関数

上記のサンプルは、グローバル関数へのポインタである`function`型を使っています。
メンバ関数またはローカル関数が参照され次第、`delegate`は使用されなければなりません。
それは関数ポインタに加えてその文脈 - または**それを作った関数(enclosure)** - の
情報を含むもので、したがって他の言語では**クロージャ**とも呼ばれます。
例えばクラスのメンバ関数を指す`delegate`はクラスオブジェクトのポインタを含みます。
ネストした関数によって作られた`delegate`は代わりに囲まれた文脈へのリンクを含みます。
しかし、それがメモリ安全のために必要ならば、Dコンパイラは自動的にヒープ上に
文脈のコピーを作ることもあり、そのときデリゲートはこのヒープ領域へリンクします。

    void foo() {
        void local() {
            writeln("local");
        }
        auto f = &local; // fはdelegate()型
    }

`delegate`をとる同様の関数`doSomething`はこのようになります:

    void doSomething(int delegate(int,int) doer);

`delegate`と`function`オブジェクトは混ざることができません。しかし標準関数
[`std.functional.toDelegate`](https://dlang.org/phobos/std_functional.html#.toDelegate)
は`function`を`delegate`に変換します。

### 無名関数とラムダ

関数を変数として保持して他の関数に渡すことができるため、それらに独自の名前を与えて定義するのは面倒です。
従ってDは名無し関数と1行の**ラムダ**が使えます。

    auto f = (int lhs, int rhs) {
        return lhs + rhs;
    };
    auto f = (int lhs, int rhs) => lhs + rhs; // ラムダ - 内部で上のように変換されます

テンプレート引数からDの標準関数の機能部品として文字列を渡すこともできます。
例えば畳み込み(folding。reducerとしても知られる)を定義する便利な方法を提供します:

    [1, 2, 3].reduce!`a + b`; // 6

文字列関数は**1つまたは2つ**の引数のみが可能で、その時1番目に`a`、2番目の引数に`b`を使います。

### 掘り下げる

- [Delegate specification](https://dlang.org/spec/function.html#closures)

## {SourceCode}

```d
import std.stdio;

enum IntOps {
    add = 0,
    sub = 1,
    mul = 2,
    div = 3
}

/**
数学の計算を提供します
Params:
    op = 選択された演算機能
Returns: 演算を行うデリゲート
*/
auto getMathOperation(IntOps op)
{
    // 4つの数学的操作のための4つのラムダ関数を定義します
    auto add = (int lhs, int rhs) => lhs + rhs;
    auto sub = (int lhs, int rhs) => lhs - rhs;
    auto mul = (int lhs, int rhs) => lhs * rhs;
    auto div = (int lhs, int rhs) => lhs / rhs;

    // switchがすべてのケースをカバーするようにできます。
    final switch (op) {
        case IntOps.add:
            return add;
        case IntOps.sub:
            return sub;
        case IntOps.mul:
            return mul;
        case IntOps.div:
            return div;
    }
}

void main()
{
    int a = 10;
    int b = 5;

    auto func = getMathOperation(IntOps.add);
    writeln("The type of func is ",
        typeof(func).stringof, "!");

    // すべての私達の仕事を行うデリゲートfuncを実行します!
    writeln("result: ", func(a, b));
}
```
