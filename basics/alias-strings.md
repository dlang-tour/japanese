# エイリアスと文字列

今私達は配列が何かを知っていて、`immutable`に触れて、基本型も見てきているので、
1行に2つの構造を導入してみましょう:

    alias string = immutable(char)[];

`string`という語は`alias`文によって、`immutable(char)`のスライスとして定義されています。
これは、一度`string`が構築されるとその内容が再び変更されることはないということを意味します。
そして実はこれが2つ目に紹介するものです: UTF-8 `string`へようこそ!

その不変性により、`string`は異なるスレッド間で完璧に共有されます。
`string`はスライスなので、メモリ割り当て無しで一部分を取り出すことができます。
例として標準関数
[`std.algorithm.splitter`](https://dlang.org/phobos/std_algorithm_iteration.html#.splitter)
は、メモリ割り当て無しに文字列を改行で分割します。

UTF-8 `string`のほかに、さらに2つの型があります:

    alias wstring = immutable(wchar)[]; // UTF-16
    alias dstring = immutable(dchar)[]; // UTF-32

`std.conv`の`to`メソッドを使うことで各種型は非常に簡単に他のものに変換できます:

    dstring myDstring = to!dstring(myString);
    string myString   = to!string(myDstring);

### ユニコード文字列

普通の`string`は8ビットユニコード[コードユニット](http://unicode.org/glossary/#code_unit)
の配列として定義されています。
文字列にはすべての配列操作が使えますが、それらは文字レベルでなくコードユニットレベルで働きます。
同時に、標準ライブラリのアルゴリズムは文字列を
[コードポイント](http://unicode.org/glossary/#code_point)の列として解釈します。
明示的な[`std.uni.byGrapheme`](https://dlang.org/library/std/uni/by_grapheme.html)の使用により、
それらを[書記素](http://unicode.org/glossary/#grapheme)の列として扱うオプションもあります。

以下の簡単な例で解釈の違いを説明します:

    string s = "\u0041\u0308"; // Ä

    writeln(s.length); // 3

    import std.range : walkLength;
    writeln(s.walkLength); // 2

    import std.uni : byGrapheme;
    writeln(s.byGrapheme.walkLength); // 1

ここで`s`の実際の配列の長さは3で、これは3つのコードユニットを含むからです:
`0x41`、`0x03` そして `0x08` です。これらの後ろの2つは単一のコードポイント
(ダイアクリティカルマーク結合文字)を定義し、
[`walkLength`](https://dlang.org/library/std/range/primitives/walk_length.html)
(任意のレンジの長さを計算する標準ライブラリ関数)は合計2つのコードポイントを数えます。
最後に、`byGrapheme`はこれら2つのコードポイントが1つの表示される文字に組み合わさることを
認識するためにかなり高価な計算を実行します。

ユニコードの正しい処理は非常に複雑になりえますが、大抵の場合、Dデベロッパーは単に`string`変数を
魔法のバイト配列として考え、標準ライブラリのアルゴリズムに頼ることで適切に仕事をこなすことができます。
要素(コードユニット)の列を望む人は、
[`byCodeUnit`](http://dlang.org/phobos/std_utf.html#.byCodeUnit)が使えます。

Dにおけるオートデコーディングは[Unicode gemsチャプター](gems/unicode)で詳細に説明されます。

### 複数行文字列

Dにおける文字列は複数行に続くことができます:

    string multiline = "
    This
    may be a
    long document
    ";

ドキュメント中に引用符がある場合は、Wysiwyg文字列(下記参照)か
[ヒアドキュメント文字列](http://dlang.org/spec/lex.html#delimited_strings)が利用できます。

### Wysiwyg文字列

予約されたシンボルの面倒なエスケープを最小限に抑えるために生文字列を使うことができます。
生文字列はバックティック(`` `... ` ``)かr(aw)プレフィックス(`r" ... "`)を使い宣言することができます。

    string raw  =  `raw "string"`; // raw "string"
    string raw2 = r"raw `string`"; // raw `string`

D言語はさらに多くの文字列を表現する方法を提供しています。
ぜひ[調べて](https://dlang.org/spec/lex.html#string_literals)みてください。

### 掘り下げる

- [Unicode gem](gems/unicode)
- [Characters in _Programming in D_](http://ddili.org/ders/d.en/characters.html)
- [Strings in _Programming in D_](http://ddili.org/ders/d.en/strings.html)
- [std.utf](http://dlang.org/phobos/std_utf.html) - UTF en-/decoding algorithms
- [std.uni](http://dlang.org/phobos/std_uni.html) - Unicode algorithms
- [String Literals in the D spec](http://dlang.org/spec/lex.html#string_literals)

## {SourceCode}

```d
import std.stdio : writeln, writefln;
import std.range : walkLength;
import std.uni : byGrapheme;
import std.string : format;

void main() {
    // formatはprintf風の構文を使って文字列を
    // 生成できます。DはネイティブUTF文字列操作が
    // できます!
    string str = format("%s %s", "Hellö",
        "Wörld");
    writeln("My string: ", str);
    writeln("Array length (code unit count)"
        ~ " of string: ", str.length);
    writeln("Range length (code point count)"
        ~ " of string: ", str.walkLength);
    writeln("Character length (grapheme count)"
        ~ " of string: ",
        str.byGrapheme.walkLength);

    // 文字列はただの配列なので、
    // 配列で可能な操作はここでも可能です!
    import std.array : replace;
    writeln(replace(str, "lö", "lo"));
    import std.algorithm : endsWith;
    writefln("Does %s end with 'rld'? %s",
        str, endsWith(str, "rld"));

    import std.conv : to;
    // Convert to UTF-32
    dstring dstr = to!dstring(str);
    // .. which of course looks the same!
    writeln("My dstring: ", dstr);
}
```
