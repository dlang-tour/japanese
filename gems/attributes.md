# 属性

Dでは関数は様々な方法で属性付けできます。
2つの組み込み属性と**ユーザ定義属性**を見てみましょう。
最初のチャプターで言及された組み込みの
`@safe`、`@system`、`@trusted`もあります。

### `@property`

`@property`とマークされた関数は外の世界から普通のメンバのように見えます:

    struct Foo {
        @property bar() { return 10; }
        @property bar(int x) { writeln(x); }
    }
    
    Foo foo;
    writeln(foo.bar); // 実際はfoo.bar()が呼ばれている
    foo.bar = 10; // foo.bar(10);が呼ばれる

### `@nogc`

Dコンパイラが`@nogc`とマークされた関数に遭遇した時、
その関数のコンテキスト内でメモリ割り当てが**行われない**ことが保証されます。
`@nogc`関数は他の`@nogc`関数のみを呼ぶことができます。


    void foo() @nogc {
      // エラー:
        auto a = new A;
    }

### ユーザ定義属性 (UDAs)

Dの任意の関数や型はユーザ定義型で属性付けできます:

    struct Bar { this(int x) {} }
    
    struct Foo {
      @("Hello") {
          @Bar(10) void foo() {
            ...
          }
      }
    }

組み込みまたはユーザ定義の任意の型は、関数に属性付けできます。
このサンプルの関数`foo()`は `"Hello"` (`string`型)と
`Bar` (値`10`での`Bar`型)で属性付けされます。関数(もしくは型)
の属性を取得する際は組み込みのコンパイラ**トレイト**
`__traits(getAttributes, Foo)`を使い、これは
[`TypeTuple`](https://dlang.org/phobos/std_typetuple.html)を返します。

UDAはコンパイル時ジェネレータが特定の型に適応するのを助ける
別の次元のユーザ定義型によりジェネリックコードを強化することができます。

### 掘り下げる

- [User Defined Attributes in _Programming in D_](http://ddili.org/ders/d.en/uda.html)
- [Attributes in D](https://dlang.org/spec/attribute.html)

