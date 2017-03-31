# opDispatchとopApply

Dは`+`、`-`のようなオペレータ、
または[クラスと構造体](https://dlang.org/spec/operatoroverloading.html)
の呼び出しオペレータ`()`をオーバーライドできます。
2つの特殊なオペレータオーバーロード`opDispatch`
または`opApply`を詳しく紹介していきます。

### opDispatch

`opDispatch`は`struct`または`class`、どちらかの型の
メンバ関数として定義できます。その型への不明なメンバ関数呼び出しは
`opDispatch`に渡され、不明なメンバ関数の名前が`string`
テンプレートパラメータとして渡されます。`opDispatch`は**包括的**
メンバ関数であり、ジェネリックプログラミングの別のレベルを
可能にします - 完璧に**コンパイル時に**!

    struct C {
        void callA(int i, int j) { ... }
        void callB(string s) { ... }
    }
    struct CallLogger(C) {
        C content;
        void opDispatch(string name, T...)(T vals) {
            writeln("called ", name);
            mixin("content." ~ name)(vals);
        }
    }
    CallLogger!C l;
    l.callA(1, 2);
    l.callB("ABC");

### opApply

ユーザ定義**レンジ**を定義する代わりの`foreach`走査を実装する
手段が、`opApply`メンバ関数の実装です。そのような型の`foreach`による反復処理は、
パラメータとしての特殊なデリゲートとともに`opApply`を呼びます:

    class Tree {
        Tree lhs;
        Tree rhs;
        int opApply(int delegate(Tree) dg) {
            if (lhs && lhs.opApply(dg)) return 1;
            if (dg(this)) return 1;
            if (rhs && rhs.opApply(dg)) return 1;
            return 0;
        }
    }
    Tree tree = new Tree;
    foreach(node; tree) {
        ...
    }

コンパイラは`foreach`の本文をオブジェクトが渡される特殊なデリゲートに変形させます。
その唯一無二のパラメータは現在の反復処理の値を含みます。魔法の返値`int`は解釈される必要があり、
それが`0`でないとき反復処理を停止させなければなりません。

### 掘り下げる

- [Operator overloading in _Programming in D_](http://ddili.org/ders/d.en/operator_overloading.html)
- [`opApply` in _Programming in D_](http://ddili.org/ders/d.en/foreach_opapply.html)
- [Operator overloading in D](https://dlang.org/spec/operatoroverloading.html)

## {SourceCode}

```d
/*
Variantは何かの型を含んでいるかもしれないものです:
https://dlang.org/phobos/std_variant.html
*/

import std.variant : Variant;

/*
opDispatchで任意の数のメンバで
埋めることができる型です。JavaScriptの
varのようなものです。
*/
struct var {
    private Variant[string] values;

    @property
    Variant opDispatch(string name)() const {
        return values[name];
    }

    @property
    void opDispatch(string name, T)(T val) {
        values[name] = val;
    }
}

void main() {
    import std.stdio : writeln;

    var test;
    test.foo = "test";
    test.bar = 50;
    writeln("test.foo = ", test.foo);
    writeln("test.bar = ", test.bar);
    test.foobar = 3.1415;
    writeln("test.foobar = ", test.foobar);
    // 存在しないのでエラーになります
    // writeln("test.notthere = ",
    //   test.notthere);
}
```
