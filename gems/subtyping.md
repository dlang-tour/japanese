# サブタイピング

`struct`は他の`struct`から継承することはできません。
しかしDは`struct`のために、その機能を拡張する別の
素晴らしい方法を提供します: **サブタイピング**です。

構造体型は`alias this`としてそのメンバの1つを定義できます:

    struct SafeInt {
        private int theInt;
        alias theInt this;
    }

それ自身型にハンドルされない`SafeInt`の任意の関数や操作は
`alias this`したメンバに転送されます。
外部から`SafeInt`は普通の整数のように見えます。

これは他の型を新しい機能で拡張することを可能にし、
それでいて実行時のメモリの面でゼロ・オーバーヘッドです。
コンパイラは`alias this`メンバにアクセスした時にするべきことをします。

`alias this`はクラスにも動作します。

## {SourceCode}

```d
import std.stdio : writeln;

struct Vector3 {
    private double[3] vec;
    alias vec this;

    double dot(Vector3 rhs) {
        return vec[0]*rhs.vec[0] +
          vec[1]*rhs.vec[1] + vec[2]*rhs.vec[2];
    }
}

void main()
{
    Vector3 vec;
    // ここでは基本的にdouble[]として対話をします。
    vec = [ 0.0, 1.0, 0.0 ];
    assert(vec.length == 3);
    assert(vec[$ - 1] == 0.0);

    auto vec2 = Vector3([1.0,0.0,0.0]);
    // しかしこの機能は拡張されたものです!
    writeln("vec dot vec2 = ", vec.dot(vec2));
}
```
