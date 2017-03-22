# ユニットテスト

テストは、安定した、バグのないアプリケーションを確実にする素晴らしい方法です。
それはインタラクティブなドキュメントとしての役割をはたし、
機能を破壊してしまうことを恐れずにコードを修正することを可能にします。
DはD言語の機能の一部のブロックとして、便利でネイティブな
`unittest`の構文を提供します。ソースコードの機能をテストするために、
Dのモジュールのどこでも`unittest`ブロックを使えます。

    // マイ関数のためのブロック
    unittest
    {
        assert(myAbs(-1) == 1);
        assert(myAbs(1)  == 1);
    }

これは必要に応じて直接的に
[テスト駆動開発](https://en.wikipedia.org/wiki/Test-driven_development)
を可能にします。

### `unittest`ブロックの実行

`unittest`ブロックはコマンドラインフラグ`-unittest`がDMDコンパイラに渡された時に
コンパイルされ実行される任意のコードを含むことができます。DUBも`dub test`コマンドによって
ユニットテストをコンパイル、実行する機能があります。

### `assert`によるサンプルの検証

典型的な`unittest`は与えられた関数の機能をテストする`assert`式を含みます。
`unittest`ブロックは典型的にはソースのトップレベルか、
クラスや構造体にある関数の定義の近くに置かれます。

### コードカバレッジの増加

ユニットテストは堅牢なアプリケーションを作るための強力な武器です。
テストによってプログラムがどれくらいカバーされているかをチェックする
一般的な測定は、**コードカバレッジ**です。それは存在するコードの行に対する実行された行の比率です。
DMDコンパイラは`-cov`と付け加えることで簡単にコードカバレッジレポートを生成することを可能にします。
すべてのモジュールの詳細な統計が含まれる`.lst`ファイルが生成されます。

コンパイラはテンプレート化されたコードの属性を自動的に推論することができるため、
テストされたコードのためにそのような属性を確保するために注釈付けされたユニットテストを追加することは
一般的なパターンです:

    unittest @safe @nogc nothrow pure
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
          Vector3(0,1,0) == 0);
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
または他の場所も。
ノーマルモードでは何もコンパイルされず、単に無視されます。
実際にあなたのモジュールをテストするためにはローカルに
dub test を実行するか
dmd -unittestでコンパイルしてください。
*/
unittest {
    Vector3 vec;
    // 型の初期値を返す特殊な組み込みプロパティ
    // .init です。
    assert(vec.x == double.init);
}
```
