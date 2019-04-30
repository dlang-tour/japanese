# 基本型

Dはプラットフォームに**関係なく**いつも同じサイズの基本型を数多く提供します。
唯一の例外は可能な限り最高の精度を提供する`real`型です。
アプリケーションが32-bitシステム用にコンパイルされたか、
64-bitシステム用にコンパイルされたかで整数型のサイズに違いはありません。

| type                          | size
|-------------------------------|------------
|`bool`                         | 8-bit
|`byte`, `ubyte`, `char`        | 8-bit
|`short`, `ushort`, `wchar`     | 16-bit
|`int`, `uint`, `dchar`         | 32-bit
|`long`, `ulong`                | 64-bit

#### Floating point types:

| type    | size
|---------|--------------------------------------------------
|`float`  | 32-bit
|`double` | 64-bit
|`real`   | >= 64-bit (一般的には64-bit、しかしIntel x86 32-bit上では80-bit)

`u`プレフィックスは**符号なし**型を示します。`char`はUTF-8 文字に変換され、
`wchar`はUTF-16 文字列で使われ、`dchar`はUTF-32 文字で使われます。

コンパイラはその精度が失われない場合のみ異なる型の変数間の変換を許可します。
ただし、浮動小数点数型間での変換(例えば`double`から`float`)は可能です。

異なる型への変換を`cast(型) myVar`式を使って強制することもできます。
`cast`式は型システムを壊しうるものとして、細心の注意を払って使用する必要があります。

特別なキーワード`auto`は変数を作成し、式の右辺からその型を推定します。
`auto myVar = 7`では`myVar`の型を`int`と推測します。
明示的に指定された型を持つ他の変数と同様、この型は
コンパイル時に設定され、明示的に型を与えられた他の変数と同じように、
以降変更できないことに注意してください。

### 型のプロパティ

すべてのデータ型はそれが初期化された`.init`プロパティを持ちます。
これはすべての整数型で`0`、浮動小数点数で`nan`(**非数**)です。

整数と浮動小数点数型は、表現できる最大値である`.max`プロパティを持ちます。
整数型は表現できる最小値である`.min`プロパティも持ち、一方で浮動小数点数型は最も小さい
正規化された0でない数として定義された`.min_normal`プロパティを持ちます。

浮動小数点数はプロパティ`.nan` (NaN値)、`.infinity` (無限値)、
`.dig` (精度の10進桁数)、`.mant_dig` (仮数部のビット数)なども持ちます。

どの型も文字列としてその名前を生成する`.stringof`プロパティを持ちます。

### Dでのインデックス

Dにおいて、インデックスは普通アドレス可能なすべてのメモリへのオフセットを表現するのに十分大きい型として
エイリアス型`size_t`を持ちます。
これは32-bitアーキテクチャで`uint`、64-bitで`ulong`です。

### Assert式

`assert`はデバッグモードで条件を検証し、それが失敗したら`AssertionError`で中止する式です。
このため`assert(0)`は到達不能コードを示すのに使われます。

### 掘り下げる

#### ベーシック・リファレンス

- [Assignment](http://ddili.org/ders/d.en/assignment.html)
- [Variables](http://ddili.org/ders/d.en/variables.html)
- [Arithmetics](http://ddili.org/ders/d.en/arithmetic.html)
- [Floating Point](http://ddili.org/ders/d.en/floating_point.html)
- [Fundamental types in _Programming in D_](http://ddili.org/ders/d.en/types.html)

#### アドバンスト・リファレンス

- [Overview of all basic data types in D](https://dlang.org/spec/type.html)
- [`auto` and `typeof` in _Programming in D_](http://ddili.org/ders/d.en/auto_and_typeof.html)
- [Type properties](https://dlang.org/spec/property.html)
- [Assert expression](https://dlang.org/spec/expression.html#AssertExpression)

## {SourceCode}

```d
import std.stdio : writeln;

void main()
{
    // 可読性を高めるために大きな数字を
    // アンダースコア"_"で区切ることができます。
    int b = 7_000_000;
    short c = cast(short) b; // キャストが必要
    uint d = b; // 良い
    int g;
    assert(g == 0);

    auto f = 3.1415f; // fはfloatを表す

    // typeid(VAR)は式の型情報を返します。
    writeln("type of f is ", typeid(f));
    double pi = f; // 良い
    // 浮動小数点数型では暗黙的な
    // ダウンキャスティングができます
    float demoted = pi;

    // 型のプロパティにアクセス
    assert(int.init == 0);
    assert(int.sizeof == 4);
    assert(bool.max == 1);
    writeln(int.min, " ", int.max);
    writeln(int.stringof); // int
}
```
