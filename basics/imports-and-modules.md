# インポートとモジュール

Dでシンプルなhello worldプログラムを書くには`import`が必要です。
`import`ステートメントは与えられた利用可能な**モジュール**から
すべてのパブリックな関数と型を作ります。

[Phobos](https://dlang.org/phobos/)と呼ばれる標準ライブラリが
`std`**パッケージ**下にあり、
それらのモジュールは`import std.モジュール名`で参照できます。

`import`ステートメントはモジュールの特定のシンボルを選択的に
インポートすることもできます。これはDのソースコードのすでに短い
コンパイル時間をさらに改善します。

    import std.stdio: writeln, writefln;

`import`ステートメントはソースファイルの一番上に出てくる必要はありません。
関数やその他のスコープ内で局所的に使用することもできます。

## {SourceCode}

```d
void main()
{
    import std.stdio;
    // または import std.stdio: writeln;
    writeln("Hello World!");
}
```
