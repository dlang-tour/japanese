# コード生成(パーサ)

この例では設定パーサをコンパイル時に生成します。
プログラムが2つの設定項目を持ち、設定`struct`に集約されているとしましょう:

```d
struct Config
{
    int runs, port;
    string name;
}
```

この構造体に対するパーサーを書くのは難しいことではないですが、
`Config`オブジェクトが変更されるたびにパーサーを更新しなければならないでしょう。
なので、任意の設定項目を読み込めるジェネリックな`parse`関数を書いてみたいです。
簡単のために`parse`は`key1=value1,key2=value2`という非常に単純な書式の設定項目を受け付けることにしますが、
同様の手法が任意の設定フォーマットに利用できます。
もちろん多くの有名な設定フォーマットのためのリーダが[DUBレジストリ](https://code.dlang.org)には存在します。

設定を読む
-------------------------

"name=dlang,port=8080"がユーザの設定文字列だとしましょう。
そして設定項目をコンマで分割し、各設定に対して`parse`を呼び出します。
すべての設定項目がパースされた後、設定オブジェクト全体が印字されます。

パース
-----

`parse`で素敵な処理が実際に行われる場所ですが、まずは与えられた設定項目(例: "name=dlang")
を"="でキー("name")と値("dlang")に分割します。
switch文はパースしたキーに対して実行されますが、興味深いのはswitch caseが静的に生成されることです。
`c.tupleof`はすべてのメンバのリストを`(idx, name)`という形式で返します。
コンパイラは`c.tupleof`がコンパイル時にわかることを検出し、コンパイル時にforeachループを展開します。
したがって、`Conf.tupleof[idx].stringof`は構造体オブジェクトの各メンバを返し、
各メンバに対してcase文を生成します。
同様に、静的ループの中では各メンバに`c.tupleof[idx]`のようにインデックスでアクセスできます。
なので、個々のメンバに設定文字列からパースした値を代入できます。
さらに、分割されたレンジはキーを指しているため`dropOne`が必要で、
`droOne.front`は2番めの要素を返します。
さらに、`to!(typeof(field))`は入力文字列の設定構造体のメンバの個々の型へのパースを行います。
最後に、foreachループはコンパイル時に展開され`break`がループを停止します。
しかし、設定項目がパースされた後にはswitch文の次のケースにジャンプしたくないため、
ラベル付されたbreakはswitch文の外に脱出するために使われます。


## {SourceCode}

```d
import std.algorithm, std.conv, std.range,
        std.stdio, std.traits;
struct Config
{
    int runs, port;
    string name;
}
void main()
{
    Config conf;
    // 生成されたパーサを各エントリに使用
    "runs=1,port=2,name=hello"
        .splitter(",")
        .each!(e => conf.parse(e));
    conf.writeln;
}
void parse(Conf)(ref Conf c, string entry)
{
    auto r = entry.splitter("=");
    auto key = r.front, value = r.dropOne.front;
    Switch: switch (key)
    {
        static foreach(idx, field; Conf.tupleof)
        {
            case field.stringof:
                c.tupleof[idx] =
                    value.to!(typeof(field));
                break Switch;
        }
        default:
        assert (0, "Unknown member name.");
    }
}
```
