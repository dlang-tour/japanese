# コンパイル時関数実行 (CTFE)

CTFEはコンパイラに関数を**コンパイル時**に実行することを可能にさせる機構です。
この機能を使うのに必要なD言語の特殊なセットはありません - 関数がコンパイル時に
わかる値のみに依存するときはいつでもDコンパイラは
コンパイル中にそれを解釈する可能性があります。

    // 結果はコンパイル時に計算されます。
    // マシンコードをチェックしてください、
    // 関数呼び出しは含まれないことでしょう!
    static val = sqrt(50);

`static`、`immutable`、または`enum`のようなキーワードは出来る限り
CTFEを使うようコンパイラに命令します。このテクニックの素晴らしいところは、
関数はそれを使うために書き換えることを必要とせず、
同じコードは完璧に共有されるところです:

    int n = doSomeRuntimeStuff();
    // 上と同じ関数ですがこの時は
    // 古典的に実行時に呼ばれます。
    auto val = sqrt(n);

Dにおける顕著な例は[std.regex](https://dlang.org/phobos/std_regex.html)
ライブラリです。それは**文字列Mixin**とCTFEをとても最適化された正規表現オートマトンを
コンパイル時に生成するために使う`ctRegex`型を提供します。同じコードベースは実行時のみ
利用できる正規表現をコンパイルすることを可能にする実行時バージョンの`regex`に再利用されます。

    auto ctr = ctRegex!(`^.*/([^/]+)/?$`);
    auto tr = regex(`^.*/([^/]+)/?$`);
    // ctrとtrは互換的に使うことができますが
    // ctrのほうが速いでしょう!

CTFEではすべての言語機能は使えませんが、サポートされる機能セットは
コンパイラのリリースとともに増えています。

### 掘り下げる

- [Introduction to regular expressions in D](https://dlang.org/regular-expression.html)
- [std.regex](https://dlang.org/phobos/std_regex.html)
- [Conditional compilation](https://dlang.org/spec/version.html)

## {SourceCode}

```d
import std.stdio : writeln;

/**
ニュートン近似法を使い数値の平方根を計算します。

Params:
    x = 平方根にされる数値
    
Returns: xの平方根
*/
auto sqrt(T)(T x) {
    // これ以上の反復の価値がないと判断して
    // 近似を停止するときのイプシロンです。
    enum GoodEnough = 0.01;
    import std.math : abs;
    // 良い開始値を選びます
    T z = x*x, old = 0;
    int iter;
    while (abs(z - old) > GoodEnough) {
        old = z;
        z -= ((z*z)-x) / (2*z);
    }

    return z;
}

void main() {
    double n = 4.0;
    writeln("The sqrt of runtime 4 = ",
        sqrt(n));
    static cn = sqrt(4.0);
    writeln("The sqrt of compile time 4 = ",
        cn);
}
```
