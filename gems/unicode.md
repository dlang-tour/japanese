# Dにおけるユニコード

ユニコードはコンピュータ上でのテキスト表現の世界標準です。
Dは言語仕様と標準ライブラリの両方でユニコードに完全対応しています。

## 何を、どうして

コンピュータは、最も低レベルのところでは数値しか扱わないため、テキストとは何かという
概念を持ちません。結果、コンピュータコードはテキストデータをとり、それをバイナリ表現
との間で変換する方法を必要とします。変換の方法を**符号化方式**といい、
ユニコードはそのような方式のひとつです。

例の文字列の根本にある数値表現を見ると、単にコードを実行しています。

ユニコードは世界のすべての言語を同じ符号化方式を使い表現することを可能にする
そのデザインにおいて独特です。ユニコード以前は、コンピュータは異なる会社によって製造されたり
コミュニケーションに苦労する異なる地域で出荷されたりしており、場合によっては符号化方式が
全くサポートされないこともあり、そのコンピュータでテキストを表示することは不可能でした。


ユニコードとその技術的詳細についてのさらなる情報については、「掘り下げる」セクションの
ウィキペディアのユニコードの記事を確認してください。

## どうやって

ユニコードはこれらの問題のほとんどを修正し、あらゆるモダンなマシンでサポートされています。
Dは古い言語の過ちから学び、Dのそのような文字列**すべて**はユニコード文字列であるのに対して、
CやC++のような言語では文字列はただのバイトの配列です。

Dでは`string`、`wstring`、`dstring`はそれぞれUTF-8、UTF-16、UTF-32でエンコードされた
文字列です。これらの文字型は`char`、`wchar`、`dchar`です。

仕様によると、Dの文字列型にユニコードでないデータを格納するとエラーになります。
文字列が不適切にエンコードされた場合プログラムは異なる方法で失敗することを期待します。

文字列エンコーディングを格納するため、またはC・C++の挙動を得るために、`ubyte[]`か`char*`が使えます。

## レンジアルゴリズムでの文字列

**このセクションのために[レンジアルゴリズム](gems/range-algorithms)を読むことを推奨します**

Dのユニコードにおいていくつか念頭に置いておく重要な注意事項があります。

まず、便利な機能として、レンジ関数を使って文字列を反復処理するとき、
Phobosは`string`と`wstrings`の全要素をUTF-32コードポイントにエンコードします。
**オートデコーディング**として知られるこの方法は、このようなことを意味します

```
static assert(is(typeof(utf8.front) == dchar));
```

この振る舞いは多くのことを示唆し、多くの人を混乱させる主なものとしては、
`std.traits.hasLength!(string)`が`False`になります。なぜ?レンジAPIの観点からいうと、
`string`の`length`は**レンジ関数が反復処理をする**要素の数ではなく**文字列内の要素の数**を返すためです。

例から、なぜこれら2つのものが常に等しいとは限らないかを見ることができます。
したがって、Phobosのレンジアルゴリズムは`string`が長さの情報を持たないかのように振る舞います。

オートデコーディングの技術的詳細、そしてそれがあなたのプログラムにとって持つ意味についての
さらなる情報については、「掘り下げる」セクションのリンクを確認してください。

### 掘り下げる

- [Unicode on Wikipedia](https://en.wikipedia.org/wiki/Unicode)
- [Basic Unicode Functions in Phobos](https://dlang.org/phobos/std_uni.html)
- [Tools for Decoding and Encoding UTF in Phobos](https://dlang.org/phobos/std_utf.html)
- [An in Depth Look at Auto Decoding](https://jackstouffer.com/blog/d_auto_decoding_and_you.html)
- [An in Depth Essay on Benefits of Using UTF-8](http://utf8everywhere.org/)

## {SourceCode}

```d
import std.range.primitives : empty,
    front, popFront;
import std.stdio : write, writeln;

void main()
{
    string utf8 = "å ø ∑ 😦";
    wstring utf16 = "å ø ∑ 😦";
    dstring utf32 = "å ø ∑ 😦";

    writeln("utf8 length: ", utf8.length);
    writeln("utf16 length: ", utf16.length);
    writeln("utf32 length: ", utf32.length);

    foreach (item; utf8)
    {
        auto c = cast(ubyte) item;
        write(c, " ");
    }
    writeln();

    // 指定された要素の型はdcharのため、文字列を
    // UTF-32コードポイントにエンコードするために
    // 先読みが使われます。
    // 非文字列では、単純なキャストが使われます
    foreach (dchar item; utf16)
    {
        auto c = cast(ushort) item;
        write(c, " ");
    }
    writeln();

    // オートデコーディングの結果
    static assert(
        is(typeof(utf8[0]) == immutable(char))
    );
    static assert(
        is(typeof(utf8.front) == dchar)
    );
}
```