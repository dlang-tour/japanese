# ビット操作

mixinによりコンパイル時にコードを生成するDの能力の好例がビット操作です。

### シンプルなビット操作

Dはビット操作のために以下の演算子を提供します:

- `&` ビット単位論理積
- `|` ビット単位論理和
- `~` ビット反転
- `<<`  ビット符号あり左シフト
- `>>`  ビット符号あり右シフト (上位ビットの符号が保持されます)
- `>>>` ビット符号なし右シフト

### 実践例

ビット操作の一般的な例はビットの値を読むことです。Dは最も一般的なタスクのために
`core.bitop.bt`を提供しますが、ビット操作に慣れ親しむためには、
ビットのテストの冗長な実装をしてみましょう。:

```d
enum posA = 1;
enum maskA = (1 << posA);
bool getFieldA()
{
    return _data & maskA;
}
```

一般化は1より長いブロックのテストをすることです。そのためブロックの長さの特殊な
リードマスクが必要で、データブロックはマスクが適用される前にシフトされます:

```d
enum posA = 1;
enum lenA = 3;
enum maskA = (1 << lenA) - 1; // ...0111
uint getFieldA()
{
    return (_data >> posA) & maskA;
}
```

そのようなブロックの設定は等しくマスクの否定によって定義し、特定のブロック内のみに書込みできます:

```d
void setFieldA(bool b);
{
    return (_data & ~maskAWrite) | ((b << aPos) & maskAWrite);
}
```

## 助けになる`std.bitmanip`

カスタムビット操作コードを書くことは非常に面白く、Dはそのための完全なツールボックスを提供します。
しかし非常にエラーを引き起こしやすく保守がしづらいため大抵の場合そのようなビット操作はコピー&ペーストはしたくありません。
したがってDでは`std.bitmanip`が保守性のあり、読みやすいビット操作を書くことを`std.bitmanip`とmixinの力で
助けます - パフォーマンスは犠牲になりません。

エクササイズのセクションをご覧ください。`BitVector`は定義されていますが、まだXビットを使い、
普通の構造体とほとんど見分けがつきません。

`std.bitmanip`と`core.bitop`少ないメモリ消費を必要とする
アプリケーションのための非常に有用な更に多くのヘルパがあります。

### パディングとアライメント

コンパイラは、`bool`, `byte`, `char` のようなOSのメモリレイアウト (`size_t.sizeof`)
よりも小さいサイズの変数に対してパディングを挿入するため、アライメントの大きいフィールドから書くと良いでしょう。

## 掘り下げる

- [std.bitmanip](http://dlang.org/phobos/std_bitmanip.html) - Bit-level manipulation facilities
- [_Bit Packing like a Madman_](http://dconf.org/2016/talks/sechet.html)

## {SourceCode}

```d
struct BitVector
{
    import std.bitmanip : bitfields;
    // 下記のプロキシで
    // プライベートフィールドを作成します
    mixin(bitfields!(
        uint, "x",    2,
        int,  "y",    3,
        uint, "z",    2,
        bool, "flag", 1));
}

void main()
{
    import std.stdio : writefln, writeln;

    BitVector vec;
    vec.x = 2;
    vec.z = vec.x - 1;
    writefln("x: %d, y: %d, z: %d",
              vec.x, vec.y, vec.z);

    // たった8ビット - 1バイトが使われます
    writeln(BitVector.sizeof);

    struct Vector { int x, y, z; }
    // 変数あたり4バイト(int)
    writeln(Vector.sizeof);

	struct BadVector
	{
		bool a;
		int x, y, z;
		bool b;
	}
	// パディングのため、それぞれの
	// フィールドに4バイトが使われます
	writeln(BadVector.sizeof);
}
```
