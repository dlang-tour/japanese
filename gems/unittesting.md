# ユニットテスト

テストは、安定した、バグのないアプリケーションを確実にする素晴らしい方法です。
それはインタラクティブなドキュメントとしての役割をはたし、
機能を破壊してしまうことを恐れずにコードを修正することを可能にします。
Dは言語機能の一部として、便利でネイティブな
`unittest`ブロック構文を提供します。ソースコードの機能をテストするために、
Dのモジュールのどこでも`unittest`ブロックを使えます。

    // 自作関数のためのブロック
    unittest
    {
        assert(myAbs(-1) == 1);
        assert(myAbs(1)  == 1);
    }

これは直接的に
[テスト駆動開発](https://en.wikipedia.org/wiki/Test-driven_development)
の需要に応えます。

### `unittest`ブロックの実行

`unittest`ブロックはコマンドラインフラグ`-unittest`がDMDコンパイラに渡された時に
コンパイルされ実行される任意のコードを含むことができます。DUBにも`dub test`コマンドによって
ユニットテストをコンパイル、実行する機能があります。

### `assert`によるサンプルの検証

典型的な`unittest`は与えられた関数の機能をテストする`assert`式を含みます。
`unittest`ブロックは典型的にはソースのトップレベルか、
クラスや構造体にある関数の定義の近くに置かれます。

### コードカバレッジの増加

ユニットテストは堅牢なアプリケーションを作るための強力な武器です。
テストによってプログラムがどれくらいカバーされているかをチェックする
一般的な測定法は、**コードカバレッジ**です。それは存在するコードの行に対する実行された行の比率です。
DMDコンパイラでは`-cov`と付け加えることで簡単にコードカバレッジレポートの生成ができます。
すべてのモジュールの詳細な統計が含まれる`.lst`ファイルが生成されます。

コンパイラはテンプレート化されたコードの属性を自動的に推論することができるため、
テストされたコードのためにそのような属性を確保するために注釈付けされたユニットテストを追加することは
一般的なパターンです:

    @safe @nogc nothrow pure unittest
    {
        assert(myAbs() == 1);
    }

### 掘り下げる

- [Unit Testing in _Programming in D_](http://ddili.org/ders/d.en/unit_testing.html)
- [Unittesting in D](https://dlang.org/spec/unittest.html)

## {SourceCode}

```d
import std.stdio : writeln;

struct Vector3 {
    double x;
    double y;
    double z;

    double dot(Vector3 rhs) const {
        return x*rhs.x + y*rhs.y + z*rhs.z;
    }

    // これは正しいです!
    unittest {
        assert(Vector3(1,0,0).dot(
          Vector3(0,1,0)) == 0);
    }

    string toString() const {
        import std.string : format;
        return format("x:%.1f y:%.1f z:%.1f",
          x, y, z);
    }

    // ……そしてこれも!
    unittest {
        assert(Vector3(1,0,0).toString() ==
          "x:1.0 y:0.0 z:0.0");
    }
}

void main()
{
    Vector3 vec = Vector3(0,1,0);
    writeln(`This vector has been tested: `,
      vec);
}

/*
また他の場所も。
ノーマルモードでは何もコンパイルされず、単に無視されます。
実際にあなたのモジュールをテストするためにはローカルで
dub test を実行するか
dmd -unittestでコンパイルしてください。
*/
unittest {
    import std.math : isNaN;
    Vector3 vec;
    // 型の初期値、浮動小数点数の場合は
    // NaNを返す特殊な組み込みプロパティ
    // .init です。
    assert(vec.x.init.isNaN);
}
```
