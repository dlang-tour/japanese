# Mir

パッケージは以下を含んでいます。

 - [mir-algorithm package](dub/mir-algorithm) ndsliceという多次元配列のパッケージを提供する、数学や金融のためのD言語における中心的なライブラリです。
 - [mir-random package](dub/mir-random) 高度な乱数生成器です。
 - 疎なテンソル(Sparse tensors)
 - 組み合わせ計算(Combinatorics)
 - Hoffman

## Links

 - [Mir Algorithm API](http://docs.algorithm.dlang.io)
 - [Mir Random API](http://docs.random.dlang.io)
 - [Mir API](http://docs.mir.dlang.io)
 - [GitHub](https://github.com/libmir/mir)
 - [Lubeck](https://github.com/kaleidicassociates/lubeck) - NDSlice APIによる線形代数ライブラリです。

## {SourceCode}

```d
/+dub.sdl:
dependency "mir" version="~>2.0"
+/
import std.stdio;
import mir.combinatorics;
void main(string[] args)
{
    writeln([1, 2].combinations);
}
```
