# 関数

ひとつの関数はすでに導入されています: `main()` - すべてのDプログラムの開始地点です。
関数は任意の個数の引数を受け取り、値を返します - または、`void`と宣言されると何も返しません:

    int add(int lhs, int rhs) {
        return lhs + rhs;
    }

### `auto` 返値型

返値型が`auto`と定義されている場合、D コンパイラは自動的に返値型を推論します。
したがって複数の`return`式は互換性のある型の値を返す必要があります。

    auto add(int lhs, int rhs) { // `int`を返す
        return lhs + rhs;
    }

    auto lessOrEqual(int lhs, int rhs) { // `double`を返す
        if (lhs <= rhs)
            return 0;
        else
            return 1.0;
    }

### デフォルト引数

関数は必要に応じてデフォルト引数を定義することができます。
これは冗長なオーバーロードの定義という退屈な仕事を回避します。

    void plot(string msg, string color = "red") {
        ...
    }
    plot("D rocks");
    plot("D rocks", "blue");

デフォルト引数が指定された後はその次からのすべての引数もデフォルト引数でなければなりません。

### ローカル関数

関数はそれが局所的に使え、外の世界に見えないようにできる、他の関数の中で宣言されることもあります。
これらの関数は親のスコープのローカルオブジェクトにアクセスすることもできます:

    void fun() {
        int local = 10;
        int fun_secret() {
            local++; // 正しい
        }
        ...

このようなネストした関数はデリゲートと呼ばれ、
[すぐに](basics/delegates)より深く説明されます。

### 掘り下げる

- [Functions in _Programming in D_](http://ddili.org/ders/d.en/functions.html)
- [Function parameters in _Programming in D_](http://ddili.org/ders/d.en/function_parameters.html)
- [Function specification](https://dlang.org/spec/function.html)

## {SourceCode}

```d
import std.stdio;
import std.random;

void randomCalculator()
{
    // 4つの異なる数学的操作のため
    // 4つのローカル関数を定義します。
    auto add(int lhs, int rhs) {
        return lhs + rhs;
    }
    auto sub(int lhs, int rhs) {
        return lhs - rhs;
    }
    auto mul(int lhs, int rhs) {
        return lhs * rhs;
    }
    auto div(int lhs, int rhs) {
        return lhs / rhs;
    }

    int a = 10;
    int b = 5;

    // uniformはSTARTからENDまでの数字を生成します。
    // ただしENDは含まれません。
    // その結果に応じて数学的操作のいずれかを呼び出します。
    switch (uniform(0, 4)) {
        case 0:
            writeln(add(a, b));
            break;
        case 1:
            writeln(sub(a, b));
            break;
        case 2:
            writeln(mul(a, b));
            break;
        case 3:
            writeln(div(a, b));
            break;
        default:
            // 到達不能コードを表す特殊なコード
            assert(0);
    }
}

void main()
{
    randomCalculator();
    // add()、 sub()、 mul() そして div()は
    // そのスコープの外からは見えません。
    static assert(!__traits(compiles,
                            add(1, 2)));
}

```
