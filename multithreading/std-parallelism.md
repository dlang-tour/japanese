# std.parallelism

モジュール`std.parallelism`は便利な並行プログラミングのための高いレベルのプリミティブを実装します。

### parallel

[`std.parallelism.parallel`](http://dlang.org/phobos/std_parallelism.html#.parallel)
は`foreach`本文を自動的に異なるスレッドに分配します:

    // 並列にarrを二乗
    auto arr = iota(1,100).array;
    foreach(ref i; parallel(arr)) {
        i = i*i;
    }

`parallel`は内部で`opApply`オペレータを使用します。
グローバルの`parallel`は**CPUの合計数**を使うワーキングスレッドである`TaskPool`である
`taskPool.parallel`へのショートカットです。
自分独自のインスタンスを作ると並列度をコントロールできます。

`parallel`な反復処理の本文は別のワーキングユニットがアクセスするかもしれない要素にアクセスしないことが
確実でなければならないことを覚えておいてください。

任意の`workingUnitSize`はワーカースレッドあたり処理される要素の数を決めます。

### reduce

関数
[`std.algorithm.iteration.reduce`](http://dlang.org/phobos/std_algorithm_iteration.html#reduce)は、
他の関数型の文脈では**accumulate**または**foldl**として知られるもので、
各要素`x`で関数`fun(acc, x)`を呼びます。ここで`acc`は前の結果です:

    // 0は"シード"です
    auto sum = reduce!"a + b"(0, elements);

[`Taskpool.reduce`](http://dlang.org/phobos/std_parallelism.html#.TaskPool.reduce)
は`reduce`と似たものです:

    // 各ワークユニットのシードとして最初の要素を使い、
    // レンジの合計を並列に求めます。
    auto sum = taskPool.reduce!"a + b"(nums);

`TaskPool.reduce`はレンジを並列にreduceされるサブレンジに分割します。
それらの結果が計算されると、その結果をreduceします。

### `task()`

[`task`](http://dlang.org/phobos/std_parallelism.html#.task)
は時間が長くかかったり、専用のワーキングスレッドで実行されるべき関数のラッパーです。
それはタスクプールのキューに追加されるか:

    auto t = task!read("foo.txt");
    taskPool.put(t);

または直接専用の新しいスレッドで実行されます:

    t.executeInNewThread();

タスクの結果を得るにはその`yieldForce`を呼びます。
それは結果が利用可能になるまでブロックします。

    auto fileData = t.yieldForce;

### 掘り下げる

- [Parallelism in _Programming in D_](http://ddili.org/ders/d.en/parallelism.html)
- [std.parallelism](http://dlang.org/phobos/std_parallelism.html)

## {SourceCode}

```d
import std.parallelism : task,
    taskPool, TaskPool;
import std.array : array;
import std.stdio : writeln;
import std.range : iota;

string theTask()
{
    import core.thread : dur, Thread;
    Thread.sleep( dur!("seconds")(1) );
    return "Hello World";
}

void main()
{
    // 2つのスレッドのタスクプールです
    auto myTaskPool = new TaskPool(2);
    // タスクプールを停止させることは重要です!
    scope(exit) myTaskPool.stop();

    // 長時間かかるタスクを実行し
    // その間に他の作業をします……
    auto task = task!theTask;
    myTaskPool.put(task);

    auto arr = iota(1, 10).array;
    foreach(ref i; myTaskPool.parallel(arr)) {
        i = i*i;
    }

    writeln(arr);

    import std.algorithm.iteration : map;

    // すべての二乗の和を並列に
    // 計算するためにreduceを使います。
    auto result = taskPool.reduce!"a+b"(
        0.0, iota(100).map!"a*a");
    writeln("Sum of squares: ", result);

    // 前にバックグラウンドに送った結果を取得します
    writeln(task.yieldForce);
}
```
