# スライス

スライスは任意の型`T`に対する型`T[]`のオブジェクトです。
スライスは`T`の配列の部分集合のビューを提供するか、
単に配列全体を指します。
**スライスと動的配列は同じものです。**

スライスは２つのメンバから構成されています。 - 
最初の要素へのポインタとスライスの長さです:

    T* ptr;
    size_t length; // 32bit環境なら符号なし32ビット、64bit環境なら符号なし64ビット

### 新しい割り当てによりスライスを得る

新しい動的配列が作成されるとき、新しく割り当てられたメモリからのスライスが返されます:

    auto arr = new int[5];
    assert(arr.length == 5); // メモリはarr.ptrから参照される

この場合割り当てられたメモリは完璧にガベージコレクタによって管理されます。
返されるスライスは根底にある要素の「ビュー」として振る舞います。

### 既存のメモリからスライスを得る

スライシング演算子を使いすでに存在するメモリと同じ所を指すスライスを得ることもできます。
スライシング演算子は別のスライス、静的配列、`opSlice`が実装されている構造体/クラスや
その他いくつかのエンティティに適用できます。

例として式`origin[start .. end]`の中のスライシング演算子は`origin`の
`start`から`end`の**前**までのすべての要素のスライスを得るのに使われます:

    auto newArr = arr[1 .. 4]; // インデックス4は含まれません
    assert(newArr.length == 3);
    newArr[0] = 10; // arr[1]の別名であるnewArr[0]を変更

そのようなスライスは既存のメモリの新しいビューを生成します。
それは新しいコピーを作り**ません**。
メモリ、またはその**スライス**された部分への参照を持つスライスがもはや存在しないなら、
それはガベージコレクタによって開放されます。

スライスを使うと、（たとえばパーサのような）1つのメモリブロックのみを操作し、
本当に必要な場所のみをスライスするようなものにおいて非常に効率的なコードを書くことができます。
この方法なら、新しいメモリブロックを割り当てる必要はありません。

[前のセクション](basics/arrays)で見たように、`[$]`式は`arr.length`の短縮形です。
そのため`arr[$]`はスライスの最後のひとつ先の要素をインデックスし、
したがって（境界チェックが無効化されていなければ）`RangeError`を生成します。

### 掘り下げる

- [Introduction to Slices in D](http://dlang.org/d-array-article.html)
- [Slices in _Programming in D_](http://ddili.org/ders/d.en/slices.html)

## {SourceCode}

```d
import std.stdio : writeln;

void main()
{
    int[] test = [ 3, 9, 11, 7, 2, 76, 90, 6 ];
    test.writeln;
    writeln("First element: ", test[0]);
    writeln("Last element: ", test[$ - 1]);
    writeln("Exclude the first two elements: ",
        test[2 .. $]);

    writeln("Slices are views on the memory:");
    auto test2 = test;
    auto subView = test[3 .. $];
    test[] += 1; // 各要素を1加算
    test.writeln;
    test2.writeln;
    subView.writeln;

    // 空のスライスを作る
    assert(test[2 .. 2].length == 0);
}
```
