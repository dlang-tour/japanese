# 可変性

Dは静的型付き言語です: 一度変数が宣言されたあと、それ以降その型は変更できません。
これはコンパイラに早期にバグを防止させることを可能にし、コンパイル時に制限を実施します。
優れた型安全は大きなプログラムをより安全に、
より保守性を高く作るというニーズに対してのサポートを提供します。

Dにはいくつか型修飾子がありますが、もっとも一般的に使用されるのは
`const`と`immutable`です。

### `immutable`

静的型付けシステムに加えて、Dは特定のオブジェクトに追加の制約を適用する
型修飾子（時々型コンストラクタとも呼ばれる）を提供します。たとえば
`immutable`なオブジェクトは一度だけ初期化でき、
その後変更できません。

    immutable int err = 5;
    // or: immutable err = 5 and int is inferred.
    err = 5; // won't compile

`immutable`なオブジェクトは定義上変更されないので、
例えば異なるスレッド間で同期なしで安全に共有できます。これは、`immutable`
なオブジェクトが完全にキャッシュされうることも意味します。

### `const`

`const`なオブジェクトも変更ができません。この制限はカレントスコープでのみ有効です。
`const`なポインタは**ミュータブル**または`immutable`なオブジェクトのどちらからでも作ることができます。
これは、カレントスコープでオブジェクトが`const`でも、
違うコンテキストで誰かが変更するかもしれない、ということを意味します。
APIが同じ関数で変更可能と不可能両方のデータを処理するために、
`const`な引数を受け入れそれを変更しないことを保証するのは一般的なことです。

    void foo(const char[] s)
    {
        // コメントアウトしないと、
        // 次の行はエラーになります (constは変更できない):
        // s[0] = 'x';

        import std.stdio : writeln;
        writeln(s);
    }

    // `const`のおかげで、どちらの呼び出しもコンパイルされます:
    foo("abcd"); // stringは変更不可能な配列
    foo("abcd".dup); // .dupは変更可能なコピーを返す

`immutable`と`const`どちらも**推移的**な型修飾子です。つまり、一度`const`が型に適用されると、
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
import std.stdio : writeln;

void main()
{
    /**
    * 変数はデフォルトで変更可能であり、
    * 編集することができます:
    */
    int m = 100; // 変更可能
    writeln("m: ", typeof(m).stringof);
    m = 10; // 良い

    /**
    * 変更可能なメモリを指す:
    */
    // 変更可能なメモリへのconstポインタ
    // は許可されます
    const int* cm = &m;
    writeln("cm: ", typeof(cm).stringof);
    // 定義上、constは変更できません:
    // *cm = 100; // エラー!

    // immutableは変更不可であることが保証されているので、
    // 変更可能なメモリを指すことはできません。
    // immutable int* im = &m; // エラー!

    /**
    * リードオンリーなメモリを指す:
    */
    immutable v = 100;
    writeln("v: ", typeof(v).stringof);
    // v = 5; // エラー!

    // `const`はリードオンリーなメモリを指すことがあり、
    // その`const`もまたリードオンリーです。
    const int* cv = &v;
    writeln("*cv: ", typeof(cv).stringof);
    // *cv = 10; // エラー!
}
```
