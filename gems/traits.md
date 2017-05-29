# トレイト

Dの武器の1つにコンパイル時間数実行(CTFE)というものがあります。
内省と組み合わせることで、ジェネリックなプログラムを書くことができ、
強力な最適化を受けることができます。

## 明示的契約

トレイトはどのような入力が許されるか明示的に示すことを可能にします。
たとえば`splitIntoWords`は任意の文字列型を操作できます:

```d
S[] splitIntoWord(S)(S input)
if (isSomeString!S)
```

これはテンプレートパラメータにも反映され、`myWrapper`に渡される
シンボルが呼び出し可能な関数であることは保証されます:

```d
void myWrapper(alias f)
if (isCallable!f)
```

シンプルな例として、2つのレンジの共通接頭辞を返す`std.algorithm.searching`の
[`commonPrefix`](https://dlang.org/phobos/std_algorithm_searching.html#.commonPrefix)
を分析します:

```d
auto commonPrefix(alias pred = "a == b", R1, R2)(R1 r1, R2 r2)
if (isForwardRange!R1
    isInputRange!R2 &&
    is(typeof(binaryFun!pred(r1.front, r2.front)))) &&
    !isNarrowString!R1)
```

これはこの関数が以下の条件でのみ呼び出せることを意味します:

- `r1`は保存可能(`isForwardRange`で保証される)
- `r2`は反復処理可能(`isInputRange`で保証される)
- `pred`は`r1`と`r2`の要素の型で呼び出せる
- `r1`はナロー文字列でない(`char[]`、 `string`、 `wchar` または `wstring`) - 簡単に言うと、他のデコーディングが必要

### 特殊化

多くのAPIは汎用を目指します、しかし一般化のために余計な実行時間を使いたくはありません。
内省とCTFEの力で、コンパイル時に与えられた型に対して最高のパフォーマンスを得るために
メソッドを特殊化することが可能になります。

よくある問題として、配列とは異なりたどってみないとストリームやリストの正確な長さがわからないことがあります。
そのため任意の反復処理可能な型に対して一般化したシンプルな実装の`std.range`
のメソッド`walkLength`でこのようにします:

```d
static if (hasMember!(r, "length"))
    return r.length; // O(1)
else
    return r.walkLength; // O(n)
```

#### `commonPrefix`

Phobosにおいてコンパイル時の内省の使用は普遍的です。たとえば`RandomAccessRange`
では場所を飛び越えることができ、アルゴリズムを高速化できるため`commonPrefix`は
`RandomAccessRange`と線形反復処理可能なレンジで異なります。

#### さらなるCTFEマジック

[std.traits](https://dlang.org/phobos/std_traits.html)は`compiles`
のような即座にコンパイルエラーを引き起こすことがあるためラップできないものを除きDの
[traits](https://dlang.org/spec/traits.html)をラップします:

```d
__traits(compiles, obvious error - $%42); // false
```

#### 特殊なキーワード

更にデバッグ用途にDは特殊なキーワードの組を提供します:

```d
void test(string file = __FILE__, size_t line = __LINE__, string mod = __MODULE__,
          string func = __FUNCTION__, string pretty = __PRETTY_FUNCTION__)
{
    writefln("file: '%s', line: '%s', module: '%s',\nfunction: '%s', pretty function: '%s'",
             file, line, mod, func, pretty);
}
```

DのCLI実行では`time`は必要ありません - CTFEが使えます!

```d
rdmd --force --eval='pragma(msg, __TIMESTAMP__);'
```

## 掘り下げる

- [std.range.primitives](https://dlang.org/phobos/std_range_primitives.html)
- [std.traits](https://dlang.org/phobos/std_traits.html)
- [std.meta](https://dlang.org/phobos/std_meta.html)
- [Specification on Traits in D](https://dlang.org/spec/traits.html)

## {SourceCode}

```d
import std.functional : binaryFun;
import std.range.primitives : empty, front,
    popFront,
    isInputRange,
    isForwardRange,
    isRandomAccessRange,
    hasSlicing,
    hasLength;
import std.stdio : writeln;
import std.traits : isNarrowString;

/**
特殊なケースの自動デコーディング無しで
2つのレンジの共通接頭辞を返します。

Params:
    pred = 共通性の比較のための述語
    r1 = 要素のforwardレンジ。
    r2 = 要素のinputレンジ。

Returns:
両方のレンジのはじめの文字列を含むr1のスライス。
 */
auto commonPrefix(alias pred = "a == b", R1, R2)
                 (R1 r1, R2 r2)
if (isForwardRange!R1 && isInputRange!R2 &&
    !isNarrowString!R1 &&
    is(typeof(binaryFun!pred(r1.front,
                             r2.front))))
{
    import std.algorithm.comparison : min;
    static if (isRandomAccessRange!R1 &&
               isRandomAccessRange!R2 &&
               hasLength!R1 && hasLength!R2 &&
               hasSlicing!R1)
    {
        immutable limit = min(r1.length,
                              r2.length);
        foreach (i; 0 .. limit)
        {
            if (!binaryFun!pred(r1[i], r2[i]))
            {
                return r1[0 .. i];
            }
        }
        return r1[0 .. limit];
    }
    else
    {
        import std.range : takeExactly;
        auto result = r1.save;
        size_t i = 0;
        for (;
             !r1.empty && !r2.empty &&
             binaryFun!pred(r1.front, r2.front);
             ++i, r1.popFront(), r2.popFront())
        {}
        return takeExactly(result, i);
    }
}

void main()
{
    // prints: "hello, "
    writeln(commonPrefix("hello, world"d,
                         "hello, there"d));
}
```
