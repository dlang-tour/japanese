# 例外

このガイドではユーザ`Exception`についてのみ取り扱います。
システム`Error`は通常致命的で、キャッチすべき**ではありません**。

### 例外をキャッチする

例外の一般的な例は、無効の可能性があるユーザの入力を確認することです。
例外が投げられると、スタックは最初にマッチする例外ハンドラが見つかるまで巻き戻されます。

```d
try
{
    readText("dummyFile");
}
catch (FileException e)
{
    // ...
}
```

複数の`catch`ブロックや、エラーが起きたかどうかに関係なく実行される`finally`ブロックを持つことができます。
例外は`throw`によって投げられます。

```d
try
{
    throw new StringException("You shall not pass.");
}
catch (FileException e)
{
    // ...
}
catch (StringException e)
{
    // ...
}
finally
{
    // ...
}
```

[スコープガード](gems/scope-guards)が通常`try-finally`パターンの良い手法であることを覚えておいてください。

### カスタム例外

`Exception`から簡単に継承ができ、カスタム例外を作ることができます:

```d
class UserNotFoundException : Exception
{
    this(string msg, string file = __FILE__, size_t line = __LINE__) {
        super(msg, file, line);
    }
}
throw new UserNotFoundException("D-Man is on vacation");
```

### `nothrow`で安全な世界へ

Dコンパイラは関数が壊滅的副作用を引き起こさないことを確実にできます。
そのような関数は`nothrow`キーワードで注釈することができます。
`nothrow`関数内でDコンパイラは例外を投げることを静的に禁止します。

```d
bool lessThan(int a, int b) nothrow
{
    writeln("unsafe world"); // 出力は例外を投げる可能性があり、従ってこれは禁止です。
    return a < b;
}
```

コンパイラはテンプレート化されたコードの属性を推論できることを覚えておいてください。

### std.exception

`assert`を使ってユーザーの入力にこの後紹介される
[契約プログラミング](gems/contract-programming)
を行うとリリースモードでコンパイルするときに削除されてしまうため、
`assert`の使用は避けなければなりません。
利便性のために[`std.exception`](https://dlang.org/phobos/std_exception.html)
は[`enforce`](https://dlang.org/phobos/std_exception.html#enforce)を提供しており、
これは`assert`のように使えますが、`AssertError`のかわりに`Exception`を投げます。

```d
import std.exception : enforce;
float magic = 1_000_000_000;
enforce(magic + 42 - magic == 42, "Floating-point math is fun");

// カスタム例外を投げる
enforce!StringException('a' != 'A', "Case-sensitive algorithm");
```

しかし`std.exception`にあるのははこれだけではありません。たとえばエラーが
致命的でないかもしれない時、それを`collect`することをオプトインできます:

```d
import std.exception : collectException;
auto e = collectException(aDangerousOperation());
if (e)
    writeln("The dangerous operation failed with ", e);
```

テスト内で例外が投げられるかどうかテストするためには、`assertThrown`を使います。

### 掘り下げる

- [Exception Safety in D](https://dlang.org/exception-safe.html)
- [std.exception](https://dlang.org/phobos/std_exception.html)
- system-level [core.exception](https://dlang.org/phobos/core_exception.html)
- [object.Exception](https://dlang.org/library/object/exception.html) - base class of all exceptions that are safe to catch and handle.

## {SourceCode}

```d
import std.file : FileException, readText;
import std.stdio : writeln;

void main()
{
    try
    {
        readText("dummyFile");
    }
    catch (FileException e)
    {
		writeln("Message:\n", e.msg);
		writeln("File: ", e.file);
		writeln("Line: ", e.line);
		writeln("Stack trace:\n", e.info);

		// デフォルトのフォーマットも使えます
		// writeln(e);
    }
}
```
