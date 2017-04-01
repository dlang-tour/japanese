# ドキュメント

Dは言語にモダンなソフトウェアエンジニアリングの重要なパーツを統合することを試みます。
**契約プログラミング**や**ユニットテスト**以外にDはソースコードの外にネイティブに
[ドキュメント](https://dlang.org/phobos/std_variant.html)を生成します。

型や関数をドキュメント化するための標準スキーマであるコマンド`dmd -D`
を使うことで便利にコマンドラインでソースファイルをもとにした
HTMLドキュメントを生成します。
実際、[Phobosライブラリドキュメント](https://dlang.org/phobos)
のすべては**DDoc**で生成されています。

下記のコメントスタイルはソースコードのドキュメントに含まれるものとDDocにみなされます:

* `/// 型もしくは関数の前の3つのスラッシュ`
* `/++ 2つの + で複数行コメント +/`
* `/** 2つの * で複数行コメント */`

標準化されたドキュメントセクションについての詳細はソースコードサンプルを見てください。

### 掘り下げる

- [DDoc design](https://dlang.org/spec/ddoc.html)
- [Phobos standard library documentation](https://dlang.org/phobos)

## {SourceCode:incomplete}

```d
/**
  数の平方根を計算します。

  Here could be a longer paragraph that
  elaborates on the great win for
  society for having a function that is actually
  able to calculate a square root of a given
  number.

  Example:
  -------------------
  double sq = sqrt(4);
  -------------------
  Params:
    number = 平方根を計算する数。

  License: 任意の用途に自由に使ってください
  Throws: 何も投げません。
  Returns: 入力の平方根。
*/
T sqrt(T)(T number) {
}
```
