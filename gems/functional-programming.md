# 関数型プログラミング

Dは**関数型プログラミング**に重点を置き、関数型スタイルでの開発の
ファーストクラスのサポートを提供します。

Dで関数は同じ入力パラメータが与えられたなら、
常に**同じ**出力が生成されることを意味する`pure`として宣言できます。
`pure`な関数は可変のグローバルな状態にアクセスまたは変更ができず、
従って他の`pure`な関数のみを呼ぶことが許可されます。

    int add(int lhs, int rhs) pure {
        impureFunction(); // ERROR: unable to call impureFunction here
        return lhs + rhs;
    }

この`add`の変種はその入力パラメータのみに依存した結果を、
それらのパラメータを変更することなく返すため**強い純粋関数**と呼ばれます。
Dは変更可能なパラメータを持つことがある**弱い純粋関数**のパラメータ定義も可能です:

    void add(ref int result, int lhs, int rhs) pure {
        result = lhs + rhs;
    }

これらの関数は純粋で、変更可能なグローバルの状態にアクセス、変更しないと考えられます。
変更可能なパラメータは変更されるかもしれません。

`pure`による制約により、純粋関数はマルチスレッディング環境において**設計によって**
データ競合を防止するのに理想的です。加えて純粋関数は簡単にキャッシュが可能であり、
様々なコンパイラ最適化を可能にします。

属性`pure`はテンプレート化された関数や`auto`関数の適用可能なところで、
自動的にコンパイラによって推論されます
(`@safe`、`@safe`や`@nogc`においても同じことが言えます)。

### 掘り下げる

- [Functional DLang Garden](https://garden.dlang.io/)

## {SourceCode}

```d
import std.bigint : BigInt;

/**
 * 基数の累乗を指数で計算します。
 *
 * Returns:
 *     任意の大きさの整数として累乗の結果を返します
 */
BigInt bigPow(uint base, uint power) pure
{
    BigInt result = 1;

    foreach (_; 0 .. power)
        result *= base;

    return result;
}

void main()
{
    import std.datetime.stopwatch : benchmark;
    import std.functional : memoize,
        reverseArgs;
    import std.stdio : writefln, writeln;

    // memoizeは入力パラメータに依存する
    // 関数呼び出しの結果をキャッシュします。
    // 純粋関数はそれに最適です!
    alias fastBigPow = memoize!(bigPow);

    void test()
    {
        writefln(".uintLength() = %s ",
        	   fastBigPow(5, 10000).uintLength);
    }

    foreach (_; 0 .. 10)
        ( benchmark!test(1)[0]
            .total!"usecs"/1000.0 )
            .reverseArgs!writefln
                (" took: %.2f miliseconds");
}
```
