# テンプレートメタプログラミング

C++の**テンプレートメタプログラミング**に触れたことがあるなら、
Dの提供するあなたの生活をより簡単にするツールに安心できるでしょう。
テンプレートメタプログラミングは意思決定をテンプレート型プロパティに依存するようにする技術で、
それがインスタンス化される時の型をもとにジェネリック型をより一層フレキシブルにします。

### `static if` & `is`

普通の`if`のように、`static if`はコンパイル時に計算できる条件をもとに
条件付きでコードブロックをコンパイルします:

    static if(is(T == int))
        writeln("T is an int");
    static if (is(typeof(x) :  int))
        writeln("Variable x implicitely converts to int");

[`is`式](http://wiki.dlang.org/Is_expression)は条件をコンパイル時に計算する
ジェネリックヘルパです。

    static if(is(T == int)) { // Tはテンプレートパラメータ
        int x = 10;
    }

条件が`true`の場合中括弧は無視されます - 新しいスコープは作られません。
`{ {`と`} }`は新しいブロックを明示的に作ります。

`static if`はコード中のどこででも使えます - 関数内で、グローバルスコープで、型定義の中で。

### `mixin template`

**定型文**を見るところどこでも、`mixin template`はあなたの友達です:

    mixin template Foo(T) {
        T foo;
    }
    ...
    mixin Foo!int; // int fooをここで利用できます。

`mixin template`は初期化点に挿入される任意の数の
複雑な式を含むことがあります。あなたがCから来たなら、
プリプロセッサとはサヨナラしましょう!

### テンプレート制約

テンプレートは型が持つ必要があるプロパティを強制する
任意の数の制約とともに定義されることがあります:

    void foo(T)(T value)
      if (is(T : int)) { // foo!T はTがintに
                         // 変換される場合のみ有効
    }

制約はブール式と組み合わせることができ、コンパイル時に計算できる
関数呼び出しを含むことがあります。たとえば`std.range.primitives.isRandomAccessRange`
は型が`[]`オペレータをサポートするレンジかどうかチェックします。

### 掘り下げる

### ベーシック・リファレンス

- [Tutorial to D Templates](https://github.com/PhilippeSigaud/D-templates-tutorial)
- [Conditional compilation](http://ddili.org/ders/d.en/cond_comp.html)
- [std.traits](https://dlang.org/phobos/std_traits.html)
- [More templates  _Programming in D_](http://ddili.org/ders/d.en/templates_more.html)
- [Mixins in  _Programming in D_](http://ddili.org/ders/d.en/mixin.html)

### アドバンスト・リファレンス

- [Conditional compilation](https://dlang.org/spec/version.html)
- [Traits](https://dlang.org/spec/traits.html)
- [Mixin templates](https://dlang.org/spec/template-mixin.html)
- [D Templates spec](https://dlang.org/spec/template.html)

## {SourceCode}

```d
import std.traits : isFloatingPoint;
import std.uni : toUpper;
import std.string : format;
import std.stdio : writeln;

/*
数値、整数または浮動小数点数
でのみ動作するベクトルです。
*/
struct Vector3(T)
  if (is(T: real))
{
private:
    T x,y,z;

    /*
    getterとsetterのジェネレータです、
    定型文は大嫌いなので!
    
    var -> T getVAR() and void setVAR(T)
    */
    mixin template GetterSetter(string var) {
        // 関数名を構築するためにmixinを使用します
        mixin("T get%s() const { return %s; }"
          .format(var.toUpper, var));

        mixin("void set%s(T v) { %s = v; }"
          .format(var.toUpper, var));
    }

    /*
    getX、 setXなどを簡単に生成します。
    mixinテンプレートにより機能します。
    */
    mixin GetterSetter!"x";
    mixin GetterSetter!"y";
    mixin GetterSetter!"z";

public:
    /*
    ドット関数は浮動小数点数型でのみ利用可能です
    */
    static if (isFloatingPoint!T) {
        T dot(Vector3!T rhs) {
            return x * rhs.x + y * rhs.y +
                z * rhs.z;
        }
    }
}

void main()
{
    auto vec = Vector3!double(3,3,3);
    // テンプレート制約のためにこれは動きません!
    // Vector3!string は違法です;

    auto vec2 = Vector3!double(4,4,4);
    writeln("vec dot vec2 = ", vec.dot(vec2));

    auto vecInt = Vector3!int(1,2,3);
    // floatの場合のみ静的に
    // 有効化されるためdot関数はありません。
    // vecInt.dot(Vector3!int(0,0,0));

    // getterとsetterが生成されます!
    vecInt.setX(3);
    vecInt.setZ(1);
    writeln(vecInt.getX, ",",
      vecInt.getY, ",", vecInt.getZ);
}
```
