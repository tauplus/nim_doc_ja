# Nimのガベージコレクター

原著：Andreas Rumpf  
原文：[https://nim-lang.org/docs/gc.html](https://nim-lang.org/docs/gc.html)  
Version：1.0.2

> 「地獄への道は善意で舗装されています。」

## イントロダクション(Introduction)
このドキュメントでは、GCの仕組みと（ソフト）リアルタイムシステム用にGCを調整する方法について説明します。

基本的なアルゴリズムは、サイクル検出による遅延参照カウントです。
スタック上の参照は、パフォーマンスの向上（およびCコード生成の容易化）のためにカウントされません。
現在、サイクル検出は単純なマーク&スイープGCによって実行されます。これは全て（スレッドローカルヒープ）をスキャンする必要があります。
`--gc：v2`は、これをインクリメンタルマーク&スイープに置き換えます。ただし、まだ正式版の準備ができていません。

GCは、メモリ割り当て操作でのみトリガーされます。タイマーによってはトリガーされず、バックグラウンドスレッドでは実行されません。

完全なコレクションを強制するには`GC_fullCollect`を呼び出します。一般に、GCに処理を行わせ、完全なコレクションを強制しない方が良いことに注意してください。

## サイクルコレクター(Cycle collector)
サイクルコレクターは、`GC_enableMarkAndSweep`および`GC_disableMarkAndSweep`を使用して、GCの他の部分から独立して有効化/無効化できます。

## リアルタイムサポート(Realtime support)
リアルタイムサポートを有効にするには、シンボルuseRealtimeGCを`--define:useRealtimeGC`で定義する必要があります（これをconfigファイルに追加することもできます）。
このスイッチを使用すると、GCは次の操作をサポートします。

```nim
proc GC_setMaxPause*(maxPauseInUs: int)
proc GC_step*(us: int, strongAdvice = false, stackSize = -1)
```

パラメーター`maxPauseInUs`および`us`の単位はマイクロ秒です。

これらの2つのプロシージャは、リアルタイムGCの2つの運用法です。

(1) GC_SetMaxPause Mode
> プログラムの起動時に`GC_SetMaxPause`を呼び出すと、トリガーされた各GCの実行が`maxPause`時間より長くかからないようにします。
ただし、`new`を呼び出すたびにGCがトリガーされ、`maxPause`時間がかかるため 、作業が均等に分散されない可能性があります（そして一般的にそうなります）。

(2) GC_step Mode
> これにより、GCに最大`us`マイクロ秒の間、処理を実行させることができます。
これは、メインループでGCを呼び出して、機能することを確認するのに役立ちます。
すべてのGCアクティビティを`GC_step`呼び出しにバインドするには、プログラムの起動時に`GC_disable`でGCを非アクティブ化します。
`strongAdvice`が`true`に設定されている場合、GCは収集サイクルの実行を強制されます。
そうしないと、収集するガベージがあまりない場合、GCは何もしないことを決定する場合があります。
`stackSize`パラメーターを使用して、現在のスタックサイズを指定することもできます。
スタック上の特定のポイントより下にユニークなNim参照がないことがわかっている場合、パフォーマンスを改善できます。
指定するサイズが、潜在的に最悪なケースのサイズよりも大きいことを確認してください。

これらのプロシージャは、「ベストエフォート」のリアルタイム保証を提供します。
特に、サイクルコレクターはまだ期限を認識していません。
これを無効にすると、より予測可能なリアルタイムの動作が得られます。
テストでは、最新のCPU（サイクルコレクターを無効にした場合）のほとんどすべてのケースで2msの最大休止時間が満たされることが示されています。

### 時間測定(Time measurement)
GCの時間測定方法（実装については`lib/system/timers.nim`を参照）：

- Windowsの`QueryPerformanceCounter`および`QueryPerformanceFrequency`
- Mac OS Xの`mach_absolute_time`
- Posixシステムの`gettimeofday`

そのため、内部でナノ秒の解像度をサポートしています。
ただし、便宜上、APIはマイクロ秒を使用します。

シンボル`reportMissedDeadlines`を定義して、期限に間に合わなかった場合にGC出力を作成します。
コレクターの以降のバージョンでは、APIによってレポート機能が強化されサポートされます。

### GCの調整(Tweaking the GC)
コレクターは、`workPackage`番目の反復ごとに作業にまだ時間が残っているかどうかをチェックします。
これは現在100に設定されています。つまり、再確認する前に最大100個のオブジェクトがトラバースされて解放されます。
したがって、`workPackage`はタイミングの粒度に影響し、高度に特殊化された環境や古いハードウェアでは調整する必要がある場合があります。

### メモリの追跡(Keeping track of memory)
Nimによって割り当てられたメモリをCに渡す必要がある場合は、プロシージャ`GC_ref`および`GC_unref`を使用してオブジェクトを参照済みとしてマークし、GCによって解放されないようにすることができます。
メモリを追跡するために使用できる[system](https://nim-lang.org/docs/system.html)の他の便利なプロシージャは次のとおりです。

- `getTotalMem()` ：GCによって管理されている合計メモリの量を返します。
- `getOccupiedMem()` ：GCによって予約され、オブジェクトによって使用されるバイト数。
- `getFreeMem()` ：GCによって予約され、使用されていないバイト。

これらの数値は通常ヒープ全体ではなく、実行中のスレッド専用です。
ただし、`--gc:boehm`および`--gc:go`は例外です。

`GC_ref`および`GC_unref`に加えて、`alloc`,`allocShared`,または`allocCStringArray`などのプロシージャを使用してメモリを手動で割り当てることにより、GCを回避できます。
GCはそれらを解放しようとしません。それらの処理が完了したら、それぞれのdeallocペアを呼び出す必要があります。そうしないと、リークが発生します。

## ヒープダンプ(Heap dump)
ヒープダンプ機能はまだ初期段階にありますが、既に有用であることが証明されているので、役に立つかもしれません。
ヒープダンプを取得するには、`-d:nimTypeNames`を指定してコンパイルし、プログラムの戦略的な場所で`dumpNumberOfInstances`を呼び出します。
これにより、プログラムで使用される型のリストが作成され、すべての型について、その型のオブジェクトインスタンスの合計量と、これらのインスタンスが占有するバイトの合計量が生成されます。
このリストは現在ソートされていません！ソートするには、外部シェルスクリプトハッキングを使用する必要があります。

数値は、すべてのGCヒープ内のオブジェクトの数をカウントし、現在のスレッドだけでなく、実行中のすべてのスレッドを参照します。
（現在のスレッドは`dumpNumberOfInstances`を呼び出すスレッドになります。）これは後のバージョンで変更される可能性があります。

### ガベージコレクターオプション(Garbage collector options)
ソースコードのコンパイル時に使用するガベージコレクタを選択できます。
選択したガベージコレクタのコンパイルコマンドで`--gc:`を渡すことができます。

- `--gc:refc` ：サイクル検出による遅延[参照カウント](https://en.wikipedia.org/wiki/Reference_counting)、 [スレッドローカルヒープ](https://en.wikipedia.org/wiki/Heap_(programming))、 デフォルト.
- `--gc:markAndSweep` ：マーク&スイープベースのガベージコレクター、 [スレッドローカルヒープ](https://en.wikipedia.org/wiki/Heap_(programming)).
- `--gc:boehm` ：[Boehm](https://en.wikipedia.org/wiki/Boehm_garbage_collector)ベースのガベージコレクター, [stop-the-world](https://en.wikipedia.org/wiki/Tracing_garbage_collection#Stop-the-world_vs._incremental_vs._concurrent)、 [shared heap](https://en.wikipedia.org/wiki/Heap_(programming)).
- `--gc:go` ：Golangのようなガベージコレクター、 [stop-the-world](https://en.wikipedia.org/wiki/Tracing_garbage_collection#Stop-the-world_vs._incremental_vs._concurrent)、 [shared heap](https://en.wikipedia.org/wiki/Heap_(programming))
- `--gc:regions` ：[スタック](https://en.wikipedia.org/wiki/Memory_management#Stack_allocation)ベースのガベージコレクタ.
- `--gc:none` ：ガベージコレクターなし.