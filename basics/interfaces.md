# インターフェース

Dは`interface`という、技術的には`class`に似ていますが、
そのメンバ関数は`interface`から継承したクラスに
実装されなければならないものを定義できます。

    interface Animal {
        void makeNoise();
    }

`makeNoise`メンバ関数は、`Dog`が`Animal`インターフェースを継承しているために
`Dog`に実装されなければいけません。
本質的に`makeNoise`は基底クラスの`abstract`メンバ関数のように振る舞います。

    class Dog : Animal {
        override void makeNoise() {
            ...
        }
    }

    auto dog = new Dog;
    Animal animal = dog; // インターフェースへの明示的なキャスト
    animal.makeNoise();

`class`が実装できる`interface`の数は無制限ですが、継承は**1つの**基底クラスからのみできます。

### NVI (non virtual interface) パターン

[NVIパターン](https://en.wikipedia.org/wiki/Non-virtual_interface_pattern)
は共通したインターフェースで _非 virtual_ メソッドを許可します。
それによって、このパターンは共通した実行パターンに対する違反を防ぎます。
Dでは`interface`において`final`(つまりオーバーライド禁止)関数を定義できるようにすることでNVIパターンを実現します。
これにより`interface`における他の関数のオーバーライドによる、特殊化した振る舞いのカスタマイズが可能になります。

    interface Animal {
        void makeNoise();
        final doubleNoise() // NVI pattern
        {
            makeNoise();
            makeNoise();
        }
    }

### 掘り下げる

- [Interfaces in _Programming in D_](http://ddili.org/ders/d.en/interface.html)
- [Interfaces in D](https://dlang.org/spec/interface.html)

## {SourceCode}

```d
import std.stdio : writeln;

interface Animal {
    /*
    オーバーライドの必要な仮想関数です!
    */
    void makeNoise();

    /*
    NVIパターンです。makeNoiseを継承先クラスの
    振る舞いをカスタマイズするため内部で使います。
    
    Params: 
        n =  繰り返し回数
    */
    final void multipleNoise(int n) {
        for(int i = 0; i < n; ++i) {
            makeNoise();
        }
    }
}

class Dog: Animal {
    override void makeNoise() {
        writeln("Woof!");
    }
}

class Cat: Animal {
    override void makeNoise() {
        writeln("Meeoauw!");
    }
}

void main() {
    Animal dog = new Dog;
    Animal cat = new Cat;
    Animal[] animals = [dog, cat];
    foreach(animal; animals) {
        animal.multipleNoise(5);
    }
}
```
