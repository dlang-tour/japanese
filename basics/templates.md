# テンプレート

**D**はC++やJavaと同じ、任意の型で動作するよう関数本文内の式をコンパイルする
**ジェネリックな**関数またはオブジェクトをを定義することを意味する、テンプレート化された関数を定義することができます:

    auto add(T)(T lhs, T rhs) {
        return lhs + rhs;
    }

テンプレート引数`T`は実際の関数の引数の前の括弧内で定義されます。
`T`は`!`演算子を使い関数が実際に**インスタンス化**される時にコンパイラによって置き換えられるプレースホルダです:

    add!int(5, 10);
    add!float(5.0f, 10.0f);
    add!Animal(dog, cat); // コンパイルされません; Animalに+は実装されていません

### 暗黙的テンプレート引数

関数テンプレートは2つのセットされる引数があります - 最初の1つはコンパイル時の引数、2つ目は実行時の引数です
(テンプレート化されていない関数は実行時引数のみで良いです)。
1つ以上のコンパイル時引数がその関数が呼ばれた時指定されていなければ、
コンパイラは実行時引数の型のリストから予測を試みます。

    int a = 5; int b = 10;
    add(a, b); // Tは`int`と予測される
    float c = 5.0;
    add(a, c); // Tは`float`と予測される

### テンプレートプロパティ

関数はインスタンス化のとき`func!(T1, T2 ..)`文を使って示される任意の個数の
テンプレート引数を持つことができますテンプレートパラメータは`string`と
浮動小数点数を含む任意の基本型にできます。

Javaのジェネリクスと違い、Dのテンプレートはコンパイル時専用で、実際に関数が呼び出されるときの
特定の型のセットに合わせたなかなか最適化されたコードを生産します

もちろん、`struct`、`class`、そして`interface`型もテンプレート型として定義できます。

    struct S(T) {
        // ...
    }

### 掘り下げる

- [Tutorial to D Templates](https://github.com/PhilippeSigaud/D-templates-tutorial)
- [Templates in _Programming in D_](http://ddili.org/ders/d.en/templates.html)

#### アドバンスト

- [D Templates spec](https://dlang.org/spec/template.html)
- [Templates Revisited](http://dlang.org/templates-revisited.html):  Walter Bright writes about how D improves upon C++ templates.
- [Variadic templates](http://dlang.org/variadic-function-templates.html): Articles about the D idiom of implementing variadic functions with variadic templates

## {SourceCode}

```d
import std.stdio : writeln;

/**
動物のジェネリックな実装のテンプレートクラスです。
Params:
    noise = 出力する文字列
*/
class Animal(string noise) {
    void makeNoise() {
        writeln(noise ~ "!");
    }
}

class Dog: Animal!("Bark") {
}

class Cat: Animal!("Meeoauw") {
}

/**
関数makeNoiseを実装した任意の型Tをとる
テンプレート関数です。
Params:
    animal = 鳴き声を発することができるオブジェクト
    n = makeNoiseの呼び出し回数
*/
void multipleNoise(T)(T animal, int n) {
    for (int i = 0; i < n; ++i) {
        animal.makeNoise();
    }
}

void main() {
    auto dog = new Dog;
    auto cat = new Cat;
    multipleNoise(dog, 5);
    multipleNoise(cat, 5);
}
```
