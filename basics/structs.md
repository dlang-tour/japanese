# 構造体

Dでは`struct`で複合型またはカスタム型を定義します:

    struct Person {
        int age;
        int height;
        float ageXHeight;
    }

`struct`は(`new`で作成されない限り)スタックに構築され、関数呼び出しの
引数や代入では**値として**コピーされます。

    auto p = Person(30, 180, 3.1415);
    auto t = p; // コピー

新しく`struct`型のオブジェクトが作成されるとき、そのメンバーは
`struct`内で定義された順序で初期化されます。カスタム構造体は`this(...)`
メンバ関数を通して定義するとができます。名前の衝突を回避したいときは、
`this`で明示的に現在のインスタンスにアクセスできます。:

    struct Person {
        this(int age, int height) {
            this.age = age;
            this.height = height;
            this.ageXHeight = cast(float)age * height;
        }
            ...

    Person p(30, 180); // 初期化
    p = Person(30, 180);  // 新しいインスタンスを代入

`struct`には任意の数のメンバ関数が含まれます。それらはデフォルトでは
`public`で、外部からアクセス可能です。They might
as well be `private` and thus only be callable by other
member functions of the same `struct` or other code in the same
module.

    struct Person {
        void doStuff() {
            ...
        private void privateStuff() {
            ...

    p.doStuff(); // doStuffメソッドを呼ぶ
    p.privateStuff(); // 禁止

### Constメンバ関数

メンバ関数が`const`で宣言されている場合、そのメンバを変更することはできません。
これはコンパイラによって強制されます。メンバ関数を`const`にすると、
それを`const`または`immutable`オブジェクトでも呼び出し可能になるだけでなく、
メンバ関数がオブジェクトの状態を変更しないことを呼び出し元に保証します。

### Staticメンバ関数

メンバ関数が`static`として宣言されている場合、たとえば`Person.myStatic()`
のようにインスタンス化されたオブジェクト無しで呼び出せるようになりますが、
非`static`メンバーにはアクセスできません。`static`メンバ関数は現在のインスタンスだけでなく、
`struct`のすべてのインスタンスにアクセスすることができます。
また、利用可能なインスタンスを持たない呼び出し元でメンバ関数が利用可能でなければならない時にも利用できます。
例えば、シングルトン(ただ一つのインスタンスのみが許可される)は`static`を使用しています。

### 継承

`struct`は他の`struct`から継承できないことに注意してください。
型のヒエラルキーはクラスを使ってのみ構築できます。後のセクションでお見せしましょう。
しかし、`alias this`または`mixins`でポリモーフィックの継承を簡単に達成することができます。

### 掘り下げる

- [Structs in _Programming in D_](http://ddili.org/ders/d.en/struct.html)
- [Struct specification](https://dlang.org/spec/struct.html)

### エクササイズ

`struct Vector3`に以下の関数を実装し、サンプルアプリケーションを正常に実行させましょう:

* `length()` - ベクトルの長さを返す
* `dot(Vector3)` - ２つのベクトルのドット積を返す
* `toString()` - ベクトルの文字列表現を返す
  関数[`std.string.format`](https://dlang.org/phobos/std_format.html)
  は`printf`風構文を使った文字列を返します:
  `format("MyInt = %d", myInt)`文字列は後のセクションで詳しく説明します。

## {SourceCode:incomplete}

```d
struct Vector3 {
    double x;
    double y;
    double z;

    double length() const {
        import std.math: sqrt;
        return 0.0;
    }

    // rhsはコピーされる
    double dot(Vector3 rhs) const {
        return 0.0;
    }

    /**
    返値: 特殊なフォーマットの文字列表現。
    出力は1桁の精度に制限されます!
    "x: 0.0 y: 0.0 z: 0.0"
    */
    string toString() const {
        import std.string: format;
        // ヒント: 浮動小数点数の出力に影響を与える
        // 方法については、std.formatのドキュメントを
        // 参照してください。
        return format("");
    }
}

void main() {
    auto vec1 = Vector3(10, 0, 0);
    Vector3 vec2;
    vec2.x = 0;
    vec2.y = 20;
    vec2.z = 0;

    // メンバ関数に引数がないとき、
    // 呼び出しカッコ()は省略できます
    assert(vec1.length == 10);
    assert(vec2.length == 20);

    // ドット積の機能をテストします
    assert(vec1.dot(vec2) == 0);

    // 1 * 1 + 2 * 1 + 3 * 1
    auto vec3 = Vector3(1, 2, 3);
    assert(vec3.dot(Vector3(1, 1, 1) == 6);

    // 1 * 3 + 2 * 2 + 3 * 1
    assert(vec3.dot(Vector3(3, 2, 1) == 10);

    // toString()のおかげでベクトルを
    // writelnで出力できます
    import std.stdio: writeln, writefln;
    writeln("vec1 = ", vec1);
    writefln("vec2 = %s", vec2);

    // 文字列表現をチェックします
    assert(vec1.toString() ==
        "x: 10.0 y: 0.0 z: 0.0");
    assert(vec2.toString() ==
        "x: 0.0 y: 20.0 z: 0.0");
}
```
