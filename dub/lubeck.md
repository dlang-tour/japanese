# Lubeck

CBLAS、LAPACK、Mir Algorithmによる高レベルな線形代数のライブラリです。

## 依存関係
LubeckはCBLASとLAPACKのAPIに依存しています。環境によってはまずそれらをインストールし、`dub.sdl`を更新する必要があるかもしれません。
MacOSは、CBLASとLAPACKが最初からインストールされています。
LinuxやWindowsでは、バックエンドとして [OpenBLAS](http://www.openblas.net) か [Intel MKL](https://software.intel.com/en-us/mkl) の利用を推奨しています。

## リンク

 - [GitHub](https://github.com/kaleidicassociates/lubeck)
 - [Mir Algorithm API](http://docs.algorithm.dlang.io)
 - [Mir Random API](http://docs.algorithm.dlang.io)
 - [Mir API](http://docs.mir.dlang.io)

## {SourceCode:incomplete}

```d
/+dub.sdl:
dependency "lubeck" version="~>1.0"
+/
import std.stdio;
import mir.ndslice: magic, repeat, as, slice;
import lubeck: mtimes;

void main()
{
    auto n = 5;
    // 魔法陣
    auto matrix = n.magic.as!double.slice;
    // [1 1 1 1 1]
    auto vec = 1.repeat(n).as!double.slice;
    // 乗算のためにCBLASを使用
    matrix.mtimes(vec).writeln;
    matrix.mtimes(matrix).writeln;
}
```
