# Nimのガベージコレクター

原著：Andreas Rumpf
原文：[https://nim-lang.org/docs/gc.html](https://nim-lang.org/docs/gc.html)
Version：1.02

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

### 時間測定(Time measurement)

### GCの調整(Tweaking the GC)

### メモリの追跡(Keeping track of memory)

## ヒープダンプ(Heap dump)

### ガベージコレクターオプション(Garbage collector options)
