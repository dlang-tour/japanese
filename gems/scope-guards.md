# スコープガード

スコープガードを使うことで、現在のブロックから脱出したときに一定の条件で文を実行できます:

* `scope(exit)`は常に文を呼び出します
* `scope(success)`文は例外が何も投げられなかった時に呼ばれます
* `scope(failure)`はブロックの終わりの前に例外が投げられた時に呼ばれる文を表します

スコープガードを使ことでコードはよりクリーンになり、
リソース割り当てとクリーンアップコードを並べて配置することができます。
これらのちょっとしたヘルパは実行時にどのようなパスを通ったかに関係なく
クリーンアップコードが**常に**呼ばれることを確実にするため、安全性を改善することができます。

Dの`scope`機能はC++で使われる、しばしば特殊なリソースのために特殊な
スコープガードオブジェクトを導入するRAIIイディオムを効率的に置き換えます。

スコープガードはそれらが定義されたのと逆順に呼ばれます。

### 掘り下げる

- [`scope` in _Programming in D_](http://ddili.org/ders/d.en/scope.html)

## {SourceCode}

```d
import std.stdio : writefln, writeln;

void main()
{
    writeln("<html>");
    scope(exit) writeln("</html>");

    {
        writeln("\t<head>");
        scope(exit) writeln("\t</head>");
        "\t<title>%s</title>".writefln("Hello");
    } // 前の行のscope(exit)はここで実行されます

    writeln("\t<body>");
    scope(exit) writeln("\t</body>");

    writeln("\t\t<h1>Hello World!</h1>");

    // スコープガードにより割り当てと
    // そのクリーンアップコードを並べて配置できます
    import core.stdc.stdlib : free, malloc;
    int* p = cast(int*) malloc(int.sizeof);
    scope(exit) free(p);
}
```
