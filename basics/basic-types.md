# 基本型

Dはプラットフォームに**関係なく**いつも同じサイズの多くの基本型を提供します。 - 
唯一の例外は可能な限り最高の精度を提供する`real`型です。
アプリケーションが32-bitシステム用にコンパイルされたか、
64-bitシステム用かと整数型のサイズは関係ありません。

<table class="table table-hover">
<tr><td width="250px"><code class="prettyprint">bool</code></td> <td>8-bit</td></tr>
<tr><td><code class="prettyprint">byte, ubyte, char</code></td> <td>8-bit</td></tr>
<tr><td><code class="prettyprint">short, ushort, wchar</code></td> <td>16-bit</td></tr>
<tr><td><code class="prettyprint">int, uint, dchar</code></td> <td>32-bit</td></tr>
<tr><td><code class="prettyprint">long, ulong</code></td> <td>64-bit</td></tr>
</table>

#### 浮動小数点型:

<table class="table table-hover">
<tr><td width="250px"><code class="prettyprint">float</code></td> <td>32-bit</td></tr>
<tr><td><code class="prettyprint">double</code></td> <td>64-bit</td></tr>
<tr><td><code class="prettyprint">real</code></td> <td>プラットフォームに依存、Intel x86 32-bitでは80-bit</td></tr>
</table>

`u`プレフィックスは**符号なし型**を示します。`char`はUTF-8 文字に変換され、
`wchar`はUTF-16 文字列で使われ、`dchar`はUTF-32 文字で使われます。

異なる型の変数間の変換はその精度が失われない場合のみコンパイラに許可されます。
浮動小数点間の変換(例えば`double`から`float`)は許可されます。

異なる型への変換を`cast(型) myVar`式を使って強制することもできます。
`cast`式は型システムを壊しうるものとして、細心の注意を払って使用する必要があります。

特別なキーワード`auto`は変数を作成し、式の右辺からその型を推定します。
`auto myVar = 7`では`myVar`の型を`int`と推測します。
型はコンパイル時に設定され変更できないことに注意してください。 - 
ちょうど明示的に指定された型を持つ他の変数と同様に。

### 型のプロパティ

すべてのデータ型はそれが初期化された`.init`プロパティを持ちます。
これはすべての整数型で`0`、浮動小数点数で`nan`(**数値でない**)です。
整数と浮動小数点数型は、表現できる最低値と最大値である`.min`と`.max`プロパティを持ちます。
浮動小数点値はさらに複数のプロパティ
`.nan` (NaN値)、`.infinity` (無限値)、`.dig` (精度の10進桁数)、`.mant_dig` (仮数部のビット数)などを持ちます。

また、すべての型はその型の名前の文字列を得る`.stringof`プロパティを持ちます。

### Dでのインデックス

Dにおいてインデックスは通常、アドレスを取ることが可能なメモリのオフセットを表現するのに十分なサイズを持つ型のエイリアスとして`size_t`があります - 
32-bitなら`uint`、64-bit アーキテクチャなら`ulong`です。

`assert`とはデバッグモードで条件式を検証し、失敗した時は`AssertionError`で中断するという組み込み機能です。

### 掘り下げる

#### ベーシック・リファレンス

- [Assignment](http://ddili.org/ders/d.en/assignment.html)
- [Variables](http://ddili.org/ders/d.en/variables.html)
- [Arithmetics](http://ddili.org/ders/d.en/arithmetic.html)
- [Floating Point](http://ddili.org/ders/d.en/floating_point.html)
- [Fundamental types in _Programming in D_](http://ddili.org/ders/d.en/types.html)

#### アドバンスド・リファレンス

- [Overview of all basic data types in D](https://dlang.org/spec/type.html)
- [`auto` and `typeof` in _Programming in D_](http://ddili.org/ders/d.en/auto_and_typeof.html)
- [Type properties](https://dlang.org/spec/property.html)

## {SourceCode}

```d
import std.stdio;

void main()
{
    // 大きな数字は可読性を高めるために
    // アンダースコア"_"で区切ることができます。
    int b = 7_000_000;
    short c = cast(short) b; // キャストが必要
    uint d = b; // 良い
    int g;
    assert(g == 0);

    auto f = 3.1415f; // fはfloatを表す

    // typeid(変数)は式の型情報を返します。
    writeln("type of f is ", typeid(f));
    double pi = f; // 良い
    // 浮動小数点数型は
    // 暗黙のダウンキャスティングができます。
    float demoted = pi;

    // 型の情報にアクセス
    assert(int.init == 0);
    assert(int.sizeof == 4);
    assert(bool.max == 1);
    writeln(int.min, " ", int.max);
    writeln(int.stringof); // int
}
```
