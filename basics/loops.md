# ループ

Dは4つのループ構造を提供します。

### 1) `while`

`while`ループは与えられたコードを一定の条件が満たされている間実行します:

    while (condition)
    {
        foo();
    }

### 2) `do ... while`

`do .. while`ループは与えられたコードを一定の条件が満たされている間
実行しますが、`while`とは対象的に**ループブロック**はループ条件が
最初に評価される前に実行されます。

    do
    {
        foo();
    } while (condition);

### 3) 古典的`for`ループ

**初期化子**、**ループ条件**、そして**ループ文**からなる
C/C++またはJavaから知られる古典的な`for`ループです:

    for (int i = 0; i < arr.length; i++)
    {
        ...

### 4) `foreach`

次のセクションでより詳細に紹介される[`foreach` ループ](basics/foreach)です:

    foreach (el; arr)
    {
        ...
    }

#### 特殊なキーワードとラベル

特殊なキーワード`break`は現在のループを直ちに中止します。
ネストしたループ内で**ラベル**は任意の外側のループをやめさせるのに使われます:

    outer: for (int i = 0; i < 10; ++i)
    {
        for (int j = 0; j < 5; ++j)
        {
            ...
            break outer;

キーワード`continue`は次のループの反復処理を始めます。

### 掘り下げる

- `for` loop in [_Programming in D_](http://ddili.org/ders/d.en/for.html), [specification](https://dlang.org/spec/statement.html#ForStatement)
- `while` loop in [_Programming in D_](http://ddili.org/ders/d.en/while.html), [specification](https://dlang.org/spec/statement.html#WhileStatement)
- `do-while` loop in [_Programming in D_](http://ddili.org/ders/d.en/do_while.html), [specification](https://dlang.org/spec/statement.html#do-statement)

## {SourceCode}

```d
import std.stdio : writeln;

/*
配列の要素の平均値を計算します。
*/
double average(int[] array)
{
    immutable initialLength = array.length;
    double accumulator = 0.0;
    while (array.length)
    {
        // これは import std.array : front;
        // として .front とすることもできました
        accumulator += array[0];
        array = array[1 .. $];
    }

    return accumulator / initialLength;
}

void main()
{
    auto testers = [ [5, 15], // 20
          [2, 3, 2, 3], // 10
          [3, 6, 2, 9] ]; // 20

    for (auto i = 0; i < testers.length; ++i)
    {
      writeln("The average of ", testers[i],
        " = ", average(testers[i]));
    }
}
```
