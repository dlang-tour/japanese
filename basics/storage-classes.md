# ストレージクラス

Dは静的型付け言語です: 一度変数が宣言されたあと、
それ以降その型は変更できません。これはコンパイラに早期にバグを防止させ、
コンパイル時に制限を実施します。巨大なプログラムをより安全に、
より保守性を高く作るときに必要なサポートを、優れた型安全性は提供します。

### `immutable`

静的型付けシステムに加えて、Dは特定のオブジェクトに追加の
制約を適用するストレージクラスを提供します。たとえば
`immutable`なオブジェクトは一度だけ初期化でき、
その後変更できません。

    immutable int err = 5;
    // または: immutable err = 5 でintが推論されます。
    err = 5; // コンパイルされない

`immutable`なオブジェクトは設計によって変更されないので、
例えば異なるスレッド間で安全に共有できます。これは、`immutable`
なオブジェクトが完全にキャッシュされることを意味します。

### `const`

`const`なオブジェクトも変更ができません。この制限はカレントスコープでのみ有効です。
`const`なポインタは**ミュータブル**または`immutable`なオブジェクトから作ることができます。
これは、カレントスコープでオブジェクトが`const`でも、
将来誰かが変更するかもしれない、ということを意味します。`immutable`
とあればオブジェクトの値が永久に変わらないことを確信するでしょう。
APIが入力を変更しないことを保証するために`const`
なオブジェクトを受け入れるのは一般的なことです。

    immutable a = 10;
    int b = 5;
    const int* pa = &a;
    const int* pb = &b;
    *pa = 7; // 認められない

`immutable`と`const`どちらも**推移的**です。つまり、一度`const`が型に適用されると、
その型のすべてのサブコンポーネントにも再帰的にそれが適用されます。

### 掘り下げる

#### ベーシック・リファレンス

- [Immutable in _Programming in D_](http://ddili.org/ders/d.en/const_and_immutable.html)
- [Scopes in _Programming in D_](http://ddili.org/ders/d.en/name_space.html)

#### アドバンスト・リファレンス

- [const(FAQ)](https://dlang.org/const-faq.html)
- [Type qualifiers D](https://dlang.org/spec/const3.html)

## {SourceCode}

```d
import std.stdio;

void main()
{
    immutable forever = 100;
    // エラー:
    // forever = 5;
    writeln("forever: ",
        typeof(forever).stringof);

    const int* cForever = &forever;
    // エラー:
    // *cForever = 10;
    writeln("const* forever: ",
        typeof(cForever).stringof);

    int mutable = 100;
    writeln("mutable: ",
        typeof(mutable).stringof);
    mutable = 10; // 良い
    const int* cMutable = &mutable; // 良い
    // エラー:
    // *cMutable = 100;
    writeln("cMutable: ",
        typeof(cMutable).stringof);

    // エラー:
    // immutable int* imMutable = &mutable;
}
```
