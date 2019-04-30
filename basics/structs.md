# 構造体

Dでは`struct`で複合型もしくはカスタム型を定義できます:

    struct Person {
        int age;
        int height;
        float ageXHeight;
    }

`struct`は(`new`で作成されない限り)常にスタックに構築され、
代入や関数呼び出しの引数では**値として**コピーされます。

    auto p = Person(30, 180, 3.1415);
    auto t = p; // コピー

`struct`型のオブジェクトが新規に作成されるとき、
そのメンバは`struct`内で定義された順に初期化されます。
カスタムコンストラクタは`this(...)`メンバ関数で定義できます。
名前の衝突を回避したいときは、`this`を使って現在のインスタンスに明示的にアクセスできます:

    struct Person {
        this(int age, int height) {
            this.age = age;
            this.height = height;
            this.ageXHeight = cast(float)age * height;
        }
            ...

    Person p = Person(30, 180); // 初期化
    p = Person(30, 180);  // 新しいインスタンスへ代入

`struct`は複数のメンバ関数を持つことができます。
メンバ関数はデフォルトで`public`で、外部からアクセスが可能です。
これは`private`にすることも可能で、その場合同じ`struct`のメンバ関数か、
同じモジュール内のコードからのみ呼び出せるようになります。

    struct Person {
        void doStuff() {
            ...
        private void privateStuff() {
            ...

    p.doStuff(); // メソッド doStuff を呼び出す
    p.privateStuff(); // これは禁止されています

### Constメンバ関数

メンバ関数が`const`で宣言されている場合、そのメンバを変更することはできません。
これはコンパイラにより強制されます。
メンバ関数を`const`にすると、`const`または`immutable`オブジェクトから呼び出せるようになり、
さらにこの関数がオブジェクトの状態を変更しないことが呼び出し元に保証されます。

### Staticメンバ関数

メンバ関数が`static`で宣言されると、
その関数はインスタンス化されたオブジェクトなしに呼び出せるようになります
(例: `Person.myStatic()`)が、非`static`メンバにアクセスできなくなります。
これはメソッドがオブジェクトメンバフィールドにアクセスする必要はないが、
論理的には同じクラスに属するときに使えます。
明示的インスタンスを作ることなく機能を提供するのにも使えます。
例えばシングルトンデザインパターンの実装には`static`が使われることがあります。

### 継承

`struct`は他の`struct`から継承できないことに注意してください。
型ヒエラルキーは、後のセクションで見ていくクラスを用いてのみ構築可能です。
しかし、`alias this`や`mixin`で簡単にポリモーフィックな継承が可能です。

### 掘り下げる

- [Structs in _Programming in D_](http://ddili.org/ders/d.en/struct.html)
- [Struct specification](https://dlang.org/spec/struct.html)

### エクササイズ

`struct Vector3`に以下の関数を実装し、サンプルアプリケーションを正常に実行されるようにしましょう:

* `length()` - ベクトルの長さを返す
* `dot(Vector3)` - 2つのベクトルのドット積を返す
* `toString()` - ベクトルの文字列表現を返す
  関数 [`std.string.format`](https://dlang.org/phobos/std_format.html)
  は`format("MyInt = %d", myInt)`のような`printf`風の構文を使って文字列を返します。
  文字列は後のセクションで詳しく解説します。

## {SourceCode:incomplete}

```d
struct Vector3 {
    double x;
    double y;
    double z;

    double length() const {
        import std.math : sqrt;
        // TODO: Vector3のlengthを実装する
        return 0.0;
    }

    // rhsはコピーされます
    double dot(Vector3 rhs) const {
        // TODO: ドット積を実装する
        return 0.0;
    }
}

void main() {
    auto vec1 = Vector3(10, 0, 0);
    Vector3 vec2;
    vec2.x = 0;
    vec2.y = 20;
    vec2.z = 0;

    // メンバ関数がパラメータを持たないとき、
    // 呼び出しのカッコ () は省略できます
    assert(vec1.length == 10);
    assert(vec2.length == 20);

    // ドット積の機能をテスト
    assert(vec1.dot(vec2) == 0);

    // 1 * 1 + 2 * 1 + 3 * 1
    auto vec3 = Vector3(1, 2, 3);
    assert(vec3.dot(Vector3(1, 1, 1)) == 6);

    // 1 * 3 + 2 * 2 + 3 * 1
    assert(vec3.dot(Vector3(3, 2, 1)) == 10);
}
```
