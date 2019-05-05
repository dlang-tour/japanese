# 連想配列

Dにはハッシュマップとしても知られる**連想配列**が組み込みで存在します。
キーの型が`string`で値の方が`int`である連想配列は下記のように宣言されます:

    int[string] arr;

値はそのキーによってアクセスでき、したがって設定できます:

    arr["key1"] = 10;

キーが連想配列内にあるかどうかをテストするのに、`in`式が使えます:

    if ("key1" in arr)
        writeln("Yes");

`in`式は値が見つかった時はそのポインタを、そうでなければ`null`ポインタを返します。
したがって存在チェックと書き込みは便利に組み合わせることができます:

    if (auto val = "key1" in arr)
        *val = 20;

存在しないキーへのアクセスは即座にアプリケーションを停止させる
`RangeError`を発生させます。デフォルト値による安全なアクセスには、
`get(key, defaultValue)`が使えます。

連想配列は配列のような`.length`プロパティを持ち、キーによって項目を
削除する`.remove(val)`メンバを持ちます。
特殊な`.byKey`と`.byValue`レンジを探索することは読者への練習問題とします。

### 掘り下げる

- [Associative arrays in _Programming in D_](http://ddili.org/ders/d.en/aa.html)
- [Associative arrays specification](https://dlang.org/spec/hash-map.html)
- [std.array.byPair](http://dlang.org/phobos/std_array.html#.byPair)

## {SourceCode}

```d
import std.array : assocArray;
import std.algorithm.iteration: each, group,
    splitter, sum;
import std.string: toLower;
import std.stdio : writefln, writeln;

void main()
{
    string text = "Rock D with D";

    // 全単語を反復処理しカウント
    int[string] words;
    text.toLower()
        .splitter(" ")
        .each!(w => words[w]++);

    foreach (key, value; words)
        writefln("key: %s, value: %d",
                       key, value);

    // `.keys` と .values` は配列を返します
    writeln("Words: ", words.keys);

    // `.byKey`、 `.byValue`、 `.byKeyValue`
    // は遅延評価される反復可能なレンジを返します
    writeln("# Words: ", words.byValue.sum);

    // 連想配列は`assocArray`に
    // キー・バリュータプルの
    // レンジを渡すことで作れます
    auto array = ['a', 'a', 'a', 'b', 'b',
                  'c', 'd', 'e', 'e'];

    // `.group`は連続した等価な要素を1つの要素と
    // その繰り返し回数のタプルにまとめます
    auto keyValue = array.group;
    writeln("Key/Value range: ", keyValue);
    writeln("Associative array: ",
             keyValue.assocArray);
}
```
