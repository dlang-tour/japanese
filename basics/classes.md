# クラス

DはJavaやC++のようなクラスとインターフェースのサポートを提供します。

任意の`class`型は[`Object`](https://dlang.org/phobos/object.html)を暗黙的に継承します。

    class Foo { } // Objectから継承
    class Bar : Foo { } // BarはFooでもある

Dのクラスは一般的に`new`を使いヒープ上にインスタンス化されます:

    auto bar = new Bar;

クラスオブジェクトは常に参照型であり、`struct`とは異なり値としてコピーされません。

    Bar bar = foo; // barはfooを指す

ガベージコレクタはオブジェクトへの参照が存在しなくなった時にメモリを開放します。

### 継承

基底クラスのメンバ関数がオーバーライドされた時、キーワード`override`を使ってそれを示さなければなりません。
これは関数の意図しないオーバーライドを防止します。

    class Bar : Foo {
        override functionFromFoo() {}
    }

Dでは、クラスはクラスを1つだけ継承できます。

### Finalと抽象メンバ関数

- それをオーバーライドすることを禁止するために、基底クラス内で`final`とマークできます
- それを派生クラスでオーバーライドすることを強制するために、関数を`abstract`として宣言できます
- それをインスタンス化されないことを確かにするために、クラス全体を`abstract`として宣言できます
- `super(..)`は基底コンストラクタを明示的に呼び出すのに使えます

### 同一性のチェック

クラスオブジェクトにおいて、`==`と`!=`演算子はオブジェクトの内容を比較します。
したがって、`null`は内容を持たないため、`null`との比較は不正です。
`is`演算子は同一性(アイデンティティ)を比較します。非同一性の比較には、`e1 !is e2`を使います。

```d
MyClass c;
if (c == null)  // エラー
    ...
if (c is null)  // ok
    ...
```

`struct`オブジェクトではすべてのビットが比較され、その他のオペランドの型において、同一性は等価性と同じです。

### 掘り下げる

- [Classes in _Programming in D_](http://ddili.org/ders/d.en/class.html)
- [Inheritance in _Programming in D_](http://ddili.org/ders/d.en/inheritance.html)
- [Object class in _Programming in D_](http://ddili.org/ders/d.en/object.html)
- [Classes in D spec](https://dlang.org/spec/class.html)

## {SourceCode}

```d
import std.stdio : writeln;

/*
なんにでも使える豪勢な型です
*/
class Any {
    // protectedは継承したクラスからのみ
    // 見ることができます
    protected string type;

    this(string type) {
        this.type = type;
    }

    // ちなみに、publicは暗黙的です
    final string getType() {
        return type;
    }

    // これは抽象メンバ関数のため、別途実装される必要があります!
    abstract string convertToString();
}

class Integer: Any {
    // Integerにしか見ることができません
    private {
        int number;
    }

    // コンストラクタ
    this(int number) {
        // 基底クラスのコンストラクタを呼ぶ
        super("integer");
        this.number = number;
    }

    // これは暗黙的です。そして保護レベルを
    // 指定する別の方法です
    public:

    override string convertToString() {
        import std.conv : to;
        // 変換の十徳ナイフです
        return to!string(number);
    }
}

class Float : Any {
    private float number;

    this(float number) {
        super("float");
        this.number = number;
    }

    override string convertToString() {
        import std.string : format;
        // 精度をコントロールします
        return format("%.1f", number);
    }
}

void main()
{
    Any[] anys = [
        new Integer(10),
        new Float(3.1415f)
        ];

    foreach (any; anys) {
        writeln("any's type = ", any.getType());
        writeln("Content = ",
            any.convertToString());
    }
}
```
