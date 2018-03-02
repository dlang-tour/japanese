# Mir Algorithm

ndsliceという多次元配列のパッケージを提供する、数学や金融のためのD言語における中心的なライブラリです。

## リンク

 - [API Documentation](http://docs.algorithm.dlang.io)
 - [GitHub](https://github.com/libmir/mir-algorithm)
 - [Lubeck](https://github.com/kaleidicassociates/lubeck) - NDSliceのAPIによる線形代数ライブラリ

## 参照

[Magic Square on Wikipedia](https://en.wikipedia.org/wiki/Magic_square).

## {SourceCode}

```d
/+dub.sdl:
dependency "mir-algorithm" version="~>0.7"
+/
import mir.math.sum;
// 多くのAPIは任意のN次元でも動作します
import mir.ndslice: magic, byDim, map, as, each,
    repeat, diagonal, reversed, slice;

import std.stdio: writefln;

void main()
{
    auto n = 3;

    auto matrix = n
        // 遅延評価のもと魔法陣を作成し、
        .magic
        // 要素の型を浮動小数点であるdouble型に変換し、
        .as!double
        // メモリを割り当てて書き換え可能なndsliceを作成します。
        .slice;

    assert (matrix.isMagic);
    "%(%s\n%)\n".writefln(matrix);

    // 対角要素をマイナス化します
    matrix.diagonal.each!"a = -a";
    "%(%s\n%)\n".writefln(matrix);
}

// 引数matrixが魔法陣になっているかチェックします
bool isMagic(S)(S matrix)
{
    auto n = matrix.length;
    auto c = n * (n * n + 1) / 2;// magic number
    return matrix.length!0 > 0   // check shape
        && matrix.length!0 == matrix.length!1
        // 行毎の合計
        && matrix.byDim!0.map!sum == c.repeat(n)
        // 列毎の合計
        && matrix.byDim!1.map!sum == c.repeat(n)
        // 対角の合計
        && matrix.diagonal.sum == c
        // 対角の合計
        && matrix.reversed!1.diagonal.sum == c;
}
```
