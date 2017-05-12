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

    class Dog: Animal {
        override makeNoise() {
            ...
        }
    }

    auto dog = new Dog;
    Animal animal = dog; // インターフェースへの明示的なキャスト
    dog.makeNoise();

`class`が実装できる`interface`の数は無制限ですが、継承は**1つの**基底クラスからのみできます。

### NVI (non virtual interface) pattern

The [NVI pattern](https://en.wikipedia.org/wiki/Non-virtual_interface_pattern)
prevents the violation of a common execution pattern by allowing _non virtual_ methods
for a common interface.
D easily enables the NVI pattern by
allowing the definition of `final` functions in an `interface`
that aren't allowed to be overridden. This enforces specific
behaviours customized by overriding the other `interface`
functions.

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