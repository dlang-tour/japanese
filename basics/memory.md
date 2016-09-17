# Memory

Dはシステムプログラミング言語であり、したがって手動での管理ができます。
しかし手動でのメモリ管理はエラーを起こしやすい傾向があり、そのため
Dはデフォルトで未使用メモリの解放に**ガベージコレクタ**を使用します。

DはCのようなポインタ型 `T*`を提供します:

    int a;
    int* b = &a; // bにはaのアドレスが入る
    auto c = &a; // ｃはint*でaのアドレスが入る

ヒープ上の新しいメモリブロックは`new`式を使って割り当てます。
これは管理されたメモリへのポインタを返します:

    int* a = new int;

`a`によって参照されるメモリがもはやプログラムのどこからも参照されなくなると、
すぐにガベージコレクタはそれを解放します。

Dには関数のための3つのセキュリティレベルがあります: `@system`、 `@trusted`、そして `@safe`です。
特に指定されない限り、デフォルトは`@system`です。
`@safe`は設計からメモリバグを防ぐDのサブセットです。
`@safe`なコードは`@safe`または`@trusted`な関数のみ呼び出すことができます。
また、`@safe`コード中での明示的なポインタ演算は禁止されています:

    void main() @safe {
        int a = 5;
        int* p = &a;
        int* c = p + 5; // エラー
    }

`@trusted`関数は手動で検証された関数で、SafeDと
根本的でダーティで低レベルな世界を橋渡しします。

### 掘り下げる

* [SafeD](https://dlang.org/safed.html)

## {SourceCode}

```d
import std.stdio;

void safeFun() @safe
{
    writeln("Hello World");
    // GCでメモリを割り当てるのはとても安全です。
    int* p = new int;
}

void unsafeFun()
{
    int* p = new int;
    int* fiddling = p + 5;
}

void main()
{
    safeFun();
    unsafeFun();
}
```
