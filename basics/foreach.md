# Foreach

{{#img-right}}dman-teacher-foreach.jpg{{/img-right}}

より間違いにくく、より可読性の高い繰り返しができるDの機能である`foreach`ループです。

### 要素の反復

`int[]`型の配列`arr`があるとき、`foreach`ループを使って要素を
反復処理することができます:

    foreach (int e; arr) {
        writeln(e);
    }

`foreach`の最初のパラメータは繰り返し内で使われる変数名です。
その型は自動的に推論されます:

    foreach (e; arr) {
        // typeof(e) は int
        writeln(e);
    }

2つ目のフィールドは配列、または[次のセクション](basics/ranges)で紹介される
**レンジ**と呼ばれる特殊な反復可能オブジェクトでなければいけません。

### 参照によるアクセス

要素は繰り返しの間に配列またはレンジからコピーされます。
これは基本型においては好ましいことですが、大きな型においては問題かもしれません。
コピーするのを防ぐ、または**インプレース**の変化を有効にするには、`ref`が使えます:

    foreach (ref e; arr) {
        e = 10; // 値を上書きする
    }

### `n`回反復

Dは`n`回実行しなければならない繰り返しを、`..`文によりより簡潔に書くことができます:

    foreach (i; 0..3) {
        writeln(i);
    }
    // 0 1 2

`a..b`の最後の数字はレンジから除外され、したがってループ本文は`3`回実行されます。

### インデックスカウンタ付きの繰り返し

配列において、また別のインデックス変数にアクセスすることもできます。

    foreach (i, e; [4, 5, 6]) {
        writeln(i, ":", e);
    }
    // 0:4 1:5 2:6

### `foreach_reverse`による逆向きの繰り返し

コレクションは`foreach_reverse`により逆順で繰り返すことができます:

    foreach_reverse (e; [1, 2, 3]) {
        writeln(e);
    }
    // 3 2 1

### 掘り下げる

- [`foreach` in _Programming in D_](http://ddili.org/ders/d.en/foreach.html)
- [`foreach` with Structs and Classes  _Programming in D_](http://ddili.org/ders/d.en/foreach_opapply.html)
- [`foreach` specification](https://dlang.org/spec/statement.html#ForeachStatement)

## {SourceCode}

```d
import std.stdio : writefln;

void main() {
    auto arr = [
        [5, 15],      // 20
        [2, 3, 2, 3], // 10
        [3, 6, 2, 9], // 20
    ];

    foreach (i, row; arr)
    {
        double total = 0.0;
        foreach (e; row)
            total += e;

        auto avg = total / row.length;
        writefln("AVG [row=%d]: %.2f", i, avg);
    }
}
```
