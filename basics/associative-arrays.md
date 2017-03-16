# 連想配列

Dにはハッシュマップとしても知られる組み込みの**連想配列**があります。
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
import std.stdio : writeln;

/**
与えられたテキストを単語で分割し
それぞれの単語数をマップした連想配列を返します。

Params:
    text = 分割されるテキスト
*/
int[string] wordCount(string text)
{
    // 関数splitterは入力をlazyにレンジに分割します
    import std.algorithm.iteration : splitter;
    import std.string : toLower;

    // 単語でインデックスされ個数を返します
    int[string] words;

    foreach(word; splitter(text.toLower(), " "))
    {
        // 単語が見つかったら単語数をインクリメントします。
        // 整数はデフォルトで0です。
        words[word]++;
    }

    return words;
}

void main()
{
    string text = "D is a lot of fun";

    auto wc = wordCount(text);
    writeln("Word counts: ", wc);

    // 反復処理可能:
    // byKey, byValue, byKeyValue
    foreach (word; wc.byValue)
        writeln(word);
}
```
