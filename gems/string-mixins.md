# 文字列Mixin

`mixin`式は任意の式を取りコンパイルし、それに応じて命令を生成します。
これは純粋に**コンパイル時**の機構であり、コンパイル中に利用できる
文字列でのみ働きます。JavaScriptの邪悪な`eval`との比較はかなりアンフェアです。

    mixin("int b = 5;");
    assert(b == 5); // 正しくコンパイルされる

`mixin`は実行時の値に依存せず利用できる限り動的に構築された文字列でも動作します。

`mixin`とともに次のセクションの**CTFE**はソースコード中の文字列として定義された
文法から文法解析器を生成する[Pegged](https://github.com/PhilippeSigaud/Pegged)
のような印象的なライブラリを書くことを可能にします。

### 掘り下げる

- [Mixins in D](https://dlang.org/spec/template-mixin.html)

## {SourceCode}

```d
import std.stdio : writeln;

auto calculate(string op, T)(T lhs, T rhs)
{
    return mixin("lhs " ~ op ~ " rhs");
}

void main()
{
    // これまでにない新しいHello Worldのアプローチ!
    mixin(`writeln("Hello World");`);

    // テンプレートパラメータとして動作する操作を渡す
    writeln("5 + 12 = ", calculate!"+"(5,12));
    writeln("10 - 8 = ", calculate!"-"(10,8));
    writeln("8 * 8 = ", calculate!"*"(8,8));
    writeln("100 / 5 = ", calculate!"/"(100,5));
}
```
