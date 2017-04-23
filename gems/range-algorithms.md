# レンジアルゴリズム

標準モジュール[std.range](http://dlang.org/phobos/std_range.html)
と[std.algorithm](http://dlang.org/phobos/std_algorithm.html)
は、構成要素として**レンジ**を基本にした、より読みやすい方法で複雑な
操作を表現することができる多くの優れた関数を提供します。

これらのアルゴリズムの素晴らしい点はただレンジを定義するだけで
すでにある標準ライブラリから直接利益を受けられるところです。

### std.algorithm

`filter` - テンプレート引数としてラムダを取り、要素をフィルタした新しいレンジを生成します:

    filter!"a > 20"(range);
    filter!(a => a > 20)(range);

`map` - テンプレートパラメータとして定義された述語を使って新しいレンジを生成します:

    [1, 2, 3].map!(x => to!string(x));

`each` - レンジを噛み砕く関数としての手軽な`foreach`です:

    [1, 2, 3].each!(a => writeln(a));

### std.range
`take` - **N**個の要素へ制限します:

    theBigBigRange.take(10);

`zip` - 2つのレンジを同時に処理しその2つからタプルを返します:

    assert(zip([1,2], ["hello","world"]).front
      == tuple(1, "hello"));

`generate` - 関数を取り各反復処理でそれを呼びレンジを生成します、たとえば:

    alias RandomRange = generate!(x => uniform(1, 1000));

`cycle` - 与えられた入力のレンジを永遠に繰り返すレンジを返します。

    auto c = cycle([1]);
    // レンジは空になりません!
    assert(!c.empty);

### ドキュメントはあなたの訪問を待っています!


### 掘り下げる

- [Ranges in _Programming in D_](http://ddili.org/ders/d.en/ranges.html)
- [More Ranges in _Programming in D_](http://ddili.org/ders/d.en/ranges_more.html)

## {SourceCode}

```d
// さあ、全軍あつまれ!
import std.algorithm : canFind, map,
  filter, sort, uniq, joiner, chunkBy, splitter;
import std.array : array, empty;
import std.range : zip;
import std.stdio : writeln;
import std.string : format;

void main()
{
    string text = q{This tour will give you an
overview of this powerful and expressive systems
programming language which compiles directly
to efficient, *native* machine code.};

    // 区切る述語
    alias pred = c => canFind(" ,.\n", c);
    // 優れたアルゴリズムとして
    // メモリ割り当てなしで遅延して動作します!
    auto words = text.splitter!pred
      .filter!(a => !a.empty);

    auto wordCharCounts = words
      .map!"a.count";

    // 単語あたり文字数を優れた方法で
    // 最初から出力します!
    zip(wordCharCounts, words)
      // ソートのため配列に変換します
      .array()
      .sort()
      // 複製は必要ない、そうですね?
      .uniq()
      // 同じ文字数を持つ行をすべて出力します。
      // chunkByは長さでチャンクされた
      // レンジのレンジを返すことでここで役立ちます
      .chunkBy!(a => a[0])
      // これらの要素は一行にまとめられます
      .map!(chunk => format("%d -> %s",
          chunk[0],
          // 単語たち
          chunk[1]
            .map!(a => a[1])
            .joiner(", ")))
      // joinerは結合します、ただし遅延して!
      // そして改行付きのラインです
      .joiner("\n")
      // 標準出力にパイプします
      .writeln();
}
```
