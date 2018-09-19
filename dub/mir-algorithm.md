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
dependency "mir-algorithm" version="~>2.0"
+/

void main()
{
    import mir.ndslice;

    auto matrix = slice!double(3, 4);
    matrix[] = 0;
    matrix.diagonal[] = 1;

    auto row = matrix[2];
    row[3] = 6;
    // D & C index order
    assert(matrix[2, 3] == 6);

    import std.stdio;
    matrix.writeln;
}
```
