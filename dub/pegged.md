# Pegged

Pegged は、解析式文法 (parsing expression grammar、PEG) ジェネレーターです。

ジェネレーターにPEG文法を与えるのが基本的な考え方です。
文法定義をすると関連するパーサーのセットが作成され、実行時またはコンパイル時に使用されます。

## 参照

- [Pegged on GitHub](https://github.com/PhilippeSigaud/Pegged)
- [Reference article for Pegged's syntax](http://bford.info/pub/lang/peg)

## {SourceCode:fullWidth}

```d
/+dub.sdl:
dependency "pegged" version="~>0.4.2"
+/
import pegged.grammar;
import std.stdio;

mixin(grammar(`
Arithmetic:
    Term     < Factor (Add / Sub)*
    Add      < "+" Factor
    Sub      < "-" Factor
    Factor   < Primary (Mul / Div)*
    Mul      < "*" Primary
    Div      < "/" Primary
    Primary  < Parens / Neg / Pos / Number / Variable
    Parens   < "(" Term ")"
    Neg      < "-" Primary
    Pos      < "+" Primary
    Number   < ~([0-9]+)

    Variable <- identifier
`));

void main()
{
    // コンパイル時にパースします
    enum parseTree1 = Arithmetic("1 + 2 - (3*x-5)*6");

    pragma(msg, parseTree1.matches);
    assert(parseTree1.matches == ["1", "+", "2", "-",
       "(", "3", "*", "x", "-", "5", ")", "*", "6"]);
    writeln(parseTree1);

    // 実行時も同様です
    auto parseTree2 = Arithmetic(" 0 + 123 - 456 ");
    assert(parseTree2.matches == ["0", "+", "123", "-", "456"]);
}
```
