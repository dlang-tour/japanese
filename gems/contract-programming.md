# 契約プログラミング

Dの契約プログラミングはコードベースが期待通りに振る舞うことを確実にする
サニティ・チェックを反映させることによりコードの質を向上させることができる
言語構成体群を含みます。契約は**デバッグ**モードでのみ利用でき、
リリースモードでは実行されません。したがってユーザの入力を
検証するのに使われてはいけません。

### `assert`

Dの最も単純な契約プログラミングの形は`assert(...)`式で、
これは一定の条件を満たすかをチェックし、
そうでない場合は`AssertionError`でプログラムを中断します。

    assert(sqrt(4) == 2);
    // 任意のカスタムアサーションエラーメッセージ
    assert(sqrt(16) == 4, "sqrt is broken!");

### 関数の契約

`in`と`out`で入力パラメータと関数の返値の契約を形式化することができます。

    long square_root(long x)
    in {
        assert(x >= 0);
    } out (result) {
        assert((result * result) <= x
            && (result+1) * (result+1) > x);
    } do {
        return cast(long)std.math.sqrt(cast(real)x);
    }

`in`ブロックの中のコンテンツは関数の本文中に表現することもできますが、
この方法のほうが意図が明確です。`out`ブロックの中で関数の返値は
`out(result)`でキャプチャしそこで検証されます。

### 不変条件のチェック

`invariant()`は`struct`や`class`型の特殊なメンバ関数で、
オブジェクトのライフタイム全体におけるその状態のサニティ・チェックを行うものです:

* それはコンストラクタが実行された後とデストラクタが呼ばれる前に呼ばれます
* それはメンバ関数に入る前に呼ばれます
* `invariant()`はメンバ関数を抜けたあとに呼ばれます。

### ユーザ入力を検証する

すべての契約はリリースビルドで削除されるため、ユーザ入力は契約を使って検証されるべきではありません。
加えて(訳注:例外ではなく)致命的な`Errors`を投げるため、`assert`は`nothrow`関数で使うことができます。
実行時の`assert`の類似物は[`std.exception.enforce`](https://dlang.org/phobos/std_exception.html#.enforce)で、
これはキャッチできる`Exceptions`を投げます。

### 掘り下げる

- [`assert` and `enforce` in _Programming in D_](http://ddili.org/ders/d.en/assert.html)
- [Contract programming in _Programming in D_](http://ddili.org/ders/d.en/contracts.html)
- [Contract Programming for Structs and Classes in _Programming in D_](http://ddili.org/ders/d.en/invariant.html)
- [Contract programming in D spec](https://dlang.org/spec/contracts.html)
- [`std.exception`](https://dlang.org/phobos/std_exception.html)

## {SourceCode:incomplete}

```d
import std.stdio: writeln;

/**
単純化されたDate型
std.datetimeの代わりに使います
*/
struct Date {
    private {
        int year;
        int month;
        int day;
    }

    this(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }

    invariant() {
        assert(year >= 1900);
        assert(month >= 1 && month <= 12);
        assert(day >= 1 && day <= 31);
    }

    /**
    DateオブジェクトをYYYY-MM-DDという文字列からシリアライズします。
    
    Params:
        date = シリアライズされる文字列
        
    Returns: Dateオブジェクト。
    /*
    void fromString(string date)
    in {
        assert(date.length == 10);
    }
    do {
        import std.format: formattedRead;
        // formattedReadは与えられたフォーマットで
        // パースし与えられた変数に結果を書き込みます
        formattedRead(date, "%d-%d-%d",
            &this.year,
            &this.month,
            &this.day);
    }

    /**
    YYYY-MM-DDの形へDateオブジェクトをシリアライズします

    Returns: Dateの文字列表現
    /*
    string toString() const
    out (result) {
        import std.algorithm: all, count,
                              equal, map;
        import std.string: isNumeric;
        import std.array: split;

        // YYYY-MM-DDの形を返すことを検証
        assert(result.count("-") == 2);
        auto parts = result.split("-");
        assert(parts.map!`a.length`
                    .equal([4, 2, 2]));
        assert(parts.all!isNumeric);
    }
    do {
        import std.format : format;
        return format("%.4d-%.2d-%.2d",
                      year, month, day);
    }
}

void main() {
    auto date = Date(2016, 2, 7);

    // これはinvariant failを引き起こします。
    // 契約でユーザ入力を検証しないで、
    // かわりに例外を投げてください。
    date.fromString("2016-13-7");

    date.writeln;
}
```
