# Nimマニュアル
原著：Andreas Rumpf, Zahary Karadjov
原文：[https://nim-lang.org/docs/manual.html](https://nim-lang.org/docs/manual.html)
Version：1.02

## このドキュメントについて
注：このドキュメントはドラフトです！Nimの機能のいくつかには、より正確な表現が必要な場合があります。このマニュアルは常に適切な仕様に進化しています。 

注：Nimの実験的な機能については、[こちら](https://nim-lang.org/docs/manual_experimental.html)で説明しています。  

注：割り当て、移動、および破棄は、[デストラクタ](https://nim-lang.org/docs/destructors.html)ドキュメントで指定されています。  

このドキュメントでは、Nim言語の語彙、構文、およびセマンティクスについて説明します。  

Nimプログラムをコンパイルしてドキュメントを生成する方法については、[コンパイラユーザーガイド](https://nim-lang.org/docs/nimc.html)および
[DocGenツールガイド](https://nim-lang.org/docs/docgen.html)を参照してください。

言語構成要素は、拡張BNFを使用して説明されます。`(a)*`は0個以上の`a`を意味し、`a+`は1個以上の`a`を意味し、`(a)?`はオプショナルな`a`を意味します。
括弧を使用して要素をグループ化できます。

`&`は先読み演算子です。`&a`は、 `a`が期待されるが消費されないことを意味します。次のルールで消費されます。

`|`,`/`記号は代替をマークするために使用され、優先順位が最も低くなります。
`/`は、パーサーが指定された順序で代替を試行することを必要とする順序付けされた選択です。
`/`は、文法が曖昧にならないようにするためによく使用されます。

非終端記号は小文字で始まり、抽象終端記号は大文字です。逐語的な終端記号（キーワードを含む）は`'`で囲みます。  
例：`ifStmt = 'if' expr ':' stmts ('elif' expr ':' stmts)* ('else' stmts)?`

2項演算子`^*`は、2番目の引数で区切られた0個以上のオカレンス（訳者注：適切な訳が分からず）の省略形として使用されます。
同様に、`^+`は1つ以上のオカレンスを意味します。`a ^+ b`は`a (b a)*`の短縮形であり、`a ^* b`は`(a (b a)*)`の短縮形です。  
例：`arrayConstructor = '[' expr ^* ',' ']'`

スコープルールやランタイムセマンティクスなど、Nimの他の部分は非公式に説明されています。

## 定義(Definitions)
Nimコードは、locationと呼ばれるコンポーネントで構成されるメモリに作用する計算を詳記します。
変数は、基本的にlocationの名前です。各変数とlocationは特定のタイプです。
変数の型は静的型と呼ばれ、locationの型は動的型と呼ばれます。
静的タイプが動的タイプと同じでない場合、それは動的タイプのスーパータイプまたはサブタイプです。

識別子は、変数、型、プロシージャなどの名前として宣言されたシンボルです。
宣言が適用されるプログラムの領域は、宣言のスコープと呼ばれます。
スコープはネストできます。
識別子の意味は（オーバーロード解決規則が別の方法を示唆しない限り）識別子が宣言される最小の囲みスコープによって決定されます。

式は値またはlocationを生成する計算を詳記します。
locationを生成する式は左辺値と呼ばれます。
左辺値は文脈に応じて、locationまたはlocationに含まれる値のいずれかを示すことができます。

Nimプログラムは、Nimコードを含む1つ以上のテキストソースファイルで構成され、Nimコンパイラーによって実行可能ファイルに処理されます。
この実行可能ファイルの性質は、コンパイラの実装に依存します。（例：ネイティブバイナリ、JavaScriptソースコード）

典型的なNimプログラムでは、ほとんどのコードが実行可能ファイルにコンパイルされます。
ただし、一部のコードはコンパイル時に実行される場合があります。
これには、定数式、マクロ定義、およびマクロ定義で使用されるNimプロシージャを含めることができます。
ほとんどのNim言語はコンパイル時にサポートされますが、いくつかの制限があります。
詳細は[コンパイル時実行の制限](#コンパイル時実行の制限)を参照。
ランタイムという用語は、コンパイル時実行と実行可能ファイルの実行の両方をカバーします。

コンパイラは、Nimソースコードを抽象構文木（AST）と呼ばれる内部データ構造に解析します。
次に、コード実行または実行可能ファイルにコンパイルする前に、セマンティック解析（意味解析）によりASTを変換します。
これにより、式のタイプ、識別子の意味、場合によっては式の値などのセマンティック情報が追加されます。
セマンティック分析中に検出されたエラーは、静的エラーと呼ばれます。
このマニュアルで説明されているエラーは、特に明記されていない限り静的エラーです。

チェックされたランタイムエラーは、実装により実行時に検出、報告されます。
このようなエラーを報告する方法は、例外を発生させるか、致命的なエラーで強制終了させることです。
ただし、実装はこれらのランタイムチェックを無効にする手段を提供します。詳細については、[プラグマ](#プラグマPragmas)セクションを参照してください。

チェックされたランタイムエラーの結果が例外になるか、致命的なエラーになるかは実装固有です。従って、次のプログラムは無効です。
コードが範囲外の配列アクセスから`IndexError`をキャッチすることを意図している場合でも、コンパイラは、プログラムが致命的なエラーで死ぬことを許可することを選択する場合があります。

```nim
var a: array[0..1, char]
let i = 5
try:
  a[i] = 'N'
except IndexError:
  echo "invalid index"
```

未確認のランタイムエラーは検出することを保証できず、その後の処理について想定外の結果となることがあります。
安全な言語機能のみが使用され、ランタイムチェックが無効になっていない場合、未チェックのランタイムエラーは発生しません。

定数式は、それが現れるコードのセマンティック解析中に値を計算できる式です。
左辺値でなく、副作用もありません。
定数式は、定数畳み込みなどのセマンティック解析の機能に制限されません。
コンパイル時の実行がサポートされている全てのNim言語機能を使用できます。
定数式は、セマンティック解析への入力として使用できるため（配列境界の定義など）、
この柔軟性のために、コンパイラはセマンティック解析とコンパイル時のコード実行をインターリーブする必要があります。

ソースコード内で上から下、左から右に進むセマンティック解析を想像し、その後のセマンティック分析に必要な値を計算するために必要なときにコンパイル時のコード実行をインターリーブすることは、ほとんど正確です。
このドキュメントの後半で、マクロの呼び出しではこのインターリーブが必要になるだけでなく、セマンティック解析が完全に上から下、左から右に進まない状況が生じることもわかります。

## 字句解析(Lexical Analysis)

### エンコーディング(Encoding)
すべてのNimソースファイルはUTF-8エンコーディング（またはそのASCIIサブセット）です。
他のエンコードはサポートされていません。
改行コードはLF(Unix)、CR+LF(Windows)、CR(古いMac)をプラットフォームに関係なく使用できます。

### インデント(Indentation)
Nimの標準文法においてインデントはとても重要です。
すべての制御構造がインデントによって認識されるからです。
インデントはスペースのみで構成され、タブは使用できません。

インデント処理は、次のように実装されます。
字句解析器は、後続のトークンにその前についているスペース数で注釈を付けます。
インデントは別個のトークンではありません。
この仕組みにより、先読みトークンを1つだけ使用してNimを解析できます。

パーサーは、インデントレベルのスタックを使用します。
スタックは、スペースをカウントする整数で構成されます。
インデント情報は、パーサーの戦略的な場所で照会されますが、そうでない場所では無視されます。
擬似終端`IND{>}`は、スタックの最上部のエントリよりも多くのスペースで構成されるインデントを示します。
`IND{=}`は、同じ数のスペースを持つインデントす示します。
`DED`は、スタックから値をポップするアクションを記述する別の疑似終端であり、
`IND{>}`はスタックへのプッシュを伴います。

この表記法により、文法の中核を簡単に定義できるようになりました。
ステートメントのブロック（簡単な例）

```
ifStmt = 'if' expr ':' stmt
         (IND{=} 'elif' expr ':' stmt)*
         (IND{=} 'else' ':' stmt)?

simpleStmt = ifStmt / ...

stmt = IND{>} stmt ^+ IND{=} DED  # list of statements
     / simpleStmt                 # or a simple statement
```

### コメント(Comments)
コメントは、文字列または文字リテラルの外側のどこからでもハッシュ記号`#`で始まります。
コメントはコメントピースの連結から成ります。
コメントピースは`#`から始まり、行の終わりまでです。
行末文字はコメントピースに属します。
次の行がコメントのみで構成されている場合、新しいコメントは開始されません。

```nim
i = 0     # これは複数行にまたがる単一のコメントです。
  # scannerはこれらコメントピースをマージします。
  # このコメントはここまでが一続きです
```
ドキュメントコメントは、`##`で始まるコメントです。
ドキュメントコメントはトークンであり、構文ツリーに属しているため、入力ファイルの特定の場所でのみ許可されます！

### 複数行コメント(Multiline comments)
Nimのバージョン0.13.0以降では複数行コメントができます。
例：

```nim
#[このように
複数行にわたって
コメントができます。]#
```
複数行コメントはネストができます。

```nim
#[  #[ Multiline comment in already
   commented out code. ]#
proc p[T](x: T) = discard
]#
```

複数行のドキュメントコメントも存在し、ネストもすることもできます。

```nim
proc foo =
  ##[Long documentation comment
  here.
  ]##
```

### 識別子とキーワード(Identifiers & Keywords)
Nimの識別子は、文字(letter)で始まり任意の文字(letter)、数字(digit)、アンダースコア`_`を組み合わせた文字列です。
連続したアンダースコア`__`は使用できません。

```nim
letter ::= 'A'..'Z' | 'a'..'z' | '\x80'..'\xff' #文字
digit ::= '0'..'9'#数字
IDENTIFIER ::= letter ( ['_'] (letter | digit) )* #一文字目は必ず文字（letter）である必要がある
```

現在、非ASCIIのUnicode文字(ordinal value > 127)は文字として分類されているため識別子に使えますが、言語の今後のバージョンで、Unicode文字が演算子に割り当てられる可能性があります。（訳者注：つまり、非ASCII文字を識別子に使うのは避けたほうが無難）

以下のキーワードは予約語であり、識別子として使用できません。
```
addr and as asm
bind block break
case cast concept const continue converter
defer discard distinct div do
elif else end enum except export
finally for from func
if import in include interface is isnot iterator
let
macro method mixin mod
nil not notin
object of or out
proc ptr
raise ref return
shl shr static
template try tuple type
using
var
when while
xor
yield
```
一部のキーワードは使用されていませんが、将来の言語開発のために予約されています。

### 識別子の等価性(Identifier equality)
次のアルゴリズムが真のとき、2つの識別子は等しいとみなされます。

```nim
proc sameIdentifier(a, b: string): bool =
  a[0] == b[0] and
    a.replace("_", "").toLowerAscii == b.replace("_", "").toLowerAscii
```
つまり、最初の文字のみ大文字と小文字を区別して比較されます。
他の文字はASCII範囲内で大文字と小文字を区別せずに比較され、アンダースコアは無視されます。

識別子の比較を行うこのかなり非正統的な方法は、partial case insensitivityと呼ばれ、従来の大文字と小文字を区別するよりもいくつかの利点があります。
これにより、プログラマーはhumpStyleでもsnake_styleでも、好みのスペルスタイルをほとんど使用でき、異なるプログラマーによって作成されたライブラリは互換性のない規則を使用できません。
Nim対応のエディターまたはIDEは、必要に応じて識別子を表示できます。
もう1つの利点は、プログラマーを識別子の正確なスペルを覚えることから解放することです。
最初の文字に関する例外により、`var foo:Foo`のような一般的なコードを明確に解析できます。

このルールにより、`notin`が`notIn`と`not_in`が同じ意味を持つことに注意して下さい。
（すべて小文字にすること(`notin`,`isnot`)がキーワードの好ましい記述法です）。

歴史的に、Nimは完全にスタイルに依存しない言語でした。
これは、大文字と小文字が区別されず、アンダースコアが無視され、fooとFooの区別さえないことを意味していました。

### 文字列リテラル(String literals)
文法の終端記号:`STR_LIT`

文字列リテラルは、ダブルクォーテーション`"`で囲まれ、以下のエスケープシーケンスを含めることができます。

|Escape<br>sequence|Meaning|
|:---|:---|
|`\p`|プラットフォーム固有の改行：WindowsではCRLF、UnixではLF|
|`\r`,`\c`|キャリッジリターン(carriage return)|
|`\n`,`\l`|line feed(よくnewlineとよばれる)|
|`\f`|form feed|
|`\t`|タブ(tabulator)|
|`\v`|垂直タブ(vertical tabulator)|
|`\\`|バックスラッシュ(backslash)|
|`\"`|クォーテーションマーク(quotation mark)|
|`\'`|アポストロフィ(apostrophe)|
|`\`'0'...'9'+|10進値dを持つ文字、`\`の直後に連続するすべての10進数の数字が使用されます|
|`\a`|アラート(alert)|
|`\b`|バックスペース(backspace)|
|`\e`|エスケープ(escape) [ESC]|
|`\x`HH|16進値HHを持つ文字。正確に2桁の16進数が許可されます|
|`\u`HHHH|16進値HHHHを持つUnicodeコードポイント。正確に4桁の16進数が許可されます|
|`\u`{H+}|Unicodeコードポイント。`{}`で囲まれたすべての16進数がコードポイントに使用されます|

Nimの文字列には、ゼロが埋め込まれていても8ビット値を含めることができます。ただし、一部の演算では、最初のバイナリゼロが終端として解釈される場合があります。

### 三重引用符付き文字列リテラル(Triple quoted string literals)
文法の終端記号:`TRIPLESTR_LIT`

文字列リテラルは、3つのダブルクォーテーション`"""`...`"""`で囲むこともできます。
この形式の文字列リテラルは複数行にまたがることができ、`"`を含むことができ、エスケープシーケンスを解釈しません。
便宜上、開始側の`"""`の後に改行が続く場合（`"""`と改行の間に空白がある場合もあります）、改行（およびその前の空白）は文字列に含まれません。
文字列リテラルの末尾は、パターン`"""[^"]`で定義されます。
そのため、

```nim
""""long string within quotes""""
```
は
```
"long string within quotes"
```
を表します。

### 生文字列リテラル(Raw string literals)
文法の終端記号:`RSTR_LIT`

`r`(または`R`)で始まり、通常の文字列リテラルと同様に`"`で囲まれた生文字列リテラルも使用できます。
これはエスケープシーケンスを解釈せず、正規表現やWIndowsパスを表すのに特に便利です。

```nim
var f = openFile(r"C:\texts\text.txt") # 生文字列なので"\t"はタブではありません
```
生文字列リテラル内で`"`を表すには2つ並べる必要があります。

```nim
r"a""b"
```
は
```
a"b
```
を意味します。

この表記法では`r""""`は使用できません。
それは3連続の`"`は三重引用符付き文字列リテラルとなるからです。
三重引用符付き文字列リテラルはエスケープシーケンスを解釈しないため、`r"""`は`"""`と同じになります。

### 一般化生文字列リテラル(Generalized raw string literals)
文法の終端記号：`GENERALIZED_STR_LIT`,`GENERALIZED_TRIPLESTR_LIT`

`idenfier"stringliteral"`(識別子(identifier)と`"`の間に空白なし)は一般化生文字列リテラルです。
これは`identifier(r"string literal")`の短縮形であり、生文字列リテラルを唯一の引数とするプロシージャ呼び出しを表します。
一般化生文字列リテラルはミニ言語を直接Nimに埋め込むのに便利です。（例えば、正規表現）

`idenfier"""stringliteral"""`も存在します。これは`idenfier("""stringliteral""")`の短縮形です。

### 文字リテラル(Character literals)
文字リテラルは`'`で囲まれ、文字列と同じエスケープシーケンスを含むことができます。
ただし、プラットフォーム依存の改行(`\p`)は、1文字よりも多くなることがあるため(CR+LFなど)除外されます。
文字リテラルの有効なエスケープシーケンスは次のとおりです。

|Escape<br>sequence|Meaning|
|:---|:---|
|`\r`,`\c`|キャリッジリターン(carriage return)|
|`\n`,`\l`|line feed(よくnewlineとよばれる)|
|`\f`|form feed|
|`\t`|タブ(tabulator)|
|`\v`|垂直タブ(vertical tabulator)|
|`\\`|バックスラッシュ(backslash)|
|`\"`|クォーテーションマーク(quotation mark)|
|`\'`|アポストロフィ(apostrophe)|
|`\`'0'...'9'+|10進値dを持つ文字、`\`の直後に連続するすべての10進数の数字が使用されます|
|`\a`|アラート(alert)|
|`\b`|バックスペース(backspace)|
|`\e`|エスケープ(escape) [ESC]|
|`\x`HH|16進値HHを持つ文字。正確に2桁の16進数が許可されます|

文字(character)はユニコード文字ではなく、1バイトです。
この理由は効率性です：圧倒的多数のユースケースのために、UTF-8は特別に設計されているため、結果のプログラムは依然としてUTF-8を適切に処理します。
もう1つの理由は、多くのアルゴリズムが依存している`array[char,int]`または`set[char]`を効率的にサポートできるからです。
`Rune`型はUnicode文字のために用いられ、任意のUnicode文字を表すことができます。
`Rune`は[unicodeモジュール](https://nim-lang.org/docs/unicode.html)で宣言されています。

### 数値リテラル(Numerical constants)
数値リテラルは単一のタイプで、以下の形式をとります。

```
hexdigit = digit | 'A'..'F' | 'a'..'f'
octdigit = '0'..'7'
bindigit = '0'..'1'
HEX_LIT = '0' ('x' | 'X' ) hexdigit ( ['_'] hexdigit )*
DEC_LIT = digit ( ['_'] digit )*
OCT_LIT = '0' 'o' octdigit ( ['_'] octdigit )*
BIN_LIT = '0' ('b' | 'B' ) bindigit ( ['_'] bindigit )*

INT_LIT = HEX_LIT
        | DEC_LIT
        | OCT_LIT
        | BIN_LIT

INT8_LIT = INT_LIT ['\''] ('i' | 'I') '8'
INT16_LIT = INT_LIT ['\''] ('i' | 'I') '16'
INT32_LIT = INT_LIT ['\''] ('i' | 'I') '32'
INT64_LIT = INT_LIT ['\''] ('i' | 'I') '64'

UINT_LIT = INT_LIT ['\''] ('u' | 'U')
UINT8_LIT = INT_LIT ['\''] ('u' | 'U') '8'
UINT16_LIT = INT_LIT ['\''] ('u' | 'U') '16'
UINT32_LIT = INT_LIT ['\''] ('u' | 'U') '32'
UINT64_LIT = INT_LIT ['\''] ('u' | 'U') '64'

exponent = ('e' | 'E' ) ['+' | '-'] digit ( ['_'] digit )*
FLOAT_LIT = digit (['_'] digit)* (('.' digit (['_'] digit)* [exponent]) |exponent)
FLOAT32_SUFFIX = ('f' | 'F') ['32']
FLOAT32_LIT = HEX_LIT '\'' FLOAT32_SUFFIX
            | (FLOAT_LIT | DEC_LIT | OCT_LIT | BIN_LIT) ['\''] FLOAT32_SUFFIX
FLOAT64_SUFFIX = ( ('f' | 'F') '64' ) | 'd' | 'D'
FLOAT64_LIT = HEX_LIT '\'' FLOAT64_SUFFIX
            | (FLOAT_LIT | DEC_LIT | OCT_LIT | BIN_LIT) ['\''] FLOAT64_SUFFIX
```
数値リテラルは可読性のために`_`を含めることができます。
整数及び浮動小数点リテラルは10進数(no prefix)、2進数(prefix `0b`)、8進数(prefix `0o`)、16進数(prefix `0x`)で記述できます。

定義されている各数値型にはリテラルが存在します。
アポストロフィ`'`で始まるサフィックスは、型サフィックスと呼ばれます。
型サフィックスのない数値リテラルは`.`または`E|e`がなければ整数型、あれば`float`型です。
この整数型はリテラルが`low(i32)..high(i32)`であれば`int`型、そうでなければ`int64`型です。
表記の利便性のため、`'`がないことで曖昧にならない場合は型サフィックスの`'`はオプションです。
(型サフィックスを持つ16進数浮動小数点のみが曖昧になります)
(訳者注:16進数は浮動小数点の型サフィックス`d|f|D|F`を含むため、数値部分とサフィックスを区別する必要がある)

型サフィックスは以下である。

|型サフィックス|結果のリテラルタイプ|
|:---|:---|
|'i8|int8|
|'i16|int16|
|'i32|int32|
|'i64|int64|
|'u|uint|
|'u8|uint8|
|'u16|uint16|
|'u32|uint32|
|'u64|uint64|
|'f|float32|
|'d|float64|
|'f32|float32|
|'f64|float64|

浮動小数点リテラルは、2進,8進,16進表記を使用できます。
`0B0_10001110100_0000101001000111101011101111111011000101001101001001'f64`は、IEEE浮動小数点標準に従って約1.72826e35です。

リテラルは、データ型に適合するように境界がチェックされます。
10進数以外のリテラルは主にフラグとビットパターン表現に使用されるため、値の範囲ではなくビット幅で境界チェックが行われます。
リテラルがデータ型のビット幅に適合する場合、受け入れられます。
したがって、0b10000000'u8 == 0x80'u8 == 128ですが、0b10000000'i8 == 0x80'i8 == -1となり、オーバーフローエラーとなりません。

### 演算子(Operators)
Nimでは、ユーザー定義の演算子を使用できます。
演算子は、次の文字の任意の組み合わせです。
```
=     +     -     *     /     <     >
@     $     ~     &     %     |
!     ?     ^     .     :     \
```
次のキーワードも演算子です:`and or not xor shl shr div mod in notin is isnot of`

. =, :, :: は一般的な演算子として使用できません。これらは他の表記上の目的に使用されます。

`*:`は、2つのトークン`*`および`:`として扱われる特別なケースです。（`var v*: T`をサポートするため）

`not`は常に単項演算子であり、`a not b`は`a(not b)`として解析され、`(a) not (b)`ではない。

### その他のトークン(Other tokens)
次の文字列は他のトークンを示します。
```
`   (    )     {    }     [    ]    ,  ;   [.    .]  {.   .}  (.  .)  [:
```
スライストークン`..`は他の`.`を含むトークンよりも優先されます。
`{..}`は3つのトークン`{`,`..`,`}`であって、2つのトークン`{.`,`.}`ではありません。

## 構文(Syntax)
このセクションでは、Nimの標準構文をリストします。
パーサーがインデントを処理する方法は、[字句解析](#字句解析Lexical-Analysis)セクションで既に説明されています。

Nimでは、ユーザー定義可能な演算子を使用できます。二項演算子には、11の優先順位のレベルがあります。

### 結合性(Associativity)
最初の文字が`^`である二項演算子は右結合であり、他のすべての二項演算子は左結合です。

```nim
proc `^/`(x, y: float): float =
  # 右結合除算演算子
  result = x / y
echo 12 ^/ 4 ^/ 8 # 24.0 (4 / 8 = 0.5, then 12 / 0.5 = 24.0)
echo 12  / 4  / 8 # 0.375 (12 / 4 = 3.0, then 3 / 8 = 0.375)
```

### 優先順位(Precedence)
単項演算子は常に2項演算子よりも強く結合します:`$a + b` は`($a) + b`であって、`$(a + b)`ではありません。

単項演算子の最初の文字が@であるときは`primarySuffix`よりも強く結合するsigilの様な演算子です:`@x.abc`は`(@x).abc`とパースされ、`$x.abc`は`$(x.abc)`とパースされます。

キーワードでない2項演算子の場合、優先順位は以下の規則によって決定されます。
`->`,`~>`,`=>`で終わる演算子はarrow likeと呼ばれ、すべての演算子の中で最も優先順位が低いです。

`=`で終わり、最初の文字が`<`,`>`,`!`,`=`,`~`,`?`以外の演算子は代入演算子と呼ばれ2番目に優先順位が低いです。

それ以外の場合、優先順位は最初の文字によって決まります。

|優先レベル|演算子|頭文字|Terminal symbol|
|:---|:---|:---|:---|
|10(highest)||`$ ^`|OP10|
|9|`* / div mod shl shr %`|`* % \ /`|OP9|
|8|`+ -`|`+ - ~ |`|OP8|
|7|`&`|`&`|OP7|
|6|`..`|`.`|OP6|
|5|`== <= < >= > != in notin is isnot not of`|`= < > !`|OP5|
|4|`and`||OP4|
|3|`or xor`||OP3|
|2||`@ : ?`|OP2|
|1|代入演算子 (例 `+=`,`*=`)||OP1|
|0 (lowest)|arrow like operator (例 `->`,`=>`)||OP0|

演算子がprefix演算子として使用されるかどうかは、先行する空白の影響も受けます。
(このパースの変更はバージョン0.13.0で導入されました)

```nim
echo $foo
# is parsed as
echo($foo)
```
空白は`(a,b)`が引数リストとしてパースされるか、タプルコンストラクタとしてパースされるかも空白によって左右されます。

```nim
echo(1, 2) # 1と2をechoに渡す
```

```nim
echo (1, 2) # タプル(1, 2)をechoに渡す
```

### 文法(Grammar)
文法の開始記号は`module`です。

```
# This file is generated by compiler/parser.nim.
module = stmt ^* (';' / IND{=})
comma = ',' COMMENT?
semicolon = ';' COMMENT?
colon = ':' COMMENT?
colcom = ':' COMMENT?
operator =  OP0 | OP1 | OP2 | OP3 | OP4 | OP5 | OP6 | OP7 | OP8 | OP9
         | 'or' | 'xor' | 'and'
         | 'is' | 'isnot' | 'in' | 'notin' | 'of'
         | 'div' | 'mod' | 'shl' | 'shr' | 'not' | 'static' | '..'
prefixOperator = operator
optInd = COMMENT? IND?
optPar = (IND{>} | IND{=})?
simpleExpr = arrowExpr (OP0 optInd arrowExpr)* pragma?
arrowExpr = assignExpr (OP1 optInd assignExpr)*
assignExpr = orExpr (OP2 optInd orExpr)*
orExpr = andExpr (OP3 optInd andExpr)*
andExpr = cmpExpr (OP4 optInd cmpExpr)*
cmpExpr = sliceExpr (OP5 optInd sliceExpr)*
sliceExpr = ampExpr (OP6 optInd ampExpr)*
ampExpr = plusExpr (OP7 optInd plusExpr)*
plusExpr = mulExpr (OP8 optInd mulExpr)*
mulExpr = dollarExpr (OP9 optInd dollarExpr)*
dollarExpr = primary (OP10 optInd primary)*
symbol = '`' (KEYW|IDENT|literal|(operator|'('|')'|'['|']'|'{'|'}'|'=')+)+ '`'
       | IDENT | KEYW
exprColonEqExpr = expr (':'|'=' expr)?
exprList = expr ^+ comma
exprColonEqExprList = exprColonEqExpr (comma exprColonEqExpr)* (comma)?
dotExpr = expr '.' optInd (symbol | '[:' exprList ']')
explicitGenericInstantiation = '[:' exprList ']' ( '(' exprColonEqExpr ')' )?
qualifiedIdent = symbol ('.' optInd symbol)?
setOrTableConstr = '{' ((exprColonEqExpr comma)* | ':' ) '}'
castExpr = 'cast' '[' optInd typeDesc optPar ']' '(' optInd expr optPar ')'
parKeyw = 'discard' | 'include' | 'if' | 'while' | 'case' | 'try'
        | 'finally' | 'except' | 'for' | 'block' | 'const' | 'let'
        | 'when' | 'var' | 'mixin'
par = '(' optInd
          ( &parKeyw complexOrSimpleStmt ^+ ';'
          | ';' complexOrSimpleStmt ^+ ';'
          | pragmaStmt
          | simpleExpr ( ('=' expr (';' complexOrSimpleStmt ^+ ';' )? )
                       | (':' expr (',' exprColonEqExpr     ^+ ',' )? ) ) )
          optPar ')'
literal = | INT_LIT | INT8_LIT | INT16_LIT | INT32_LIT | INT64_LIT
          | UINT_LIT | UINT8_LIT | UINT16_LIT | UINT32_LIT | UINT64_LIT
          | FLOAT_LIT | FLOAT32_LIT | FLOAT64_LIT
          | STR_LIT | RSTR_LIT | TRIPLESTR_LIT
          | CHAR_LIT
          | NIL
generalizedLit = GENERALIZED_STR_LIT | GENERALIZED_TRIPLESTR_LIT
identOrLiteral = generalizedLit | symbol | literal
               | par | arrayConstr | setOrTableConstr
               | castExpr
tupleConstr = '(' optInd (exprColonEqExpr comma?)* optPar ')'
arrayConstr = '[' optInd (exprColonEqExpr comma?)* optPar ']'
primarySuffix = '(' (exprColonEqExpr comma?)* ')' doBlocks?
      | doBlocks
      | '.' optInd symbol generalizedLit?
      | '[' optInd indexExprList optPar ']'
      | '{' optInd indexExprList optPar '}'
      | &( '`'|IDENT|literal|'cast'|'addr'|'type') expr # command syntax
condExpr = expr colcom expr optInd
        ('elif' expr colcom expr optInd)*
         'else' colcom expr
ifExpr = 'if' condExpr
whenExpr = 'when' condExpr
pragma = '{.' optInd (exprColonExpr comma?)* optPar ('.}' | '}')
identVis = symbol opr?  # postfix position
identVisDot = symbol '.' optInd symbol opr?
identWithPragma = identVis pragma?
identWithPragmaDot = identVisDot pragma?
declColonEquals = identWithPragma (comma identWithPragma)* comma?
                  (':' optInd typeDesc)? ('=' optInd expr)?
identColonEquals = ident (comma ident)* comma?
     (':' optInd typeDesc)? ('=' optInd expr)?)
inlTupleDecl = 'tuple'
    [' optInd  (identColonEquals (comma/semicolon)?)*  optPar ']'
extTupleDecl = 'tuple'
    COMMENT? (IND{>} identColonEquals (IND{=} identColonEquals)*)?
tupleClass = 'tuple'
paramList = '(' declColonEquals ^* (comma/semicolon) ')'
paramListArrow = paramList? ('->' optInd typeDesc)?
paramListColon = paramList? (':' optInd typeDesc)?
doBlock = 'do' paramListArrow pragmas? colcom stmt
procExpr = 'proc' paramListColon pragmas? ('=' COMMENT? stmt)?
distinct = 'distinct' optInd typeDesc
forStmt = 'for' (identWithPragma ^+ comma) 'in' expr colcom stmt
forExpr = forStmt
expr = (blockExpr
      | ifExpr
      | whenExpr
      | caseExpr
      | forExpr
      | tryExpr)
      / simpleExpr
typeKeyw = 'var' | 'out' | 'ref' | 'ptr' | 'shared' | 'tuple'
         | 'proc' | 'iterator' | 'distinct' | 'object' | 'enum'
primary = typeKeyw typeDescK
        /  prefixOperator* identOrLiteral primarySuffix*
        / 'bind' primary
typeDesc = simpleExpr
typeDefAux = simpleExpr
           | 'concept' typeClass
postExprBlocks = ':' stmt? ( IND{=} doBlock
                           | IND{=} 'of' exprList ':' stmt
                           | IND{=} 'elif' expr ':' stmt
                           | IND{=} 'except' exprList ':' stmt
                           | IND{=} 'else' ':' stmt )*
exprStmt = simpleExpr
         (( '=' optInd expr colonBody? )
         / ( expr ^+ comma
             doBlocks
              / macroColon
           ))?
importStmt = 'import' optInd expr
              ((comma expr)*
              / 'except' optInd (expr ^+ comma))
includeStmt = 'include' optInd expr ^+ comma
fromStmt = 'from' moduleName 'import' optInd expr (comma expr)*
returnStmt = 'return' optInd expr?
raiseStmt = 'raise' optInd expr?
yieldStmt = 'yield' optInd expr?
discardStmt = 'discard' optInd expr?
breakStmt = 'break' optInd expr?
continueStmt = 'break' optInd expr?
condStmt = expr colcom stmt COMMENT?
           (IND{=} 'elif' expr colcom stmt)*
           (IND{=} 'else' colcom stmt)?
ifStmt = 'if' condStmt
whenStmt = 'when' condStmt
whileStmt = 'while' expr colcom stmt
ofBranch = 'of' exprList colcom stmt
ofBranches = ofBranch (IND{=} ofBranch)*
                      (IND{=} 'elif' expr colcom stmt)*
                      (IND{=} 'else' colcom stmt)?
caseStmt = 'case' expr ':'? COMMENT?
            (IND{>} ofBranches DED
            | IND{=} ofBranches)
tryStmt = 'try' colcom stmt &(IND{=}? 'except'|'finally')
           (IND{=}? 'except' exprList colcom stmt)*
           (IND{=}? 'finally' colcom stmt)?
tryExpr = 'try' colcom stmt &(optInd 'except'|'finally')
           (optInd 'except' exprList colcom stmt)*
           (optInd 'finally' colcom stmt)?
exceptBlock = 'except' colcom stmt
blockStmt = 'block' symbol? colcom stmt
blockExpr = 'block' symbol? colcom stmt
staticStmt = 'static' colcom stmt
deferStmt = 'defer' colcom stmt
asmStmt = 'asm' pragma? (STR_LIT | RSTR_LIT | TRIPLESTR_LIT)
genericParam = symbol (comma symbol)* (colon expr)? ('=' optInd expr)?
genericParamList = '[' optInd
  genericParam ^* (comma/semicolon) optPar ']'
pattern = '{' stmt '}'
indAndComment = (IND{>} COMMENT)? | COMMENT?
routine = optInd identVis pattern? genericParamList?
  paramListColon pragma? ('=' COMMENT? stmt)? indAndComment
commentStmt = COMMENT
section(p) = COMMENT? p / (IND{>} (p / COMMENT)^+IND{=} DED)
constant = identWithPragma (colon typeDesc)? '=' optInd expr indAndComment
enum = 'enum' optInd (symbol optInd ('=' optInd expr COMMENT?)? comma?)+
objectWhen = 'when' expr colcom objectPart COMMENT?
            ('elif' expr colcom objectPart COMMENT?)*
            ('else' colcom objectPart COMMENT?)?
objectBranch = 'of' exprList colcom objectPart
objectBranches = objectBranch (IND{=} objectBranch)*
                      (IND{=} 'elif' expr colcom objectPart)*
                      (IND{=} 'else' colcom objectPart)?
objectCase = 'case' identWithPragma ':' typeDesc ':'? COMMENT?
            (IND{>} objectBranches DED
            | IND{=} objectBranches)
objectPart = IND{>} objectPart^+IND{=} DED
           / objectWhen / objectCase / 'nil' / 'discard' / declColonEquals
object = 'object' pragma? ('of' typeDesc)? COMMENT? objectPart
typeClassParam = ('var' | 'out')? symbol
typeClass = typeClassParam ^* ',' (pragma)? ('of' typeDesc ^* ',')?
              &IND{>} stmt
typeDef = identWithPragmaDot genericParamList? '=' optInd typeDefAux
            indAndComment?
varTuple = '(' optInd identWithPragma ^+ comma optPar ')' '=' optInd expr
colonBody = colcom stmt doBlocks?
variable = (varTuple / identColonEquals) colonBody? indAndComment
bindStmt = 'bind' optInd qualifiedIdent ^+ comma
mixinStmt = 'mixin' optInd qualifiedIdent ^+ comma
pragmaStmt = pragma (':' COMMENT? stmt)?
simpleStmt = ((returnStmt | raiseStmt | yieldStmt | discardStmt | breakStmt
           | continueStmt | pragmaStmt | importStmt | exportStmt | fromStmt
           | includeStmt | commentStmt) / exprStmt) COMMENT?
complexOrSimpleStmt = (ifStmt | whenStmt | whileStmt
                    | tryStmt | forStmt
                    | blockStmt | staticStmt | deferStmt | asmStmt
                    | 'proc' routine
                    | 'method' routine
                    | 'iterator' routine
                    | 'macro' routine
                    | 'template' routine
                    | 'converter' routine
                    | 'type' section(typeDef)
                    | 'const' section(constant)
                    | ('let' | 'var' | 'using') section(variable)
                    | bindStmt | mixinStmt)
                    / simpleStmt
stmt = (IND{>} complexOrSimpleStmt^+(IND{=} / ';') DED)
     / simpleStmt ^+ ';'
```

### 評価の順序(Order of evaluation)
評価の順序は、他のほとんどの言語と同様に、厳密に左から右、内から外です。

```nim
var s = ""

proc p(arg: int): int =
  s.add $arg
  result = arg

discard p(p(1) + p(2))

doAssert s == "123"
```

代入は特別ではありません。左辺の式は右辺の前に評価されます。

```nim
var v = 0
proc getI(): int =
  result = v
  inc v

var a, b: array[0..2, int]

proc someCopy(a: var int; b: int) = a = b

a[getI()] = getI()

doAssert a == [1, 0, 0]

v = 0
someCopy(b[getI()], getI())

doAssert b == [1, 0, 0]
```
根拠:assignment演算子とassignment-like演算子の一貫性により、`a = b`は`performSomeCopy(a, b)`と読むことができます。

ただし、「評価の順序」の概念は、コードが正規化された後にのみ適用されます。
正規化には、テンプレート展開と名前付きパラメーターとして渡された引数の並べ替えが含まれます。

```nim
var s = ""

proc p(): int =
  s.add "p"
  result = 5

proc q(): int =
  s.add "q"
  result = 3

# テンプレートの展開により、bはaよりも先に評価されます。
# expansion's semantics.
template swapArgs(a, b): untyped =
  b + a

# "q() - p()"は"p() + q()"よりも先に評価されます。
doAssert swapArgs(p() + q(), q() - p()) == 6
doAssert s == "qppq"

# 評価の順序は名前付きパラメータの影響を受けません。
proc construct(first, second: int) =
  discard

# "p"は"q"よりも先に評価されます!
construct(second = q(), first = p())

doAssert s == "qppqpq"
```
根拠:これは仮想的な代替案よりも実装がはるかに簡単です。

### 定数と定数式
定数は定数式の値にバインドされているシンボルです。
以下のカテゴリの値と演算のみに依存するように制限されています。
それは、これらが言語に組み込まれているか定数式のセマンティック解析の前に宣言および評価されるためです。

- リテラル
- 組み込み演算子
- 宣言済み定数とコンパイル時変数
- 宣言済みマクロとテンプレート
- コンパイル時変数を変更するよりも大きな副作用を持たない宣言済みプロシージャ

定数式にはコンパイル時にサポートされるNimの機能(次節で説明)を内部的に使用するコードブロックを含めることができます。
そのようなコードブロック内で、変数を宣言したのちに読み取り、更新をするか、変数を宣言して変数を変更するプロシージャに渡すことができます。
ただし、そのようなブロック内のコードは、ブロック外の値と演算を参照するための上記の制限に従う必要があります。

他の静的型付け言語から来た人は驚くかもしれせんが、コンパイル時変数にアクセスして変更する機能は定数式に柔軟性を追加します。
例えば、次のコードはコンパイル時にフィボナッチ数列の始まり部分を出力します。
（これは定数の定義における柔軟性のデモンストレーションであり、この問題を解決するための推奨スタイルではありません！）

```nim
import strformat

var fib_n {.compileTime.}: int
var fib_prev {.compileTime.}: int
var fib_prev_prev {.compileTime.}: int

proc next_fib(): int =
  result = if fib_n < 2:
    fib_n
  else:
    fib_prev_prev + fib_prev
  inc(fib_n)
  fib_prev_prev = fib_prev
  fib_prev = result

const f0 = next_fib()
const f1 = next_fib()

const display_fib = block:
  const f2 = next_fib()
  var result = fmt"Fibonacci sequence: {f0}, {f1}, {f2}"
  for i in 3..12:
    add(result, fmt", {next_fib()}")
  result

static:
  echo display_fib
```

## コンパイル時実行の制限
コンパイル時に実行されるNimコードは、次の言語機能を使用できません。

- methods
- closure iterators
- キャスト演算子
- 参照(ポインタ)型
- the FFI

これらの制限の一部または全ては今後、解除される可能性があります。

## 型(Types)
全ての式はセマンティック解析中に既知となる型を持ちます。
Nimは静的型付けされます。
新しい型を定義できます。
これは本質的にはカスタム型を表すために使われる識別子を定義することです。

主な型クラスは以下の通りです。

- 順序型(整数型、ブール型、文字型、列挙型(及びそれらの部分範囲型)で構成されます)
- 浮動小数点型
- 文字列型
- 構造型(structured types)
- 参照(ポインタ)型
- プロシージャー型(procedural type)
- 総称型(generic type)

### 順序型(Ordinal types)
順序型は以下の特性があります。

- 順序型は加算かつ順序付けられます。この性質により、順序型において`inc`,`ord`,`dec`などの関数の演算を定義できます。
- 順序型には最小値があります。最小値よりもさらにカウントダウンしようとすると実行時チェックまたは静的エラーとなります。
- 順序型には最大値があります。最大値よりもさらにカウントしようとすると実行時チェックまたは静的エラーとなります。

整数型、ブール型、文字型、列挙型(及びそれらの部分範囲型)は順序型に属します。
実装を単純にするため、`uint`型と`uint64`型は順序型ではありません。
(これは今後のバージョンで変更されます。)

distinct型はその基となる配列が順序型のとき、順序型となります。

### 定義済み整数型(Pre-defined integer types)
以下の整数型は定義済みです。

- `int`

汎用符号付整数型。そのサイズはプラットフォームに依存し、ポインターと同じサイズです。
このタイプは一般的に使用されるべきです。
型サフィックスを持たない整数リテラルは、`low(int32)..high(int32)`の範囲内にある場合、この型になります。
それ以外の場合、リテラルの型は`int64`です。

- intXX

XXビット符号付き整数型は、この命名スキームを使用します（例:`int16`は16ビット幅の整数です）。
現在の実装では、`int8`,`int16`,`int32`,`int64`がサポートされています。
これらのタイプのリテラルには、サフィックス'iXXが付いています。

- `uint`

汎用の符号なし整数型。
そのサイズはプラットフォームに依存し、ポインターと同じサイズです。
型の接尾辞が'uの整数リテラルはこの型です。

- uintXX

XXビット符号なし整数型は、この命名スキームを使用します（例:`uint16`は16ビット幅の符号なし整数です）。
現在の実装では、`uint8`,`uint16`,`uint32`,`uint64`がサポートされています。
これらのタイプのリテラルには、サフィックス'uXXが付いています。
符号なし演算はすべてラップアラウンドします。
オーバーフローまたはアンダーフローエラーにつながることはありません。

符号付きと符号なし整数の通常の算術演算子(`+ - *`など)に加え、符号付き整数で正式に機能するが引数を符号なしとして扱う演算子もあります。
それらはほとんどが、符号なし整数がなかった旧バージョンとの後方互換性のために提供されています。

|演算|意味|
|:---|:---|
|`a +% b`|符号なし整数加算|
|`a -% b`|符号なし整数減算|
|`a *% b`|符号なし整数乗算|
|`a /% b`|符号なし整数除算|
|`a %% b`|符号なし整数剰余演算|
|`a <% b`|`a`と`b`を符号なしとして比較|
|`a <=% b`|`a`と`b`を符号なしとして比較|
|`ze(a)`|`int`型の幅になるまでa`のビットを0で拡張|
|`toU8(a)`|`a`を符号なしとして扱い、8ビット符号なし整数に変換(型はint8のまま)|
|`toU16(a)`|`a`を符号なしとして扱い、16ビット符号なし整数に変換(型はint16のまま)|
|`toU32(a)`|`a`を符号なしとして扱い、32ビット符号なし整数に変換(型はint32のまま)|

自動型変換は、異なる種類の整数型が使用される式で実行され、小さな型が大きな型に変換されます。

縮小変換は大きい型を小さい型に変換します(例:`int32 -> int16`)。
拡大変換は小さい型を大きい型に変換します(例:`int16 -> int32`)。
Nimでは拡大変換のみが暗黙的です。

```nim
var myInt16 = 5i16
var myInt: int
myInt16 + 34     # of type ``int16``
myInt16 + myInt  # of type ``int``
myInt16 + 2i32   # of type ``int32``
```
ただし、リテラルの値が小さい型に適合する場合、他の暗黙的な変換よりも安価なので`int`リテラルは暗黙的に小さい整数型に変換されます。
従って、`myInt16 + 34`は`int16`となります。

### Subrange型(Subrange types)
subrange型はその基本型となる順序型または浮動小数点型の範囲の値を持ちます。
subrange型を定義するには、その制限値（型の最小値と最大値）を指定する必要があります。
例えば、

```nim
type
  Subrange = range[0..5]
  PositiveFloat = range[0.0..Inf]
```
`Subrange`は0から5の整数のみをとれるsubrange型である。
`PositiveFloat`は0以上の浮動小数点のsubrange型を定義します。
NaNはいかなる浮動小数点型のSubrange型にも属しません。
subrange型の変数にそれ以外の値を代入すると、実行時エラーとなります(セマンティック解析中に決まる場合は静的エラー)。
ベース型からそのsubrange型の1つへの代入（およびその逆）が許可されます。

subrange型のサイズは、そのベース型と同じサイズです（`Subrange`の例では`int`型）。

### 定義済み浮動小数点型(Pre-defined floating point types)
次の浮動小数点型が定義済みです。

- `float`
一般的な浮動小数点型。以前はそのサイズはプラットフォームに依存していましたが、現在は常に`float64`にマップされます。このタイプは一般的に使用されるべきです。

- floatXX
実装は、この命名スキームを使用してXXビット浮動小数点タイプを定義できます(例:`float64`は64ビット幅浮動小数点型です)。
現在の実装は、`float32`と`float64`をサポートしています。これらのタイプのリテラルには、サフィックス'fXXが付いています。

異なる種類の浮動小数点型の式で自動型変換が実行されます。
詳細については、[変換可能な関係](#変換可能な関係Convertible-relation)を参照してください。
浮動小数点型で実行される算術演算は、IEEE標準に従います。
整数型は浮動小数点型に自動的に変換されず、その逆も行われません。

IEEE標準では、以下の5種類の浮動小数点例外を定義しています。

- 無効(invalid) : 0.0/0.0,sqrt(-1.0),log(-37.8)などの数学的に無効なオペランドを使用した演算。
- ゼロ除算 : 1.0/0.0などの除数がゼロで、被除数が有限なゼロ以外の数値である演算。
- オーバーフロー : MAXDOUBLE+0.0000000000001e308などの指数の範囲を超える結果となる演算。
- アンダーフロー : MINDOUBLE * MINDOUBLEなどの通常の数値として表現するには小さすぎる結果となる演算。
- 不正確(inexact) : 2.0/3.0,log(1.1),0.1などの任意精度で表現できない結果となる演算。

IEEE例外は実行中に無視されるか、Nimの例外(FloatInvalidOpError,FloatDivByZeroError,FloatOverflowError,FloatUnderflowError,FloatInexactError)にマップされます。
これらの例外はFloatingPointError基本クラスから継承されます。

Nimは、IEEE例外を無視するかNim例外をトラップするかを制御するプラグマnanChecksおよびinfChecksを提供します。

```nim
{.nanChecks: on, infChecks: on.}
var a = 1.0
var b = 0.0
echo b / b # raises FloatInvalidOpError
echo a / b # raises FloatOverflowError
```
現在の実装では、`FloatDivByZeroError`と`FloatInexactError`は発生しません。
`FloatDivByZeroError`の代わりに`FloatOverflowError`が発生します。
nanChecksプラグマとinfChecksプラグマの組み合わせのショートカットであるfloatChecksプラグマもありますが、デフォルトでオフになっています

`floatChecks`プラグマの影響を受ける操作は、浮動小数点型の`+`,`-`,`*`,`/`演算子のみです。

実装では、セマンティック解析中に浮動小数点値を評価するために利用可能な最大精度を常に使用する必要があります。
これは、`0.09'f32 + 0.01'f32 == 0.09'f64 + 0.01'f64`のような定数畳み込み中に評価される式が真であることを意味します。

### ブール型(Boolean type)
ブール型は、Nimではboolという名前で、2つの定義済みの値`true`および`false`のいずれかです。
while、if、elif、when文の条件は`bool`型である必要があります。

この条件は次のとおりです。

```nim
ord(false) == 0 and ord(true) == 1
```

`not, and, or, xor, <, <=, >, >=, !=, ==`の演算子がブール型に対して定義されています。
`and`と`or`は短絡評価されます。例えば、

```nim
while p != nil and p.name != "xyz":
  # p == nil の時、p.name は評価されません
  p = p.next
```
bool型のサイズは1バイトです。

### 文字型(Character type)
文字型は、Nimではcharという名前で、サイズは1バイトです。
従って、UTF-8文字を表すことはできませんが、その一部を表します。
この理由は効率性です：圧倒的多数のユースケースのために、UTF-8は特別に設計されているため、結果のプログラムは依然としてUTF-8を適切に処理します。
もう1つの理由は、多くのアルゴリズムが依存している`array[char,int]`または`set[char]`を効率的にサポートできるからです。
`Rune`型はUnicode文字のために用いられ、任意のUnicode文字を表すことができます。
`Rune`は[unicodeモジュール](https://nim-lang.org/docs/unicode.html)で宣言されています。

### 列挙型(Enumeration types)
列挙型は、指定された値で構成される値を持つ新しい型を定義します。値は順序付けられています。例えば

```nim
type
  Direction = enum
    north, east, south, west
```
これは以下のことが成り立ちます。

```nim
ord(north) == 0
ord(east) == 1
ord(south) == 2
ord(west) == 3

# Also allowed:
ord(Direction.west) == 3
```
従って、north < east < south < westです。
`north`などの代わりに、列挙値はそれが存在する列挙型で修飾し`Direction.north`とも書けます。

他のプログラミング言語とのインターフェイスを改善するために、列挙型のフィールドに明示的な序数値を割り当てることができます。
ただし、序数値は昇順である必要があります。
順序値が明示的に指定されていないフィールドには、前のフィールドの値+1が割り当てられます

明示的に順序付けられた列挙型には、抜けがある場合があります。

```nim
type
  TokenType = enum
    a = 2, b = 4, c = 89 # holes are valid
```
ただし、これは序数ではないため、これらの列挙型を配列のインデックス型として使用することはできません。
プロシージャ`inc`,`dec`,`succ`,`pred`も使用できません。

コンパイラーは、列挙型に対して組み込みのstringify演算子`$`をサポートします。
stringifyの結果は、使用する文字列値を明示的に指定することで制御できます。

```nim
type
  MyEnum = enum
    valueA = (0, "my value A"),
    valueB = "value B",
    valueC = 2,
    valueD = (3, "abc")
```
例からわかるように、タプルを使用してフィールドの序数値とその文字列値の両方を指定することができまし、それらの内の1つだけを指定することもできます。

列挙型は、最後の試行としてのみ照会される特別なモジュール固有の隠されたスコープにフィールドが追加されるように、`pure`プラグマでマークできます。
このスコープには、あいまいでないシンボルのみが追加されます。
ただし、これらには`MyEnum.value`のように型修飾を記述することで常にアクセスできます。

```nim
type
  MyEnum {.pure.} = enum
    valueA, valueB, valueC, valueD, amb
  
  OtherEnum {.pure.} = enum
    valueX, valueY, valueZ, amb


echo valueA # MyEnum.valueA
echo amb    # Error: MyEnum.amb であるか OtherEnum.amb であるか明確でない
echo MyEnum.amb # OK.
```
列挙型でビットフィールドを実装するには[ビットフィールド](#ビットフィールドBit-fields)を参照してください

### 文字列型(String type)
全ての文字列リテラルは`string`型である。
Nimのstringは文字のsequenceと似ています。
ただし、Nimのstringはゼロ終端でlengthフィールドを持ちます。
組み込みプロシージャの`len`を使用して、長さを取得できます。
lengthは終端のゼロをカウントしません。

文字列が最初に`cstring`型に変換されない限り、終端のゼロにアクセスできません。
終端のゼロは、この変換がO(1)で割り当てなしで実行できることを保証します。

文字列の代入演算子は、常に文字列をコピーします。`&`演算子は文字列を連結します。

ほとんどのネイティブなNimの型は、特別な`$`演算子で文字列に変換できます。
例えば、`echo`プロシージャーを呼び出すと、パラメーターに組み込みのstringfy演算が呼び出されます。

```nim
echo 3 # calls `$` for `int`
```
ユーザーが特別なオブジェクトを作成するたびに、このプロシージャの実装は`string`の表現を提供します。

```nim
type
  Person = object
    name: string
    age: int

proc `$`(p: Person): string = # `$` は常に string を返します
  result = p.name & " is " &
          $p.age & # 整数を文字列に変換するために
                   # p.age の前に `$` が必要です。
          " years old."
```
`$p.name`とも書けますが、この`$`演算子は何もしません。
`echo`プロシージャーのような、`int`から`string`への自動変換に頼れないことに注意して下さい。

文字列は辞書式順序で比較されます。
すべての比較演算子が利用可能です。
文字列は配列のようにインデックスを付けることができます（下限は0です）。
配列とは異なり、caseステートメントで使用できます。

```nim
case paramStr(i)
of "-v": incl(options, optVerbose)
of "-h", "-?": incl(options, optHelp)
else: write(stdout, "invalid command line option!\n")
```
規約により、すべての文字列はUTF-8文字列ですが、これは強制されません。
たとえば、バイナリファイルから文字列を読み取る場合、それらは単なるバイトシーケンスです。
インデックス演算`s[i]`は、i番目のunicharではなく、`s`のi番目のcharを意味します。
unicodeモジュールのイテレータ`rune`は、すべてのUnicode文字のイテレーションに使用できます。

### cstring型(cstring type)
`compatible string`の意味を持つ`cstring`型はコンパイルのバックエンドのための文字列のネイティブ表現です。
Cバックエンドの場合、`cstring`型は、Ansi Cの`char *`型と互換性のあるゼロ終端char配列へのポインタを表します。
その主な目的は、Cとの簡単なインターフェイスにあります。
インデックス演算`s[i]`は`s`のi番目の文字を意味します。
ただし、`cstring`は境界チェックされないため、インデックス演算は安全ではありません。

Nimの`string`は便宜上、暗黙的に`cstring`に変換可能です。Nim`string`がCスタイルの可変長プロシージャに渡されると、暗黙的に`cstring`に変換されます。

```nim
proc printf(formatstr: cstring) {.importc: "printf", varargs,
                                  header: "<stdio.h>".}

printf("This works %s", "as expected")
```
変換は暗黙的ですが、安全ではありません。
ガベージコレクターは`cstring`をルートとは見なさないため、基になるメモリを回収する場合があります。
しかし実際には、GCはスタックルートを保守的に考慮するため、これはほとんど起こりません。
組み込みプロシージャ`GC_ref`および`GC_unref`を使用して、まれに文字列データが機能しない場合に備えて文字列データを保持できます。

`$`プロシージャーはcstringに対してstringを返すように定義されています。
従って、cstringからstringを得るには以下のようにします。

```nim
var str: string = "Hello!"
var cstr: cstring = str
var newstr: string = $cstr
```

### 構造型(Structured types)
構造化型の変数は、同時に複数の値を保持できます。
構造化型は、無制限のレベルにネストできます。
配列、シーケンス、タプル、オブジェクト、およびセットは、構造型に属します。

### 配列とシーケンス型(Array and sequence types)
配列は各要素が同じ型である同質的な型です。
配列は常に定数式として指定された固定長を持ちます（オープン配列を除く）。
これらは、任意の順序型でインデックス付けできます。
0から`len(A)-1`の整数でイデックス付けられたパラメータ`A`はオープン配列とすることができます。
配列式は、配列コンストラクター`[]`によって構築できます。
この配列式の要素の型は、最初の要素の型から推測されます。
この配列式の他の全ての要素は最初の要素の型に暗黙的に変換可能である必要があります。

シーケンスは配列と似ていますが、動的な長さを持ち、実行時に変更される場合があります（文字列など）。
シーケンスは拡張可能な配列として実装され、アイテムが追加されるとメモリの一部を割り当てます。
シーケンス`S`は常に0から`len(S)-1`の整数でインデックス付けされ、その境界がチェックされます。
シーケンスは、配列コンストラクター`[]`と配列からシーケンスへの演算子`@`を組み合わせて構築できます。
シーケンスにメモリを割り当てるもう一つの方法は、組み込みプロシージャー`newSeq`を呼び出すことです。

シーケンスはオープン配列型のパラメータに渡すことができます。

例:

```nim
type
  IntArray = array[0..5, int] # 配列は 0..5 でインデックス付けられる
  IntSeq = seq[int] # 整数型のシーケンス
var
  x: IntArray
  y: IntSeq
x = [1, 2, 3, 4, 5, 6]  # [] は配列コンストラクタ
y = @[1, 2, 3, 4, 5, 6] # @ は配列をシーケンスに変換します

let z = [1.0, 2, 3, 4] # 配列 z の型は array[0..3, float]
```
配列またはシーケンスの下限は組み込みプロシージャー`low()`により、上限は`high()`により、長さは`len()`により得られます。
シーケンスまたはオープン配列の`low()`は常に0を返します。
`add()`プロシージャーまたは`&`演算子を使用してシーケンスに要素を追加し、`pop()`プロシージャーを使用してシーケンスの最後の要素を削除（および取得）できます。

配列コンストラクターは、可読性のために明示的なインデックスを持つことができます。

```nim
type
  Values = enum
    valA, valB, valC

const
  lookupTable = [
    valA: "A",
    valB: "B",
    valC: "C"
  ]
```

インデックスが省略されている場合、`succ(lastIndex)`がインデックス値として使用されます。

```nim
type
  Values = enum
    valA, valB, valC, valD, valE

const
  lookupTable = [
    valA: "A",
    "B",
    valC: "C",
    "D", "e"
  ]
```

### オープン配列(Open arrays)
多くの場合、固定サイズの配列は柔軟性に欠けます。
プロシージャは、異なるサイズの配列を処理できる必要があります。
パラメータのみに用いることができるオープン配列型はこれができます。
オープン配列は常に0から始まる`int`でインデックス付けられます。
互換性のある基本型を持つ配列は、オープン配列パラメーターに渡すことができ、インデックスの型は関係ありません。
配列に加えて、シーケンスをオープン配列パラメーターに渡すこともできます。

オープン配列型をネストすることはできません。
多次元のオープン配列はほとんど必要ではなく、効率的に実行できないため、サポートされていません。

```nim
proc testOpenArray(x: openArray[int]) = echo repr(x)

testOpenArray([1,2,3])  # array[]
testOpenArray(@[1,2,3]) # seq[]
```

### 可変長引数(Varargs)
`varargs` パラメータはプロシージャーに可変数の引数を渡すことができます。
コンパイラは引数リストを暗黙的に配列に変換します。

```nim
proc myWriteln(f: File, a: varargs[string]) =
  for s in items(a):
    write(f, s)
  write(f, "\n")

myWriteln(stdout, "abc", "def", "xyz")
# is transformed to:
myWriteln(stdout, ["abc", "def", "xyz"])
```
この変換は、varargsパラメーターがプロシージャヘッダーの最後のパラメーターである場合にのみ行われます。
下記のコンテキストで型変換を実行することもできます。

```nim
proc myWriteln(f: File, a: varargs[string, `$`]) =
  for s in items(a):
    write(f, s)
  write(f, "\n")

myWriteln(stdout, 123, "abc", 4.0)
# is transformed to:
myWriteln(stdout, [$123, $"def", $4.0])
```
この例では、パラメーター`a`に渡される引数に`$`が適用されます(`$`は文字列には適用されません)。

`varargs`パラメーターに渡された明示的な配列コンストラクターは、別の暗黙的な配列構築にラップされないことに注意してください。

```nim
proc takeV[T](a: varargs[T]) = discard

takeV([123, 2, 1]) # Tの型は intでなく、intの配列です
```

`varargs[typed]`は特別に扱われます。
これは、任意の型の引数の変数リストに一致しますが、常に暗黙的な配列を構築します。
これは、組み込みプロシージャー`echo`が期待される動作を行うために必要です。

```nim
proc echo*(x: varargs[typed, `$`]) {...}

echo @[1, 2, 3]
# prints "@[1, 2, 3]" and not "123"
```

### 未チェック配列(Unchecked arrays)
`UncheckedArray[T]`型はその境界をチェックしない特別な種類の配列です。
これは、カスタマイズされた柔軟なサイズの配列を実装するのに役立ちます。
さらに、未チェックの配列は、未定サイズのC配列に変換されます。

```nim
type
  MySeq = object
    len, cap: int
    data: UncheckedArray[int]
```
上記コードはおおよそ次のCコードを生成します。

```nim
typedef struct {
  NI len;
  NI cap;
  NI data[];
} MySeq;
```
未チェックの配列の基本型には、GCされたメモリが含まれていない可能性がありますが、現在はチェックされていません。

将来の方向性：未チェックの配列でGCされたメモリを許可し、GCが配列の実行時サイズを決定する方法についての明示的な注釈が必要です。

### タプルとオブジェクト型(Tuples and object types)
タプルまたはオブジェクト型の変数は、異種のストレージコンテナです。
タプルまたはオブジェクトは、型のさまざまな名前付きフィールドを定義します。
タプルは、フィールドの順序も定義します。
タプルは、オーバーヘッドがなく、抽象化の可能性がほとんどない異種ストレージ型を対象としています。
コンストラクター`()`を使用してタプルを作成できます。
コンストラクター内のフィールドの順序は、タプルの定義の順序と一致する必要があります。
同じ型の同じフィールドを同じ順序で指定する場合、異なるタプル型は同値です。
フィールドの名前も同一でなければなりません。

タプルの割り当て演算子は、各コンポーネントをコピーします。
オブジェクトのデフォルトの割り当て演算子は、各コンポーネントをコピーします。
代入演算子のオーバーロードについては、type-bound-operations-operatorで説明しています。

```nim
type
  Person = tuple[name: string, age: int] # type representing a person:
                                         # a person consists of a name
                                         # and an age
var
  person: Person
person = (name: "Peter", age: 30)
# the same, but less readable:
person = ("Peter", 30)
```

括弧と末尾のコンマを使用して、名前のないフィールドが1つあるタプルを作成できます。

```nim
proc echoUnaryTuple(a: (int,)) =
  echo a[0]

echoUnaryTuple (1,)
```
実際、すべてのタプルの構築には末尾のコンマが許可されます。

実装は、最高のアクセスパフォーマンスのためにフィールドを調整します。
アライメントは、Cコンパイラが行う方法と互換性があります

`object`宣言との一貫性を保つために、`type`セクションのタプルは`[]`の代わりにインデントで定義することもできます。

```nim
type
  Person = tuple   # type representing a person
    name: string   # a person consists of a name
    age: natural   # and an age
```

オブジェクトは、タプルにはない多くの機能を提供します。
オブジェクトは、継承と情報の隠蔽を提供します。
オブジェクトは実行時に型にアクセスできるため、`of`演算子を使用してオブジェクトの型を判別できます。
`of`演算子はJavaの`instanceof`に似ています。

```nim
type
  Person = object of RootObj
    name*: string   # *は`name`が他のモジュールからアクセスできることを意味します。
    age: int        # *がないため、フィールドは隠蔽されます
  
  Student = ref object of Person # a student is a person
    id: int                      # with an id field

var
  student: Student
  person: Person
assert(student of Student) # is true
assert(student of Person) # also true
```
定義モジュールの外部から見えるべきオブジェクトフィールドは、`*`でマークする必要があります。
タプルとは対照的に、異なるオブジェクト型が同値になることはありません。
継承元を持たないオブジェクトは暗黙的にfinalであるため、非表示型フィールドはありません。
`inheritable`プラグマを使用して、`system.RootObj`とは別に新しいオブジェクトルートを導入できます。

### オブジェクト構築(Object construction)
オブジェクトは、構文`T(fieldA: valueA, fieldB: valueB, ...)`を持つオブジェクト構築式を使用して作成することもできます。
ここで、`T`はオブジェクト型または`ref`オブジェクト型です。

```nim
var student = Student(name: "Anton", age: 5, id: 3)
```
タプルとは異なり、オブジェクトには値とともにフィールド名が必要です。
`ref`オブジェクトタイプの場合、`system.new`が暗黙的に呼び出されます。

### オブジェクトバリアント(Object variants)
多くの場合、単純なバリアント型が必要な特定の状況では、オブジェクト階層が過剰になります。
オブジェクトバリアントは、実行時の型の柔軟性に使用される列挙型によって識別されるタグ付き共用体であり、他の言語に見られる和型と代数データ型(ADTs)の概念を反映しています。

例

```nim
# これは、抽象構文ツリーをNimのでモデル化する方法の例である
type
  NodeKind = enum  # the different node types
    nkInt,          # a leaf with an integer value
    nkFloat,        # a leaf with a float value
    nkString,       # a leaf with a string value
    nkAdd,          # an addition
    nkSub,          # a subtraction
    nkIf            # an if statement
  Node = ref NodeObj
  NodeObj = object
    case kind: NodeKind  # ``kind``フィールドは判別子です
    of nkInt: intVal: int
    of nkFloat: floatVal: float
    of nkString: strVal: string
    of nkAdd, nkSub:
      leftOp, rightOp: Node
    of nkIf:
      condition, thenPart, elsePart: Node

# create a new case object:
var n = Node(kind: nkIf, condition: nil)
# ``nkIf``ブランチがアクティブであるためn.thenPartへのアクセスは有効:
n.thenPart = Node(kind: nkFloat, floatVal: 2.0)

# 次のステートメントはn.kindの値が適合せず、 `` nkString``ブランチがアクティブではないため、
# `FieldError`例外を発生させます:
n.strVal = ""

# 無効: アクティブなオブジェクトブランチを変更しようとしています:
n.kind = nkInt

var x = Node(kind: nkAdd, leftOp: Node(kind: nkInt, intVal: 4),
                          rightOp: Node(kind: nkInt, intVal: 2))
# 有効: アクティブなオブジェクトブランチを変更していません:
x.kind = nkSub
```
この例からわかるように、オブジェクト階層の利点は、異なるオブジェクトタイプ間でキャストする必要がないことです。
ただし、無効なオブジェクトフィールドにアクセスすると例外が発生します。

オブジェクト宣言の`case`の構文は、`case`ステートメントの構文に厳密に従います。
`case`セクションのブランチもインデントされる場合があります。

この例では、`kind`フィールドは判別子(discriminator)と呼ばれます。
安全のために、アドレスを取得することはできず、割り当ては制限されます。
新しい値によってアクティブなオブジェクトブランチが変更されてはなりません。
また、オブジェクトの構築中に特定のブランチのフィールドを指定する場合、対応する判別子の値を定数式として指定する必要があります。

アクティブなオブジェクトブランチを変更する代わりに、メモリ内の古いオブジェクトを完全に新しいオブジェクトに置き換えます。

```nim
var x = Node(kind: nkAdd, leftOp: Node(kind: nkInt, intVal: 4),
                          rightOp: Node(kind: nkInt, intVal: 2))
# change the node's contents:
x[] = NodeObj(kind: nkString, strVal: "abc")
```

バージョン0.20 以降では、`system.reset`を使用してオブジェクトブランチの変更をサポートすることはできません。
これは完全なメモリセーフではないためです。

特別なルールとして、判別子の種類は、`case`ステートメントを使用して制限することもできます。
`case`ステートメントブランチの判別子変数の可能な値が、選択したオブジェクトブランチの判別子値のサブセットである場合、初期化は有効と見なされます。
この解析は、順序型の不変の判別子に対してのみ機能し、`elif`分岐は無視します。
`range`型の判別子値の場合、コンパイラは、判別子値の可能な値の範囲全体が選択されたオブジェクトブランチに対して有効かどうかをチェックします。

例

```nim
let unknownKind = nkSub

# 無効: kindフィールドが静的に既知でないため安全でない初期化:
var y = Node(kind: unknownKind, strVal: "y")

var z = Node()
case unknownKind
of nkAdd, nkSub:
  # 有効: この分岐で可能な値は nkAdd/nkSub オブジェクトブランチのサブセットです:
  z = Node(kind: unknownKind, leftOp: Node(), rightOp: Node())
else:
  echo "ignoring: ", unknownKind

# これも有効, unknownKindBounded はnkAddまたはnkSubの値のみを含めることができるため
let unknownKindBounded = range[nkAdd..nkSub](unknownKind)
z = Node(kind: unknownKindBounded, leftOp: Node(), rightOp: Node())
```

### セット型(Set type)
セット型は、集合の数学的な概念をモデル化します。
セットのベース型は、特定のサイズの順序型のみになります。

- `int8`-`int16`
- `uint8`/`byte`-`unit16`
- `char`
- `enum`

またはその同等の型。
符号付き整数の場合、セットの基本型は`0..MaxSetElements-1`の範囲内にあると定義されます。ここで、`MaxSetElements`は常に2^16です。

その理由は、セットが高性能ビットベクトルとして実装されているためです。
より大きな型でセットを宣言しようとすると、エラーが発生します。

```nim
var s: set[int64] # Error: set is too large
```

セットは、セットコンストラクターを使用して構築できます。`{}`は空のセットです。
空のセットは、具体的なセットタイプと互換性のあるタイプです。
コンストラクターを使用して、要素（および要素の範囲）を含めることもできます。

```nim
type
  CharSet = set[char]
var
  x: CharSet
x = {'a'..'z', '0'..'9'} # 'a'から'z'の文字と数字の'0'to'9'を含むセットを構築します
```

以下の演算がセットでサポートされています。

|演算|意味|
|:---|:---|
|`A + B`|2つのセットの和|
|`A * B`|2つのセットの積|
|`A - B`|2つのセットの差(Bの要素を含まないA)|
|`A == B`|セットの等価性|
|`A <= B`|サブセット関係(AはBのサブセットまたは等価)|
|`A < B`|サブセット関係(AはBのサブセットであり等価でない)|
|`e in A`|セットメンバーシップ(Aは要素eを含む)|
|`e notin A`|Aは要素eを含まない|
|`contains(A, e)`|Aは要素eを含む|
|`card(A)`|Aの基数(Aの要素の数)|
|`incl(A, elem)`|`A = A + {elem}`と同じ|
|`excl(A, elem)`|`A = A - {elem}`と同じ|

### ビットフィールド(Bit fields)
セットは、プロシージャのフラグのタイプを定義するためによく使用されます。
これは、論理和`or`をとる必要のある整数定数を定義するよりも、クリーンで（かつ型安全な）解決法です。

列挙型、セット、およびキャストは、次のように一緒に使用できます。

```nim
type
  MyFlag* {.size: sizeof(cint).} = enum
    A
    B
    C
    D
  MyFlags = set[MyFlag]

proc toNum(f: MyFlags): int = cast[cint](f)
proc toFlags(v: int): MyFlags = cast[MyFlags](v)

assert toNum({}) == 0
assert toNum({A}) == 1
assert toNum({D}) == 8
assert toNum({A, C}) == 5
assert toFlags(0) == {}
assert toFlags(7) == {A, B, C}
```
セットが列挙値を2の累乗に変換する方法に注意してください。

Cで列挙型とセットを使用する場合は、厳密なcintを使用します。

Cとの相互運用性については、[ビットサイズプラグマ](#ビットサイズプラグマBitsize-pragma)も参照してください。

### 参照とポインタ型
参照（他のプログラミング言語のポインターと同様）は、多対1の関係を導入する方法です。
これは、異なる参照がメモリ内の同じ場所を指し、変更できることを意味します（エイリアシングとも呼ばれます）。

Nimは、トレースされた参照とトレースされていない参照を区別します。
トレースされていない参照は、ポインターとも呼ばれます。
トレースされた参照は、ガベージコレクションされるヒープのオブジェクトを指し、トレースされていない参照は、手動で割り当てられたオブジェクトまたはメモリ内の別のオブジェクトを指します。
したがって、トレースされていない参照は安全ではありません。ただし、特定の低レベルの操作（ハードウェアへのアクセス）では、トレースされていない参照は避けられません。

トレースされた参照はrefキーワードで宣言され、トレースされていない参照はptrキーワードで宣言されます。
一般に、`ptr T`は暗黙的にポインター型に変換できます。

空の添字[]表記を使用して参照を間接参照できます。
`addr`プロシージャはアイテムのアドレスを返します。
アドレスは常にトレースされていない参照です。
したがって、`addr`の使用は安全でない機能です。

`.`(タプル/オブジェクトフィールドアクセス演算子)および[] (配列/文字列/シーケンスインデックス演算子)演算子は、参照型の暗黙的な逆参照操作を実行します。

```nim
type
  Node = ref NodeObj
  NodeObj = object
    le, ri: Node
    data: int

var
  n: Node
new(n)
n.data = 9
# n[].dataと書く必要はありません; 実際 n[].data は非常に不推奨です!
```

ルーチン呼び出しの最初の引数に対しても、自動参照解除が実行されます。
ただし、現在この機能は`{.experimental： "implicitDeref".}`を介してのみ有効にする必要があります。

```nim
{.experimental: "implicitDeref".}

proc depth(x: NodeObj): int = ...

var
  n: Node
new(n)
echo n.depth
# n[].depth と書く必要もありません
```

構造型チェックを簡素化するために、再帰タプルは無効です。

```nim
# 無効な再帰
type MyTuple = tuple[a: ref MyTuple]
```

同様に、`T = ref T`は無効な型です。

構文の拡張として、`ref`オブジェクトまたは`ptr`オブジェクト表記法を介して型セクションで宣言された場合、`object`型は匿名になります。
この機能は、オブジェクトが参照セマンティクスのみを取得する必要がある場合に役立ちます。

```nim
type
  Node = ref object
    le, ri: Node
    data: int
```

新しいトレース対象オブジェクトを割り当てるには、組み込みプロシージャ`new`を使用する必要があります。
トレースされていないメモリを処理するには、プロシージャ`alloc`、`dealloc`および`realloc`を使用できます。
システムモジュールのドキュメントには、詳細情報が含まれています。

Nil  
参照が何も指していない場合、値は`nil`です。
`nil`は、すべての`ref`および`ptr`タイプのデフォルト値でもあります。
`nil`の逆参照は、回復不能な致命的なランタイムエラーです。
参照解除操作`p[]`は、`p`が`nil`でないことを意味します。
これは、実装によって悪用されて、次のようにコードを最適化できます。

```nim
p[].field = 3
if p != nil:
  # もしpがnilならば ``p[]`` によって既にクラッシュしているため、
  # ``p`` はここでは常にnilでないことが分かる
  action()
```
が

```nim
p[].field = 3
action()
```
となる。

注：これは、NULLポインターの逆参照に関するCの「未定義の動作」とは比較できません。

### GCされるメモリとptrの混合(Mixing GC'ed memory with ptr)
トレースされていないオブジェクトにトレースされた参照、文字列、シーケンスなどのトレースされたオブジェクトが含まれている場合は、特に注意する必要があります。
すべてを適切に解放するには、トレースされていないメモリを手動で解放する前に、組み込みプロシージャ`GCunref`を呼び出す必要があります

```nim
type
  Data = tuple[x, y: int, s: string]

# allocate memory for Data on the heap:
var d = cast[ptr Data](alloc0(sizeof(Data)))

# create a new string on the garbage collected heap:
d.s = "abc"

# tell the GC that the string is not needed anymore:
GCunref(d.s)

# free the memory:
dealloc(d)
```

`GCunref`呼び出しがなければ、`d.s`文字列に割り当てられたメモリは決して解放されません。
この例では、低レベルプログラミングの2つの重要な機能も示しています。
`sizeof`プロシージャーは、型または値のサイズをバイト単位で返します。
`cast`演算子は、型システムを回避することができます。
コンパイラは、`alloc0`呼び出し（型指定されていないポインタを返す）の結果を、`ptr Data`型を持つかのように処理することを強制されます。
キャストはやむを得ない場合にのみ行う必要があります。
型の安全性が損なわれ、バグが原因で不可解なクラッシュが発生する可能性があります。

注：この例は、メモリがゼロに初期化されるためにのみ機能します（`alloc`の代わりに`alloc0`を用いることで行われます）。
したがって、`d.s`は文字列割り当てが処理できるバイナリゼロに初期化されます。
ガベージコレクションされたデータをアンマネージメモリと混合する場合、このような低レベルの詳細を知る必要があります。

### Not nil annotation
nilが有効な値であるすべての型に`not nil`注釈(annotation)を付けて、nilを有効な値から除外できます。
(訳者注：下記コードをWandboxのNim1.0.2で試そうとした所、`{.experimental: "notnil".}`プラグマを要求されたため、要検証)

```nim
type
  PObject = ref TObj not nil
  TProc = (proc (x, y: int)) not nil

proc p(x: PObject) =
  echo "not nil"

# compiler catches this:
p(nil)

# and also this:
var x: PObject
p(x)
```
コンパイラは、すべてのコードパスが、nilできないポインターを含む変数を初期化することを保証します。
この解析の詳細は、まだここで指定する必要があります。

### プロシージャー型(Procedural type)
プロシージャー型は、内部的にはプロシージャーへのポインタです。
`nil`は、プロシージャー型の変数に許可される値です。
Nimはプロシージャー型を使用して、関数型プログラミング手法を実現します

例

```nim
proc printItem(x: int) = ...

proc forEach(c: proc (x: int) {.cdecl.}) =
  ...

forEach(printItem)  # 呼び出し規約が異なるため、これはコンパイルされません
```

```nim
type
  OnMouseMove = proc (x, y: int) {.closure.}

proc onMouseMove(mouseX, mouseY: int) =
  # デフォルトの呼び出し規約があります
  echo "x: ", mouseX, " y: ", mouseY

proc setOnMouseMove(mouseMoveEvent: OnMouseMove) = discard

# ok, 'onMouseMove'にはデフォルトの呼び出し規約があり、これは'closure'と互換性があります
setOnMouseMove(onMouseMove)
```

プロシージャー型の微妙な問題は、プロシージャーの呼び出し規則が型の互換性に影響することです。
プロシージャー型は、同じ呼び出し規則を持っている場合にのみ互換性があります。 
特別な拡張として、呼び出し規約`nimcall`のプロシージャを、呼び出し規約`closure`のprocを予期するパラメーターに渡すことができます。

Nimは次の呼び出し規約をサポートしています。

- nimcall
Nim procに使用されるデフォルトの規則です。
`fastcall`と同じですが、Cは`fastcall`をサポートするCコンパイラのみが対象です。

- closure
プラグマ注釈のない手続き型のデフォルトの呼び出し規則です。
プロシージャに隠された暗黙的なパラメータ（環境）があることを示します。
呼び出し規約の`closure`を持つProc変数は、2つのマシンワードを使用します。
1つはprocポインター用で、もう1つは暗黙的に渡された環境へのポインター用です。

- stdcall
これは、Microsoftが指定したstdcall規則です。
生成されたCプロシージャは、`__stdcall`キーワードで宣言されます。

- cdecl
cdecl規則は、プロシージャがCコンパイラと同じ規則を使用することを意味します。
Windowsでは、生成されたCプロシージャは`__cdecl`キーワードで宣言されます。

- safecall
これは、Microsoftが指定したsafecall規則です。生成されたCプロシージャは、`__safecall`キーワードで宣言されます。
セーフという言葉は、すべてのハードウェアレジスタがハードウェアスタックにプッシュされるという事実を指します。

- inline
インライン規則は、呼び出し元がプロシージャを呼び出すのではなく、コードを直接インライン化することを意味します。
Nimはインライン化せず、これをCコンパイラーに任せることに注意してください。`__inline`プロシージャを生成します。
これはコンパイラのヒントにすぎません。完全に無視されたり、`inline`としてマークされていないプロシージャをインライン化したりする場合があります。

- fastcall
fastcallは、Cコンパイラごとに異なることを意味を持ちます。Cの`__fastcall`が意味するものは何でも取得できます。

- syscall
syscallの規則は、Cの`__syscall`と同じです。割り込みに使用されます。

- noconv
生成されたCコードには明示的な呼び出し規約はないため、Cコンパイラのデフォルトの呼び出し規約が使用されます。
これが必要なのは、Nimのプロシージャに対するデフォルトの呼び出し規則が、速度を向上させるためのfastcallであるためです。

### distinct型(Distinct type)
distinct型は、その基本型と互換性のない基本タイプから派生した新しいタイプです。
distinct型とその基本型の間にはサブタイプの関係がないというのが、distinct型の本質的な性質です。

#### 通貨のモデル化(Modelling currencies)
例えば、distinct型を使用して、数値ベースタイプを使用して異なる物理ユニットをモデル化できます。次の例では、通貨をモデル化します。

通貨の計算に異なる通貨を混在させないでください。distinct型は、異なる通貨をモデル化するのに最適なツールです。

```nim
type
  Dollar = distinct int
  Euro = distinct int

var
  d: Dollar
  e: Euro

echo d + 12
# エラー: 単位なしユニットと``Dollar``は加算できない
```
残念ながら、`d + 12.Dollar`も使用できません。
これは、`+`が`int`（とりわけ）に対して定義されているがドルに対しては定義されていないためです。
したがって、ドルに対して`+`を定義する必要があります。

```nim
proc `+` (x, y: Dollar): Dollar =
  result = Dollar(int(x) + int(y))
```

ドルにドルを掛けることは意味がありませんが、単位のない数字を掛けることは意味があります。除算でも同じことが言えます。

```nim
proc `*` (x: Dollar, y: int): Dollar =
  result = Dollar(int(x) * y)

proc `*` (x: int, y: Dollar): Dollar =
  result = Dollar(x * int(y))

proc `div` ...
```

これはすぐに嫌になります。
実装は些末であり、コンパイラは後で最適化するためだけにこのコードをすべて生成すべきではありません-ドルのすべて`+`はintの`+`と同じバイナリコードを生成する必要があります。
borrowプラグマは、この問題を解決するように設計されています。 原則として、上記の簡単な実装を生成します。

```nim
proc `*` (x: Dollar, y: int): Dollar {.borrow.}
proc `*` (x: int, y: Dollar): Dollar {.borrow.}
proc `div` (x: Dollar, y: int): Dollar {.borrow.}
```
borrowプラグマにより、コンパイラは、distinct型の基本型を処理するprocと同じ実装を使用するため、コードは生成されません。

しかし、このすべての定型コードそユーロ通貨に対して繰り返す必要があるようです。これは[テンプレート](#テンプレートTemplates)で解決できます。

```nim
template additive(typ: typedesc) =
  proc `+` *(x, y: typ): typ {.borrow.}
  proc `-` *(x, y: typ): typ {.borrow.}
  
  # 単項演算子:
  proc `+` *(x: typ): typ {.borrow.}
  proc `-` *(x: typ): typ {.borrow.}

template multiplicative(typ, base: typedesc) =
  proc `*` *(x: typ, y: base): typ {.borrow.}
  proc `*` *(x: base, y: typ): typ {.borrow.}
  proc `div` *(x: typ, y: base): typ {.borrow.}
  proc `mod` *(x: typ, y: base): typ {.borrow.}

template comparable(typ: typedesc) =
  proc `<` * (x, y: typ): bool {.borrow.}
  proc `<=` * (x, y: typ): bool {.borrow.}
  proc `==` * (x, y: typ): bool {.borrow.}

template defineCurrency(typ, base: untyped) =
  type
    typ* = distinct base
  additive(typ)
  multiplicative(typ, base)
  comparable(typ)

defineCurrency(Dollar, int)
defineCurrency(Euro, int)
```

borrowプラグマを使用して、distinct型に注釈を付けて、特定の組み込み演算を引き継ぐこともできます。

```nim
type
  Foo = object
    a, b: int
    s: string
  
  Bar {.borrow: `.`.} = distinct Foo

var bb: ref Bar
new bb
# field access now valid
bb.a = 90
bb.s = "abc"
```

現在、この方法で借用できるのはドットアクセサーのみです。

#### SQLインジェクション攻撃の回避(Avoiding SQL injection attacks)
NimからSQLデータベースに渡されるSQLステートメントは、文字列としてモデル化される場合があります。
ただし、文字列テンプレートを使用して値を入力すると、有名なSQLインジェクション攻撃に対して脆弱になります。

```nim
import strutils

proc query(db: DbHandle, statement: string) = ...

var
  username: string

db.query("SELECT FROM users WHERE name = '$1'" % username)
# 恐ろしいセキュリティホールですが、コンパイラは気にしません!
```

これは、SQLを含む文字列とそうでない文字列を区別することで回避できます。
distinct型は、`string`と互換性のない新しいストリングタイプ`SQL`を導入する手段を提供します。

```nim
type
  SQL = distinct string

proc query(db: DbHandle, statement: SQL) = ...

var
  username: string

db.query("SELECT FROM users WHERE name = '$1'" % username)
# 静的エラー：`query`にはSQL文字列が必要です！
```

抽象型とその基本型との間のサブタイプの関係を暗示しないことは、抽象型の本質的な特性です。
文字列からSQLへの明示的な型変換が許可されます。

```nim
import strutils, sequtils

proc properQuote(s: string): SQL =
  # quotes a string properly for an SQL statement
  return SQL(s)

proc `%` (frmt: SQL, values: openarray[string]): SQL =
  # quote each argument:
  let v = values.mapIt(SQL, properQuote(it))
  # we need a temporary type for the type conversion :-(
  type StrSeq = seq[string]
  # call strutils.`%`:
  result = SQL(string(frmt) % StrSeq(v))

db.query("SELECT FROM users WHERE name = '$1'".SQL % [username])
```
これで、SQLインジェクション攻撃に対するコンパイル時のチェックができました。
`"".SQL`は`SQL("")`に変換されるため、見栄えの良いSQL文字列リテラルに新しい構文は必要ありません。
仮想SQL型は、[db_sqlite](https://nim-lang.org/docs/db_sqlite.html)などのモジュールの[TSqlQuery](https://nim-lang.org/docs/db_sqlite.html#TSqlQuery)型として実際にライブラリに存在します。

### オート型(Auto type)
`auto`型は戻り値の型やパラメータのために使用することができます。
戻り型の場合、コンパイラはルーチン本体から型を推測します。

```nim
proc returnsInt(): auto = 1984
```
現在、パラメータについては、暗黙的にジェネリックルーチンが作成されます。

```nim
proc foo(a, b: auto) = discard
```
これは次と同じです。

```nim
proc foo[T1, T2](a: T1, b: T2) = discard
```
ただし、後のバージョンの言語では、これを「ボディからパラメーターの型を推測する」ように変更する場合があります。
また、空の`discard`ステートメントからパラメーターの型を推測できないため、上記の`foo`は拒否されます。

## 型の関係性(Type relations)
以下のセクションでは、コンパイラーが行う型チェックを記述するために必要な、型に関するいくつかの関係を定義します。

### 型の等価性(Type Equality)
Nimは、ほとんどの型に対して構造的な型の等価性を使用します。
オブジェクト、列挙、およびdistinct型に対してのみ、名前の等価性が使用されます。
次の擬似コードのアルゴリズムは、型の等価性を決定します。

```nim
proc typeEqualsAux(a, b: PType,
                   s: var HashSet[(PType, PType)]): bool =
  if (a,b) in s: return true
  incl(s, (a,b))
  if a.kind == b.kind:
    case a.kind
    of int, intXX, float, floatXX, char, string, cstring, pointer,
        bool, nil, void:
      # leaf type: kinds identical; nothing more to check
      result = true
    of ref, ptr, var, set, seq, openarray:
      result = typeEqualsAux(a.baseType, b.baseType, s)
    of range:
      result = typeEqualsAux(a.baseType, b.baseType, s) and
        (a.rangeA == b.rangeA) and (a.rangeB == b.rangeB)
    of array:
      result = typeEqualsAux(a.baseType, b.baseType, s) and
               typeEqualsAux(a.indexType, b.indexType, s)
    of tuple:
      if a.tupleLen == b.tupleLen:
        for i in 0..a.tupleLen-1:
          if not typeEqualsAux(a[i], b[i], s): return false
        result = true
    of object, enum, distinct:
      result = a == b
    of proc:
      result = typeEqualsAux(a.parameterTuple, b.parameterTuple, s) and
               typeEqualsAux(a.resultType, b.resultType, s) and
               a.callingConvention == b.callingConvention

proc typeEquals(a, b: PType): bool =
  var s: HashSet[(PType, PType)] = {}
  result = typeEqualsAux(a, b, s)
```
型はサイクルを持つことができるグラフであるため、上記のアルゴリズムではこのケースを検出するために補助セット`s`が必要です。

### distictを除外した型の等価性
次のアルゴリズム（擬似コード）は、2つの型が`distinct`型に関係なく等しいかどうかを判断します。
簡潔にするために、補助セット`s`を使用したサイクルチェックは省略されています。

```nim
proc typeEqualsOrDistinct(a, b: PType): bool =
  if a.kind == b.kind:
    case a.kind
    of int, intXX, float, floatXX, char, string, cstring, pointer,
        bool, nil, void:
      # leaf type: kinds identical; nothing more to check
      result = true
    of ref, ptr, var, set, seq, openarray:
      result = typeEqualsOrDistinct(a.baseType, b.baseType)
    of range:
      result = typeEqualsOrDistinct(a.baseType, b.baseType) and
        (a.rangeA == b.rangeA) and (a.rangeB == b.rangeB)
    of array:
      result = typeEqualsOrDistinct(a.baseType, b.baseType) and
               typeEqualsOrDistinct(a.indexType, b.indexType)
    of tuple:
      if a.tupleLen == b.tupleLen:
        for i in 0..a.tupleLen-1:
          if not typeEqualsOrDistinct(a[i], b[i]): return false
        result = true
    of distinct:
      result = typeEqualsOrDistinct(a.baseType, b.baseType)
    of object, enum:
      result = a == b
    of proc:
      result = typeEqualsOrDistinct(a.parameterTuple, b.parameterTuple) and
               typeEqualsOrDistinct(a.resultType, b.resultType) and
               a.callingConvention == b.callingConvention
  elif a.kind == distinct:
    result = typeEqualsOrDistinct(a.baseType, b)
  elif b.kind == distinct:
    result = typeEqualsOrDistinct(a, b.baseType)
```

### サブタイプの関係性
オブジェクト`a`が`b`を継承する場合、`a`は`b`のサブタイプです。
このサブタイプの関係は、`var`,`ref`,`ptr`型に拡張されます。

```nim
proc isSubtype(a, b: PType): bool =
  if a.kind == b.kind:
    case a.kind
    of object:
      var aa = a.baseType
      while aa != nil and aa != b: aa = aa.baseType
      result = aa == b
    of var, ref, ptr:
      result = isSubtype(a.baseType, b.baseType)
```

### 変換可能な関係(Convertible relation)
次のアルゴリズムがtrueを返す場合、型`a`は暗黙的に型`b`に変換可能です。

```nim
proc isImplicitlyConvertible(a, b: PType): bool =
  if isSubtype(a, b) or isCovariant(a, b):
    return true
  if isIntLiteral(a):
    return b in {int8, int16, int32, int64, int, uint, uint8, uint16,
                 uint32, uint64, float32, float64}
  case a.kind
  of int:     result = b in {int32, int64}
  of int8:    result = b in {int16, int32, int64, int}
  of int16:   result = b in {int32, int64, int}
  of int32:   result = b in {int64, int}
  of uint:    result = b in {uint32, uint64}
  of uint8:   result = b in {uint16, uint32, uint64}
  of uint16:  result = b in {uint32, uint64}
  of uint32:  result = b in {uint64}
  of float32: result = b in {float64}
  of float64: result = b in {float32}
  of seq:
    result = b == openArray and typeEquals(a.baseType, b.baseType)
  of array:
    result = b == openArray and typeEquals(a.baseType, b.baseType)
    if a.baseType == char and a.indexType.rangeA == 0:
      result = b == cstring
  of cstring, ptr:
    result = b == pointer
  of string:
    result = b == cstring
```

暗黙的な変換は、Nimの`range`型コンストラクターに対しても実行されます。

`a0`,`b0`の型を`T`とし、`A = range[a0..b0]`を引数、`F`を仮引数の型とします。
`a0 >= low(F) and b0 <= high(F)`かつ`T`と`F`が共に符号あり整数か共に符号なし整数のとき、
`A`から`T`への暗黙の型変換が存在します。

次のアルゴリズムがtrueを返す場合、型`a`は型`b`に**明示的**に変換可能です。

```nim
proc isIntegralType(t: PType): bool =
  result = isOrdinal(t) or t.kind in {float, float32, float64}

proc isExplicitlyConvertible(a, b: PType): bool =
  result = false
  if isImplicitlyConvertible(a, b): return true
  if typeEqualsOrDistinct(a, b): return true
  if isIntegralType(a) and isIntegralType(b): return true
  if isSubtype(a, b) or isSubtype(b, a): return true
```

変換可能な関係は、ユーザー定義の型コンバーターによって緩和できます。

```nim
converter toInt(x: char): int = result = ord(x)

var
  x: int
  chr: char = 'a'

# ここで暗黙の変換マジックが発生します
x = chr
echo x # => 97
# 明示的な形式も使用できます
x = chr.toInt
echo x # => 97
```

`a`が左辺値で`typeEqualsOrDistinct(T,type(a))`が成り立つ場合、型変換`T(a)`は左辺値です。

### 割り当ての互換性(Assignment compatibility)
`a`が左辺値で`isImplicitlyConvertible(b.typ, a.typ)`が成り立つとき、式`b`は式`a`に割り当てることができます。

## オーバーロード解決
`p(args)`の呼び出しにおいて、`p`は最も一致するルーチンが選ばれます。
複数のルーチンが同等に一致するとき、セマンティック解析中にあいまいさが報告されます。

argsの全ての引数は一致する必要があります。
引数がどのように一致するかは、複数の異なるカテゴリがあります。
`f`を仮パラメータの型とし、`a`を引数の型とします。

- 完全一致：aとfが同じ型。
- リテラル一致：`a`は値`v`の整数リテラルであり、`f`は符号付きまたは符号なし整数型であり、かつ`v`は`f`の範囲にある。または、`a`は浮動小数点リテラルであり、`f`は浮動小数点型であり、かつ`v`は`f`の範囲にある。
- 総称一致：`f`が総称型で`a`が一致する。例えば、`a`が`int`型で`f`が総称（制約付き）パラメータ型である(`[T]`や`[T:int|char]`など)。
- subrangeまたはサブタイプ一致：`a`が`range[T]`でかつ`T`が`f`と完全一致。または、`a`が`f`のサブタイプ。
- Integral conversion一致：`a`は`f`に変換可能でかつ、`f`と`a`は整数型か浮動小数点型である。
- 変換一致：`a`はユーザ定義の`converter`を介して`f`に変換可能である。

これらの一致カテゴリには優先度があります。
完全一致はリテラル一致よりも優先され、総称一致などよりも優先されます。
次の`count(p,m)`では、ルーチン`p`の一致するカテゴリー`m`の一致数をカウントします。

```nim
for each matching category m in ["exact match", "literal match",
                                "generic match", "subtype match",
                                "integral match", "conversion match"]:
  if count(p, m) > count(q, m): return true
  elif count(p, m) == count(q, m):
    discard "continue with next category m"
  else:
    return false
return "ambiguous"
```

いくつかの例：

```nim
proc takesInt(x: int) = echo "int"
proc takesInt[T](x: T) = echo "T"
proc takesInt(x: int16) = echo "int16"

takesInt(4) # "int"
var x: int32
takesInt(x) # "T"
var y: int16
takesInt(y) # "int16"
var z: range[0..4] = 0
takesInt(z) # "T"
```

このアルゴリズムが「曖昧な」結果を返す場合、さらなる曖昧性解消が実行されます。
引数`a`がサブタイプ関係を介してパラメータータイプ`f`の`p`と`g`の両方に一致する場合、継承の深さが考慮されます。

```nim
type
  A = object of RootObj
  B = object of A
  C = object of B

proc p(obj: A) =
  echo "A"

proc p(obj: B) =
  echo "B"

var c = C()
# あいまいでない。Bを呼び出す
p(c)

proc pp(obj: A, obj2: B) = echo "A B"
proc pp(obj: B, obj2: A) = echo "B A"

# これはあいまいである:
pp(c, c)
```

同様に総称一致の場合、一致する中で最も特化した総称型が優先されます。

```nim
proc gen[T](x: ref ref T) = echo "ref ref T"
proc gen[T](x: ref T) = echo "ref T"
proc gen[T](x: T) = echo "T"

var ri: ref int
gen(ri) # "ref T"
```

### `var T`に基づくオーバーロード
仮パラメータ`f`が通常の型チェックに加えて`var T`型の場合、引数は左辺値であることがチェックされます。`var T`は、`T`だけであるよりもよく一致します。

```nim
proc sayHi(x: int): string =
  # matches a non-var int
  result = $x
proc sayHi(x: var int): string =
  # matches a var int
  result = $(x + 10)

proc sayHello(x: int) =
  var m = x # a mutable version of x
  echo sayHi(x) # matches the non-var version of sayHi
  echo sayHi(m) # matches the var version of sayHi

sayHello(3) # 3
            # 13
```

### untypedの遅延型解決
注：未解決の式とは、シンボルの検索や型チェックが実行されていない式です。
`immediate `として宣言されていないテンプレートとマクロはオーバーロード解決に関与するため、未解決の式をテンプレートまたはマクロに渡す方法が不可欠です。
これは、`untyped`メタタイプが達成することです。

```nim
template rem(x: untyped) = discard

rem unresolvedExpression(undeclaredIdentifier)
```

`untyped`型のパラメータは常に引数に一致します(引数に一致する限り)。

ただし、他のオーバーロードが引数の解決をトリガーする可能性があるため、注意が必要です。

```nim
template rem(x: untyped) = discard
proc rem[T](x: T) = discard

# undeclared identifier: 'unresolvedExpression'
rem unresolvedExpression(undeclaredIdentifier)
```

`untyped`と`varargs[untyped]`はこの意味で遅延な唯一なメタタイプであり、`typed`や`typedesc`などの他のメタタイプは遅延ではありません。

### Varargs matching
[可変長引数(Varargs)](#可変長引数Varargs)参照

## ステートメントと式(Statements and expressions)
Nimは一般的なステートメント/式のパラダイムを使用します。
ステートメントは式とは対照的に値を生成しません。
ただし、一部の式はステートメントです。

ステートメントは、単純なステートメントと複雑なステートメントに分けられます。
単純なステートメントは、割り当て、呼び出し、`return`ステートメントなどの他のステートメントを含むことができないステートメントです。
複雑なステートメントには、他のステートメントを含めることができます。
他の問題を回避するために、複雑なステートメントは常にインデントする必要があります。
詳細は[文法(grammar)](#文法Grammar)に記載されています。

### ステートメントリスト式(Statement list expression)
ステートメントは、(stmt1; stmt2; ...; ex)のような式コンテキストでも生成される可能性があります。
これはステートメントリスト式または`(;)`と呼ばれます。
(stmt1; stmt2; ...; ex)の型は`ex`の型です。
他のすべてのステートメントは`void`型でなければなりません(`discard`を使用して`void`型を生成できます)。
`(;)`は新しいスコープを導入しません。

### Discardステートメント(Discard statement)
例：

```nim
proc p(x, y: int): int =
  result = x + y

discard p(3, 4) # `p`の戻り値を破棄
```

`discard`ステートメントは式の副作用について評価を行い、結果の値を破棄します。

discardステートメントを使用せずにプロシージャの戻り値を無視すると、静的エラーになります。
呼び出されたproc/iteratorがdiscardableプラグマで宣言されている場合、戻り値は暗黙的に無視できます。

```nim
proc p(x, y: int): int {.discardable.} =
  result = x + y

p(3, 4) # now valid
```

空のdiscardステートメントは、多くの場合、nullステートメントとして使用されます。

```nim
proc classify(s: string) =
  case s[0]
  of SymChars, '_': echo "an identifier"
  of '0'..'9': echo "a number"
  else: discard
```

### Void context
ステートメントのリストでは、最後の式を除くすべての式が`void`型である必要があります。
この規則に加えて、組み込み`result`シンボルへの割り当てを行った場合、後続の式が全て`void`コンテキストである必要があります。

```nim
proc invalid*(): string =
  result = "foo"
  "invalid"  # Error: value of type 'string' has to be discarded
```

```nim
proc valid*(): string =
  let x = 317
  "valid"
```

### Var statement
Varステートメントは、新しいローカル変数とグローバル変数を宣言し、それらを初期化します。
変数のコンマ区切りリストを使用して、同じ型の変数を指定できます。

```nim
var
  a: int = 0
  x, y, z: int
```

初期化子が指定されている場合、型は省略できます。
変数は初期化式と同じ型になります。
初期化式がない場合、変数は常にデフォルト値で初期化されます。
デフォルト値は型によって異なり、バイナリでは常にゼロです。

|型|デフォルト値|
|:---|:---|
|any integer type|0|
|any float|0.0|
|char|'\0'|
|bool|false|
|ref or pointer type|nil|
|procedural type|nil|
|sequence|`@[]`|
|string|`""`|
|tuple[x: A, y: B, ...]|(default(A), default(B), ...) (オブジェクトに類似)|
|array[0..., T]|[default(T), ...]|
|range[T]|default(T); これは有効値の範囲外である可能性があります|
|T = enum|castT; これは無効な値である可能性があります|

noinitプラグマを使用すると、最適化のために暗黙的な初期化を回避できます。

```nim
var
  a {.noInit.}: array[0..1023, char]
```

procに`noinit`プラグマで注釈が付けられている場合、暗黙の`result`変数を参照します。

```nim
proc returnUndefinedValue: int {.noinit.} = discard
```

暗黙的な初期化は、requiresInit型プラグマによっても防ぐことができます。
コンパイラは、オブジェクトとそのすべてのフィールドの明示的な初期化を必要とします。
ただし、変数が初期化されたことを証明するために制御フロー分析を行い、構文プロパティに依存しません。

```nim
type
  MyObject = object {.requiresInit.}

proc p() =
  # the following is valid:
  var x: MyObject
  if someCondition():
    x = a()
  else:
    x = a()
  # use x
```

### Let statement
`let`文は、新しいローカルとグローバル宣言する単一代入変数を、それらに値をバインドします。
構文は、キーワード`var`がキーワード`let`に置き換えられることを除いて、varステートメントの構文と同じです。
変数は左辺値ではないため、`var`パラメータに渡すことも、アドレスを取得することもできません。
新しい値を割り当てることはできません。

let変数では、通常の変数と同じプラグマを使用できます。

### Tuple unpacking
`var`または`let`ステートメントでタプルのアンパックを実行できます。
特別な識別子`_`を使用して、タプルの一部を無視できます。

```nim
proc returnsTuple(): (int, int, int) = (4, 2, 3)

let (x, _, z) = returnsTuple()
```

### Const section
constセクションは、値が定数式である定数を宣言します。

```nim
import strutils
const
  roundPi = 3.1415
  constEval = contains("abc", 'b') # computed at compile time!
```

宣言すると、定数のシンボルを定数式として使用できます。

詳細については、[定数と定数式](#定数と定数式)を参照してください。

### Static statement/expression
静的ステートメント/式は、コンパイル時の実行を明示的に必要とします。
副作用のあるコードでさえ、静的ブロックはで許可されています。

```nim
static:
  echo "echo at compile time"
```

コンパイル時に実行できるNimコードには制限があります。
詳細については、[コンパイル時実行の制限](#コンパイル時実行の制限)を参照してください。
コンパイラがコンパイル時にブロックを実行できない場合、静的エラーです。

### If statement
例

```nim
var name = readLine(stdin)

if name == "Andreas":
  echo "What a nice name!"
elif name == "":
  echo "Don't you have a name?"
else:
  echo "Boring name..."
```

`if`ステートメントは、制御フローでブランチを作成する簡単な方法です。
キーワード`if`の後の式が評価され、trueの場合、`:`の後の対応するステートメントが実行されます。
そうでない場合、`elif`の後の式が評価され（`elif`ブランチがある場合）、trueの場合、`:`の後の対応するステートメントが実行されます。
これは最後の`elif`まで続きます。
すべての条件が失敗すると、`else`パートが実行されます。
`else`パートがない場合、実行は次のステートメントから続行されます。

`if`ステートメントでは、新しいスコープは`if/elif/else`キーワードの直後から始まり、対応するthenブロックの後に終わります。
次の例ではスコープは視覚化のために`{| |}`で囲まれています。

```nim
if {| (let m = input =~ re"(\w+)=\w+"; m.isMatch):
  echo "key ", m[0], " value ", m[1]  |}
elif {| (let m = input =~ re""; m.isMatch):
  echo "new m in this scope"  |}
else: {|
  echo "m not declared here"  |}
```

### Case statement
例：

```nim
case readline(stdin)
of "delete-everything", "restart-computer":
  echo "permission denied"
of "go-for-a-walk":     echo "please yourself"
else:                   echo "unknown command"

# ブランチのインデントも許可されます
# 'readline(stdin)'の後のコロン':'はオプショナルです
case readline(stdin):
  of "delete-everything", "restart-computer":
    echo "permission denied"
  of "go-for-a-walk":     echo "please yourself"
  else:                   echo "unknown command"
```

`case`の文は、`if`文に似ていますが、多分岐の選択を表します。
キーワード`case`の後の式が評価され、その値がslicelistにある場合、対応するステートメント（`of`キーワードの後）が実行されます。
値がslicelistにない場合、`else`パートが実行されます。
もし`else`パートがなく、`expr`が保持できるすべての可能な値がslicelistにあるわけではない場合、静的エラーが発生します。
これは、順序型の式にのみ当てはまります。
`expr`の「すべての可能な値」は、`expr`の型によって決まります。
静的エラーを抑制するには、空の`discard`ステートメントを持つ`else`パートを使用する必要があります。

非順序型の場合、可能な値をすべてリストすることはできないため、これらには常に`else`パートが必要です。

セマンティック解析中に`case`ステートメントの網羅性がチェックされるため、すべての`of`ブランチの値は定数式である必要があります。
また、この制限により、コンパイラはよりパフォーマンスの高いコードを生成できます。

特別なセマンティック拡張として、`case`ステートメントの`of`ブランチの式は、セットまたは配列コンストラクターで評価することもできます。
セットまたは配列は、その要素のリストに展開されます。

```nim
const
  SymChars: set[char] = {'a'..'z', 'A'..'Z', '\x80'..'\xFF'}

proc classify(s: string) =
  case s[0]
  of SymChars, '_': echo "an identifier"
  of '0'..'9': echo "a number"
  else: echo "other"

# is equivalent to:
proc classify(s: string) =
  case s[0]
  of 'a'..'z', 'A'..'Z', '\x80'..'\xFF', '_': echo "an identifier"
  of '0'..'9': echo "a number"
  else: echo "other"
```

### When statement
例：

```nim
when sizeof(int) == 2:
  echo "running on a 16 bit system!"
elif sizeof(int) == 4:
  echo "running on a 32 bit system!"
elif sizeof(int) == 8:
  echo "running on a 64 bit system!"
else:
  echo "cannot happen!"
```
`when`ステートメントは、いくつかの例外を除いて`if`ステートメントとほとんど同じです。

- 各条件（`expr`）は（`bool`型の）定数式でなければなりません。
- ステートメントは、新しいスコープを開きません。
- trueと評価された式に属するステートメントはコンパイラーによって変換され、他のステートメントはセマンティクスがチェックされません！ただし、各条件のセマンティクスがチェックされます。

`when`ステートメントは、条件付きコンパイル手法を有効にします。
特別な構文拡張として、`when`構文は`object`定義内でも使用できます。

### When nimvm statement
`nimvm`は特別なシンボルであり、コンパイル時と実行可能ファイルの実行パスを区別するための`when nimvm`ステートメントの式として使用できます。

例：

```nim
proc someProcThatMayRunInCompileTime(): bool =
  when nimvm:
    # This branch is taken at compile time.
    result = true
  else:
    # This branch is taken in the executable.
    result = false
const ctValue = someProcThatMayRunInCompileTime()
let rtValue = someProcThatMayRunInCompileTime()
assert(ctValue == true)
assert(rtValue == false)
```

`when nimvm`ステートメントは次の要件を満たしている必要があります。

- その式は常にnimvmでなければなりません。より複雑な式は許可されていません。
- `elif`ブランチを含めることはできません。
- `else`ブランチが含まれている必要があります。
- ブランチ内のコードは、`when nimvm`ステートメントに続くコードのセマンティクスに影響を与えてはなりません。たとえば、後続のコードで使用されるシンボルを定義してはなりません。

### Return statement
例：

```nim
return 40+2
```

`return`ステートメントは、現在のプロシージャの実行を終了します。
プロシージャでのみ許可されます。
`expr`がある場合、これは次の構文糖衣です

```nim
result = expr
return result
```

戻り値があるプロシージャーでの式のない`return`は`return result`の短縮表記です。
return変数は常にプロシージャーの戻り値です。
これはコンパイラによって自動的に宣言されます。
`result`は(バイナリ)ゼロに初期化されます。

```nim
proc returnZero(): int =
  # implicitly returns 0
```

### Yield statement
例：

```nim
yield (1, 2, 3)
```

イテレータでは、`return`ステートメントの代わりに`yield`ステートメントが使用されます。
イテレータでのみ有効です。イテレータを呼び出したforループの本体に実行が返されます。
Yieldは反復プロセスを終了しませんが、次の反復が開始されると、実行はイテレータに戻されます。
詳細については、イテレーターに関するセクション[Iterators and the for statement](#Iterators-and-the-for-statement)を参照してください。

### Block statement
例：

```nim
var found = false
block myblock:
  for i in 0..3:
    for j in 0..3:
      if a[j][i] == 7:
        found = true
        break myblock # leave the block, in this case both for-loops
echo found
```

`block`ステートメントは、ステートメントを（名前付き）ブロックにグループ化する手段です。
ブロック内では、`break`ステートメントを使用してすぐにブロックを離脱することができます。
ブレーク文は離脱するブロックを指定するために、ブロックの名前を含めることができます。

### Break statement
例：

```nim
break
```

`break`文は即座にブロックを離脱するために使います。
`symbol`が指定されている場合、それは離脱するブロックの名前です。
指定しない場合、最も内側のブロックを離脱します。

### While statement
例：

```nim
echo "Please tell me your password:"
var pw = readLine(stdin)
while pw != "12345":
  echo "Wrong password! Next try:"
  pw = readLine(stdin)
```

`while`ステートメントは、`expr`が`false`と評価されるまで実行されます。
無限ループはエラーではありません。
`while`ステートメントは暗黙的なブロックを開くため、`break`ステートメントで離脱することができます。

### Continue statement
`continue`ステートメントは、ループ構造の次の反復に移ります。
ループ内でのみ許可されます。
`continue`ステートメントは、ネストされたブロックの糖衣構文です。

```nim
while expr1:
  stmt1
  continue
  stmt2
```
は以下と同等です。

```nim
while expr1:
  block myBlockName:
    stmt1
    break myBlockName
    stmt2
```

### Assembler statement
Nimコードへのアセンブラコードの直接埋め込みは、アンセーフな`asm`ステートメントによってサポートされています。
Nim識別子を参照するアセンブラコード内の識別子は、ステートメントのプラグマで指定できる特殊文字で囲む必要があります。
デフォルトの特殊文字は'`'です。

```nim
{.push stackTrace:off.}
proc addInt(a, b: int): int =
  # a in eax, and b in edx
  asm """
      mov eax, `a`
      add eax, `b`
      jno theEnd
      call `raiseOverflow`
    theEnd:
  """
{.pop.}
```

GNUアセンブラーを使用する場合、引用符と改行が自動的に挿入されます。

```nim
proc addInt(a, b: int): int =
  asm """
    addl %%ecx, %%eax
    jno 1
    call `raiseOverflow`
    1:
    :"=a"(`result`)
    :"a"(`a`), "c"(`b`)
  """
```
は次の代わりになります。

```nim
proc addInt(a, b: int): int =
  asm """
    "addl %%ecx, %%eax\n"
    "jno 1\n"
    "call `raiseOverflow`\n"
    "1: \n"
    :"=a"(`result`)
    :"a"(`a`), "c"(`b`)
  """
```

### Using statement
usingステートメントは、同じパラメーター名と型が繰り返し使用されるモジュールで構文上の利便性を提供します。

```nim
proc foo(c: Context; n: Node) = ...
proc bar(c: Context; n: Node, counter: int) = ...
proc baz(c: Context; n: Node) = ...
```

名前`c`のパラメーターはデフォルトで`Context`型になり、`n`はデフォルトで`Node`などになるという規則についてコンパイラーに伝えることができます。

```nim
using
  c: Context
  n: Node
  counter: int

proc foo(c, n) = ...
proc bar(c, n, counter) = ...
proc baz(c, n) = ...

proc mixedMode(c, n; x, y: int) =
  # 'c' is inferred to be of the type 'Context'
  # 'n' is inferred to be of the type 'Node'
  # But 'x' and 'y' are of type 'int'.
```

`using`セクションは、`var`または`let`セクションと同じインデントベースのグループ化構文を使用します。

型指定されていないテンプレートパラメータはデフォルトで`system.untyped`型であるため、`template`には`using`は適用されないことに注意してください。

`using`宣言を使用する必要があるパラメーターと、明示的に入力されるパラメーターを混在させることは可能です。
それらの間にセミコロンが必要です。

### If expression
`if expression `はif文とほとんど同じですが、式です。

```nim
var y = if x > 8: 9 else: 10
```

if式の結果は常に値になるため、`else`のパートが必要です。`Elif`パートも使用できます。

### When expression
`if expression `と同じですが、whenステートメントに対応します。

### Case expression
`case expression`もまた、caseステートメントにとてもよく似ています。

```nim
var favoriteFood = case animal
  of "dog": "bones"
  of "cat": "mice"
  elif animal.endsWith"whale": "plankton"
  else:
    echo "I'm not sure what to serve, but everybody loves ice cream"
    "ice cream"
```

上記の例に見られるように、case式は副作用を引き起こす可能性もあります。
ブランチに複数のステートメントが指定されている場合、Nimは最後の式を結果値として使用します。

### Block expression
`block expression`は、ブロックステートメントに似ていますが、ブロックの下の最後の式を値として使用する式です。
ステートメントリスト式に似ていますが、ステートメントリスト式は新しいブロックスコープを開きません。

```nim
let a = block:
  var fib = @[0, 1]
  for i in 0..10:
    fib.add fib[^1] + fib[^2]
  fib
```

### Table constructor
テーブルコンストラクターは、配列コンストラクターの糖衣構文です。

```nim
{"key1": "value1", "key2", "key3": "value2"}

# is the same as:
[("key1", "value1"), ("key2", "value2"), ("key3", "value2")]
```

空のテーブルは`{:}`と書くことができ（空のセット`{}`とは対照的です）、これは空の配列コンストラクター`[]`として書く別の方法です。
テーブルをサポートするこの少し変わった方法には、多くの利点があります。

- (key,value)ペアの順序は保持されるため、たとえば`{key:val}.newOrderedTable`を使用すると、順序付けされた辞書を簡単にサポートできます。
- テーブルリテラルは`const`セクションに配置でき、コンパイラーは配列の場合と同じように実行可能ファイルのデータセクションに簡単に配置できます。また、生成されたデータセクションには最小限のメモリが必要です。
- すべてのテーブル実装は、構文的に同等に扱われます。
- 最小限の糖衣構文は別として、言語コアはテーブルについて知る必要はありません。

### 型変換(Type conversions)
構文的には、型変換はプロシージャコールに似ていますが、プロシージャ名はタイプ名に置き換えられます。
型の変換は、型を別の型に変換できないと例外が発生するという意味で常に安全です（静的に決定できない場合）。

Nimの型変換よりも通常のprocが好まれることがよくあります。
たとえば、慣例により$は`toString`演算子であり、`toFloat`と`toInt`を使用して浮動小数点から整数へ、またはその逆に変換できます。

型変換は、オーバーロードされたルーチンを明確にするためにも使用できます。

```nim
proc p(x: int) = echo "int"
proc p(x: string) = echo "string"

let procVar = (proc(x: string))(p)
procVar("a")
```

### 型キャスト(Type casts)
例：

```nim
cast[int](x)
```

型キャストは、式のビットパターンを別の型であるかのように解釈する粗雑なメカニズムです。
型キャストは低レベルのプログラミングにのみ必要であり、本質的にアンセーフです。

### アドレス演算子(The addr operator)
`addr`演算子は、左辺値のアドレスを返します。
ロケーションのタイプが`T`の場合、`addr`演算子の結果は`ptr T`型です。
アドレスは常にトレースされない参照です。
スタックにあるオブジェクトのアドレスを取得することは**安全ではありません**。
なぜなら、ポインターはスタック上のオブジェクトよりも長く存続し、存在しないオブジェクトを参照できるからです。
変数のアドレスを取得できますが、`let`ステートメントで宣言された変数では使用できません。

```nim
let t1 = "Hello"
var
  t2 = t1
  t3 : pointer = addr(t2)
echo repr(addr(t2))
# --> ref 0x7fff6b71b670 --> 0x10bb81050"Hello"
echo cast[ptr string](t3)[]
# --> Hello
# The following line doesn't compile:
echo repr(addr(t1))
# Error: expression has no address
```

### アンセーフアドレス演算子(The unsafeAddr operator)
`C`などの他のコンパイル言語との相互運用を容易にするため、`let`変数、パラメーター、または`for`ループ変数のアドレスを取得するには、`unsafeAddr`演算を使用できます。

```nim
let myArray = [1, 2, 3]
foreignProcThatTakesAnAddr(unsafeAddr myArray)
```

## プロシージャー(Procedures)
ほとんどのプログラミング言語でメソッドまたは関数を呼ばれるものは、Nimではプロシージャと呼ばれます。
プロシージャ宣言は、識別子、0個以上の仮パラメータ、戻り値の型、およびコードのブロックで構成されます。
仮パラメータは、カンマまたはセミコロンで区切られた識別子のリストとして宣言されます。
パラメーターには、`typename`によって型が与えられます。
型は、パラメーターリストの先頭、セミコロン区切り文字、または既に入力されたパラメーターのいずれかに到達するまで、その直前のすべてのパラメーターに適用されます。
セミコロンを使用して、タイプの分離と後続の識別子をより明確にすることができます。

```nim
# Using only commas
proc foo(a, b: int, c, d: bool): int

# Using semicolon for visual distinction
proc foo(a, b: int; c, d: bool): int

# Will fail: a is untyped since ';' stops type propagation.
proc foo(a; b: int; c, d: bool): int
```

パラメータは、呼び出し側が引数に値を提供しない場合に使用されるデフォルト値で宣言できます。

```nim
# b is optional with 47 as its default value
proc foo(a: int, b: int = 47): int
```

型修飾子`var`を使用してパラメーターをミュータブルとして宣言し、procがこれらの引数を変更できるようになります。

```nim
# "returning" a value to the caller through the 2nd argument
# Notice that the function uses no actual return value at all (ie void)
proc foo(inp: int, outp: var int) =
  outp = inp + 47
```

proc宣言に本体がない場合は、前方宣言です。
プロシージャが値を返す場合、プロシージャ本体は、戻り値を表す`result`という名前の暗黙的に宣言された変数にアクセスできます。
Procsはオーバーロードされる可能性があります。オーバーロード解決アルゴリズムは、引数に最適なprocを決定します。

```nim
proc toLower(c: char): char = # toLower for characters
  if c in {'A'..'Z'}:
    result = chr(ord(c) + (ord('a') - ord('A')))
  else:
    result = c

proc toLower(s: string): string = # toLower for strings
  result = newString(len(s))
  for i in 0..len(s) - 1:
    result[i] = toLower(s[i]) # calls toLower for characters; no recursion!
```

プロシージャの呼び出しは、さまざまな方法で実行できます。

```nim
proc callme(x, y: int, s: string = "", c: char, b: bool = false) = ...

# call with positional arguments      # parameter bindings:
callme(0, 1, "abc", '\t', true)       # (x=0, y=1, s="abc", c='\t', b=true)
# call with named and positional arguments:
callme(y=1, x=0, "abd", '\t')         # (x=0, y=1, s="abd", c='\t', b=false)
# call with named arguments (order is not relevant):
callme(c='\t', y=1, x=0)              # (x=0, y=1, s="", c='\t', b=false)
# call as a command statement: no () needed:
callme 0, 1, "abc", '\t'              # (x=0, y=1, s="abc", c='\t', b=false)
```

プロシージャは、それ自体を再帰的に呼び出すことができます。

演算子は、識別子として特別な演算子記号を使用したプロシージャです。

```nim
proc `$` (x: int): string =
  # converts an integer to a string; this is a prefix operator.
  result = intToStr(x)
```

1つのパラメーターを持つ演算子は前置演算子であり、2つのパラメーターを持つ演算子は中置演算子です。
（ただし、パーサーはこれらを式内の演算子の位置と区別します。）
後置演算子を宣言する方法はありません。
すべての後置演算子は組み込みであり、文法によって明示的に処理されます。

\`opr\`表記を使用して、通常のprocのように任意の演算子を呼び出すことができます。（したがって、演算子は3つ以上のパラメーターを持つことができます）

```nim
proc `*+` (a, b, c: int): int =
  # Multiply and add
  result = a * b + c

assert `*+`(3, 4, 6) == `+`(`*`(a, b), c)
```

### Export marker
宣言されたシンボルがアスタリスクでマークされている場合、現在のモジュールからエクスポートされます。

```nim
proc exportedEcho*(s: string) = echo s
proc `*`*(a: string; b: int): string =
  result = newStringOfCap(a.len * b)
  for i in 1..b: result.add a

var exportedVar*: int
const exportedConst* = 78
type
  ExportedType* = object
    exportedField*: int
```

### メソッド呼び出し構文(Method call syntax)
オブジェクト指向プログラミングのために、`method(obj,args)`の代わりに`obj.method(args)`を使用できます。
引数が残っていない場合は括弧を省略できます：`len(obj)`の代わりに`obj.len`。

このメソッド呼び出し構文はオブジェクトに限定されず、プロシージャの任意の型の最初の引数を提供するために使用できます。

```nim
echo "abc".len # is the same as echo len "abc"
echo "abc".toUpper()
echo {'a', 'b', 'c'}.card
stdout.writeLine("Hallo") # the same as writeLine(stdout, "Hallo")
```

メソッド呼び出し構文を見るもう1つの方法は、欠落している後置記法を提供することです。

メソッド呼び出しの構文は、明示的なジェネリックインスタンス化と競合します。
`x.p[T]`は常に`(x.p)[T]`として解析されるため、`p[T](x)`を`x.p[T]`と記述できません。

参照：[メソッド呼び出し構文の制限](#メソッド呼び出し構文の制限Limitations-of-the-method-call-syntax)

`[: ]`表記はこの問題を軽減するために設計されています。
`x.p[:T]`はパーサーによって`p[T](x)`と書き換えられ、`x.p[:T](y)`はパーサーによって`p[T](x, y)`と書き換えられます。
`[: ]`にはAST表現がないため、書き換えは解析ステップで直接実行されることに注意してください。

### プロパティ(Properties)
Nimはget-propertiesを必要としません。
メソッド呼び出し構文で呼び出される通常のget-proceduresは同じことを達成します。
ただし、値の設定は異なります。このためには、特別なセッター構文​​が必要です。

```nim
# Module asocket
type
  Socket* = ref object of RootObj
    host: int # cannot be accessed from the outside of the module

proc `host=`*(s: var Socket, value: int) {.inline.} =
  ## setter of hostAddr.
  ## This accesses the 'host' field and is not a recursive call to
  ## ``host=`` because the builtin dot access is preferred if it is
  ## avaliable:
  s.host = value

proc host*(s: Socket): int {.inline.} =
  ## getter of hostAddr
  ## This accesses the 'host' field and is not a recursive call to
  ## ``host`` because the builtin dot access is preferred if it is
  ## avaliable:
  s.host
````

```nim
# module B
import asocket
var s: Socket
new s
s.host = 34  # same as `host=`(s, 34)
```

`f=`(後ろに`=`がつく)として定義されたprocはセッターと呼ばれます。
セッターはバッククォート表記を介して明示的に呼び出すことができます。

```nim
proc `f=`(x: MyObject; value: string) =
  discard

`f=`(myObject, "value")
```

`x`の型に`f`という名前のフィールドがない場合または`f`がカレントモジュールから不可視の場合に限り、
パターン`x.f = value`で暗黙的に`f=`を呼び出すことができます。
これらの規則により、オブジェクトフィールドとアクセサに同じ名前を付けることができます。
モジュール内では`x.f`は常にフィールドアクセスとして解釈され、モジュール外ではアクセサproc呼び出しとして解釈されます。

### コマンド呼び出し構文(Command invocation syntax)
呼び出しが構文的にステートメントである場合、ルーチンは`()`なしで呼び出すことができます。
このコマンド呼び出し構文は式でも機能しますが、その後に続く引数は1つだけです。
この制限は、`echo f 1, f 2`が`echo(f(1, f(2)))`としてではなく`echo(f(1), f(2))`として解析されることを意味します。
この場合、メソッド呼び出し構文を使用してもう1つの引数を提供できます。

```nim
proc optarg(x: int, y: int = 0): int = x + y
proc singlearg(x: int): int = 20*x

echo optarg 1, " ", singlearg 2  # prints "1 40"

let fail = optarg 1, optarg 8   # Wrong. Too many arguments for a command call
let x = optarg(1, optarg 8)  # traditional procedure call with 2 arguments
let y = 1.optarg optarg 8    # same thing as above, w/o the parenthesis
assert x == y
```

また、コマンド呼び出し構文には、引数として複雑な式を含めることはできません。
例：[匿名プロシージャー](#匿名プロシージャーAnonymous-Procs),`if`,`case`,`try`。
引数なしの関数呼び出しでは、呼び出しと関数自体を最初のクラス値として区別するために()が必要です。

### クロージャー(Closures)
プロシージャは、モジュールの最上位レベルおよび他のスコープ内に表示できます。
この場合、プロシージャはネストされたプロシージャと呼ばれます。
ネストされたprocは、それを囲むスコープからローカル変数にアクセスできます。
そうすると、クロージャーになります。
キャプチャされた変数は、クロージャー（その環境）への非表示の追加引数に格納され、クロージャーとそのエンクロージングスコープの両方から参照によってアクセスされます（つまり、それらに加えられた変更は両方の場所で表示されます）。
コンパイラがこれが安全であると判断した場合、クロージャー環境はヒープまたはスタックに割り当てられます。

#### ループ内でのクロージャーの作成(Creating closures in loops)
クロージャーは参照によってローカル変数をキャプチャするため、ループ本体内での動作が望ましくないことがよくあります。
この動作を変更する方法の詳細については[closureScope](https://nim-lang.org/docs/system.html#closureScope)参照。

### 匿名プロシージャー(Anonymous Procs)
名前のないプロシージャは、他のプロシージャに渡すラムダ式として使用できます。

```nim
var cities = @["Frankfurt", "Tokyo", "New York", "Kyiv"]

cities.sort(proc (x,y: string): int =
    cmp(x.len, y.len))
```

式としてのProcsは、ネストされたprocとして最上位の実行可能コード内にも表示できます。
[sugar](https://nim-lang.org/docs/sugar.html)モジュールには`=>`マクロが含まれており、JavaScript,C#などの言語のようにラムダに似た匿名プロシージャのより簡潔な構文を有効にします。

### 関数(func)
`func`キーワードは、副作用のないプロシージャの短縮表記です。

```nim
func binarySearch[T](a: openArray[T]; elem: T): int
```
は下記の短縮形です。

```nim
proc binarySearch[T](a: openArray[T]; elem: T): int {.noSideEffect.}
```

### オーバーロードできない組込み機能(Nonoverloadable builtins)
次の組み込みプロシージャは、実装が単純であるため、オーバーロードできません（特別なセマンティックチェックが必要です）。

```nim
declared, defined, definedInScope, compiles, sizeOf,
is, shallowCopy, getAst, astToStr, spawn, procCall
```

したがって、通常の識別子よりもキーワードのように機能します。
ただし、キーワードとは異なり、再定義は`system`モジュールの定義をシャドウイングする場合があります。
このリストから、`x`は`f`に渡される前に型チェックできないため、ドット表記`x.f`で次のように記述しないでください。

```nim
declared, defined, definedInScope, compiles, getAst, astToStr
```

### Varパラメーター(Var parameters)
パラメーターのタイプには、`var`キーワードをプレフィックスとして付けることができます。

```nim
proc divmod(a, b: int; res, remainder: var int) =
  res = a div b
  remainder = a mod b

var
  x, y: int

divmod(8, 5, x, y) # modifies x and y
assert x == 1
assert y == 3
```

この例では、`res`と`remainder`は`var`パラメーターです。
`Var`パラメータはプロシージャによって変更でき、変更は呼び出し元に反映されます。
varパラメーターに渡される引数は左辺値でなければなりません。
Varパラメーターは、非表示ポインターとして実装されます。上記の例は次と同等です

```nim
proc divmod(a, b: int; res, remainder: ptr int) =
  res[] = a div b
  remainder[] = a mod b

var
  x, y: int
divmod(8, 5, addr(x), addr(y))
assert x == 1
assert y == 3
```

この例では、varパラメーターまたはポインターを使用して2つの戻り値を提供しています。
これは、タプルを返すことにより、よりクリーンな方法で実行できます。

```nim
proc divmod(a, b: int): tuple[res, remainder: int] =
  (a div b, a mod b)

var t = divmod(8, 5)

assert t.res == 1
assert t.remainder == 3
```

タプルのアンパックを使用して、タプルのフィールドにアクセスできます。

```nim
var (x, y) = divmod(8, 5) # tuple unpacking
assert x == 1
assert y == 3
```
注：`var`パラメーターは、効率的なパラメーターの受け渡しには必要ありません。
var以外のパラメーターは変更できないことで、コンパイラーが実行を高速化できると見なす場合、常に参照によって引数を自由に渡すことができます。

### Var return type
proc,converter,iteratorは`var`型を返す場合があります。
これは、戻り値が左辺値であり、呼び出し元が変更できることを意味します。

```nim
var g = 0

proc writeAccessToG(): var int =
  result = g

writeAccessToG() = 6
assert g == 6
```

暗黙的に導入されたポインターを使用して、その存続期間を超えてlocationにアクセスできる場合は、静的エラーです。

```nim
proc writeAccessToG(): var int =
  var g = 0
  result = g # Error!
```

イテレータの場合、タプルの戻り値型のコンポーネントには`var`型も含めることができます。

```nim
iterator mpairs(a: var seq[string]): tuple[key: int, val: var string] =
  for i in 0..a.high:
    yield (i, a[i])
```

標準ライブラリでは、`var`型を返すルーチンの名前はすべて、規則で接頭辞`m`で始まります。

`var T`で返すためのメモリの安全性は、単純な借用ルールによって保証されます：
`result`がヒープを指す場所を参照しない場合（つまり、`result = X`で`X`が`ptr`か`ref`アクセスを含まない）、ルーチンの最初のパラメーターによって逸脱する必要があります ：

```nim
proc forward[T](x: var T): var T =
  result = x # ok, deviated from the first parameter.

proc p(param: var int): var int =
  var x: int
  # we know 'forward' provides a view into the location deviated by
  # its first argument 'x'.
  result = forward(x) # Error: location is derived from ``x``
                      # which is not p's first parameter and lives
                      # on the stack.
```

つまり、`result`が指すライフタイムは最初のパラメーターのライフタイムに付加され、コールサイトでのメモリの安全性を検証するのに十分な知識です。

#### 今後の方向性(Future directions)
Nimの今後のバージョンでは、次のような構文を使用して借用ルールをより正確にできます。

```nim
proc foo(other: Y; container: var X): var T from container
```

ここで、`var T from container`は、locationが2番目のパラメータ（この場合はcontainerと呼ばれる）から外れていることを明示的に公開します。
構文`var T from p`は、`varTy[T, 1]`と互換性のない型`varTy[T, 2]`を指定します。

### 添字演算子のオーバーロード(Overloading of the subscript operator)
配列/openarrays/sequencesの添え字演算子`[]`はオーバーロードできます。

## マルチメソッド(Multi-methods)
注：Nim 0.20以降、マルチメソッドを使用するには、コンパイル時に`--multimethods:on`を明示的に渡す必要があります。

プロシージャは常に静的ディスパッチを使用します。
マルチメソッドは動的ディスパッチを使用します。
動的ディスパッチがオブジェクトで機能するには、オブジェクトが参照型である必要があります。

```nim
type
  Expression = ref object of RootObj ## abstract base class for an expression
  Literal = ref object of Expression
    x: int
  PlusExpr = ref object of Expression
    a, b: Expression

method eval(e: Expression): int {.base.} =
  # override this base method
  raise newException(CatchableError, "Method without implementation override")

method eval(e: Literal): int = return e.x

method eval(e: PlusExpr): int =
  # watch out: relies on dynamic binding
  result = eval(e.a) + eval(e.b)

proc newLit(x: int): Literal =
  new(result)
  result.x = x

proc newPlus(a, b: Expression): PlusExpr =
  new(result)
  result.a = a
  result.b = b

echo eval(newPlus(newPlus(newLit(1), newLit(2)), newLit(4)))
```

この例では、コンストラクタ`newLit`および`newPlus`は静的バインディングを使用する必要があるためprocですが、`eval`は動的バインディングを必要とするためメソッドです。

例に見られるように、baseメソッドにはbaseプラグマで注釈を付ける必要があります。
baseプラグマは、maseメソッドmがmの呼び出しが引き起こす可能性のあるすべての効果を決定するための基盤として使用されることをプログラマーに思い出させる役割も果たします。

注：メソッドのコンパイル時実行は（まだ）サポートされていません。

注：Nim 0.20以降、ジェネリックメソッドは廃止されました。

### procCallによる動的メソッド解決の禁止(Inhibit dynamic method resolution via procCall)
組み込みのsystem.procCallを使用して、動的メソッドの解決を禁止できます。
これは、従来のOOP言語が提供するsuperキーワードにいくらか匹敵します。

```nim
type
  Thing = ref object of RootObj
  Unit = ref object of Thing
    x: int

method m(a: Thing) {.base.} =
  echo "base"

method m(a: Unit) =
  # Call the base method:
  procCall m(Thing(a))
  echo "1"
```

## Iterators and the for statement
for文はコンテナの要素を反復処理するための抽象的メカニズムです。
これを行うには、イテレータに依存します。
`while`ステートメントと同様に、`for`ステートメントは暗黙的な`block`を開くため、`break`ステートメントで離脱することができます。

forループは反復変数を宣言します。それらのスコープはループ本体の終わりまで到達します。反復変数の型は、イテレータの戻り型によって推測されます。

イテレータは、forループのコンテキストで呼び出すことができることを除いて、プロシージャに似ています。
イテレータは、抽象型の反復を指定する方法を提供します。
`for`ループの実行における重要な役割は、呼び出されたイテレーターで`yield`ステートメントを果たします。
`yield`ステートメントに到達すると、データは`for`ループ変数にバインドされ、`for`ループの本体で制御が継続されます。
イテレータのローカル変数と実行状態は、呼び出し間で自動的に保存されます。

```nim
# this definition exists in the system module
iterator items*(a: string): char {.inline.} =
  var i = 0
  while i < len(a):
    yield a[i]
    inc(i)

for ch in items("hello world"): # `ch` is an iteration variable
  echo ch
```

コンパイラは次のようにプログラマが書いたかのようなコードを生成します。

```nim
var i = 0
while i < len(a):
  var ch = a[i]
  echo ch
  inc(i)
```

イテレータがタプルを生成する場合、タプル内のコンポーネントと同じ数の反復変数が存在する可能性があります。
i番目の反復変数の型は、i番目のコンポーネントの型です。
つまり、forループコンテキストでの暗黙的なタプルのアンパックがサポートされています。

### 暗黙的なアイテム/ペアの呼び出し(Implict items/pairs invocations)
forループ式`e`がイテレータを示さず、forループに変数が1つだけある場合、forループ式は`items(e)`に書き換えられます。すなわち、アイテムイテレータが暗黙的に呼び出されます。

```nim
for x in [1,2,3]: echo x
```
forループにちょうど2つの変数がある場合、`pairs`反復子が暗黙的に呼び出されます。

識別子のアイテム/ペアのシンボル検索は、書き換えステップの後に実行されるため、アイテム/ペアのすべてのオーバーロードが考慮されます。

### First class iterators
Nimには2種類のイテレータがあります：インラインイテレータとクロージャイテレータです。
インラインイテレータは、コンパイラによって常にインライン化され、抽象化のオーバーヘッドがゼロになるイテレータですが、コードサイズが大幅に増加する可能性があります。

注意：インラインイテレーターのforループの本体は、イテレーターコードに表示される各`yield`ステートメントにインライン化されるため、理想的には、コードが肥大化しないように、単一のyieldを含むようにコードをリファクタリングする必要があります。

インラインイテレータは、second class citizensです。
テンプレート、マクロ、その他のインラインイテレータなど、他のインラインコード機能にのみパラメーターとして渡すことができます。

それとは対照的に、クロージャイテレータはより自由に渡すことができます。

```nim
iterator count0(): int {.closure.} =
  yield 0

iterator count2(): int {.closure.} =
  var x = 1
  yield x
  inc x
  yield x

proc invoke(iter: iterator(): int {.closure.}) =
  for x in iter(): echo x

invoke(count0)
invoke(count2)
```

クロージャイテレータとインラインイテレータにはいくつかの制限があります。

- 今のところ、クロージャイテレータはコンパイル時に実行できません。
- returnはクロージャイテレータで許可されていますが、インラインイテレータでは許可されておらず（しかしほとんど有用ではありません）反復を終了します。
- インラインイテレータもクロージャイテレータも再帰的ではありません。
- インラインイテレータにもクロージャイテレータにも特別な`result`変数はありません。
- クロージャイテレータは、jsバックエンドではサポートされていません。

`{.closure.}`とも`{.inline.}`とも、明示的にマークされていないイテレータはデフォルトでインラインですが、これは実装の将来のバージョンで変更されることがあります。

`iterator`型は常に呼び出し規約により暗黙的にクロージャーです。
次の例は、イテレータを使用して共同作業システムを実装する方法を示しています。

```nim
# simple tasking:
type
  Task = iterator (ticker: int)

iterator a1(ticker: int) {.closure.} =
  echo "a1: A"
  yield
  echo "a1: B"
  yield
  echo "a1: C"
  yield
  echo "a1: D"

iterator a2(ticker: int) {.closure.} =
  echo "a2: A"
  yield
  echo "a2: B"
  yield
  echo "a2: C"

proc runTasks(t: varargs[Task]) =
  var ticker = 0
  while true:
    let x = t[ticker mod t.len]
    if finished(x): break
    x(ticker)
    inc ticker

runTasks(a1, a2)
```
組み込みの`system.finished`を使用して、イテレーターが操作を完了したかどうかを判別できます。
既に作業を終了したイテレーターを呼び出そうとしても例外は発生しません。

`system.finished`はエラーが発生しやすいことに注意してください。
これは、イテレータが終了した後に1回だけ`true`を返すためです。

```nim
iterator mycount(a, b: int): int {.closure.} =
  var x = a
  while x <= b:
    yield x
    inc x

var c = mycount # instantiate the iterator
while not finished(c):
  echo c(1, 3)

# Produces
1
2
3
0
```

かわりに、下のコードを使用する必要があります。

```nim
var c = mycount # instantiate the iterator
while true:
  let value = c(1, 3)
  if finished(c): break # and discard 'value'!
  echo value
```

イテレータは実際にペア`(value, done)`を返し、`finished`は非表示の`done`フィールドにアクセスするために使用されると考えると役立ちます。

クロージャイテレータは再開可能な関数であるため、すべての呼び出しに引数を提供する必要があります。
この制限を回避するには、外部ファクトリプロシージャのパラメーターをキャプチャします。

```nim
proc mycount(a, b: int): iterator (): int =
  result = iterator (): int =
    var x = a
    while x <= b:
      yield x
      inc x

let foo = mycount(1, 4)

for f in foo():
  echo f
```

## コンバーター(Converters)
コンバーターは、「暗黙的に変換可能な」型の関係を拡張することを除いて、通常のプロシージャに似ています([変換可能な関係](#変換可能な関係Convertible-relation)を参照)。

```nim
# bad style ahead: Nim is not C.
converter toBool(x: int): bool = x != 0

if 4:
  echo "compiles"
```

読みやすくするために、コンバーターを明示的に呼び出すこともできます。
暗黙的なコンバーターチェーンはサポートされていないことに注意してください。
タイプAからタイプBへ、およびタイプBからタイプCへのコンバーターがある場合、AからCへの暗黙的な変換は提供されません。

## 型セクション(Type sections)
例：

```nim
type # example demonstrating mutually recursive types
  Node = ref object  # an object managed by the garbage collector (ref)
    le, ri: Node     # left and right subtrees
    sym: ref Sym     # leaves contain a reference to a Sym
  
  Sym = object       # a symbol
    name: string     # the symbol's name
    line: int        # the line the symbol was declared in
    code: Node       # the symbol's abstract syntax tree
```

型セクションは、`type`キーワードで始まります。 複数のタイプ定義が含まれています。
型定義は、型を名前にバインドします。型定義は、再帰的または相互再帰的にできます。
相互再帰型は、単一の型セクション内でのみ可能です。
オブジェクトや列挙型などの名目上の型は、型セクションでのみ定義できます。

## 例外処理(Exception handling)

### Try statement
例：

```nim
# read the first two lines of a text file that should contain numbers
# and tries to add them
var
  f: File
if open(f, "numbers.txt"):
  try:
    var a = readLine(f)
    var b = readLine(f)
    echo "sum: " & $(parseInt(a) + parseInt(b))
  except OverflowError:
    echo "overflow!"
  except ValueError:
    echo "could not convert string to integer"
  except IOError:
    echo "IO error!"
  except:
    echo "Unknown exception!"
  finally:
    close(f)
```

`try`の後のステートメントは、例外`e`が発生しない限り、順番に実行されます。
`e`の例外タイプが`except`節にリストされているものと一致する場合、対応するステートメントが実行されます。
`except`節に続くステートメントは、例外ハンドラーと呼ばれます。

空の`except`節は、他にリストされていない例外がある場合に実行されます。
`if`ステートメントの`else`句に似ています。

`finally`節がある場合は、常に例外ハンドラーの後に実行されます。

例外は例外ハンドラーで消費されます。
ただし、例外ハンドラは別の例外を発生させる場合があります。
例外が処理されない場合、コールスタックを通じて伝播されます。
これは、多くの場合、プロシージャの残りの部分（finally節内にない分）は実行されない（例外が発生した場合）。

### Try expression
`try`は式としても使用できます。
`try`ブランチの型は例外ブランチの型に適合する必要がありますが、`finally`ブランチのタイプは常に`void`でなければなりません。

```nim
let x = try: parseInt("133a")
        except: -1
        finally: echo "hi"
```

コードの混乱を防ぐために、解析の制限があります。`try`が`(`に続く場合、1ライナーとして記述する必要があります。

```nim
let x = (try: parseInt("133a") except: -1)
```

### Except clauses
except節内では、次の構文を使用して現在の例外にアクセスできます。

```nim
try:
  # ...
except IOError as e:
  # Now use "e"
  echo "I/O error: " & e.msg
```

または、`getCurrentException`を使用して、発生した例外を取得することもできます。

```nim
try:
  # ...
except IOError:
  let e = getCurrentException()
  # Now use "e"
```

`getCurrentException`は常に`ref Exception`タイプを返すことに注意してください。
適切な型の変数が必要な場合（上記の例では`IOError`）、明示的に変換する必要があります。

```nim
try:
  # ...
except IOError:
  let e = (ref IOError)(getCurrentException())
  # "e" is now of the proper type
```

ただし、これはほとんど必要ありません。
最も一般的なケースは、`e`からエラーメッセージを抽出することです。
このような状況では、`getCurrentExceptionMsg`を使用するだけで十分です。

```nim
try:
  # ...
except:
  echo getCurrentExceptionMsg()
```

### Defer statement
`try finally`ステートメントの代わりに、`defer`ステートメントを使用できます。

現在のブロックの`defer`に続くステートメントは、暗黙的なtryブロックにあると見なされます。

```nim
proc main =
  var f = open("numbers.txt")
  defer: close(f)
  f.write "abc"
  f.write "def"
```
は下記に書き換えられます。

```nim
proc main =
  var f = open("numbers.txt")
  try:
    f.write "abc"
    f.write "def"
  finally:
    close(f)
```
トップレベルの`defer`ステートメントは、そのようなステートメントが何を参照すべきかが不明であるため、サポートされていません。

### Raise statement
例：

```nim
raise newEOS("operating system failed")
```

配列のインデックス付け、メモリ割り当てなどの組み込み操作は別として、`raise`ステートメントは例外を発生させる唯一の方法です。
例外名が指定されていない場合、現在の例外が再発生します。
再発生する例外がない場合、ReraiseError例外が発生します。
したがって、`raise`ステートメントは常に例外を発生させます。

### Exception hierarchy
例外ツリーは[system](https://nim-lang.org/docs/system.html)モジュールで定義されます。
すべての例外は`system.Exception`から継承します。
プログラミングのバグを示す例外は`system.Defect`（`Exception`のサブタイプ）を継承し、厳密にはキャッチできません。
これらはプロセス全体を終了する操作にマップすることもできるためです。
キャッチできるその他のランタイムエラーを示す例外は、`system.CatchableError`（`Exception`のサブタイプ）から継承します。

### Imported exceptions
インポートされたC ++例外を発生/キャッチすることができます。
`importcpp`を使用してインポートされた型は、発生または捕捉できます。
例外は値によってraiseされ、参照によってキャッチされます。

```nim
type
  std_exception {.importcpp: "std::exception", header: "<exception>".} = object

proc what(s: std_exception): cstring {.importcpp: "((char *)#.what())".}

try:
  raise std_exception()
except std_exception as ex:
  echo ex.what()
```

## エフェクトシステム(Effect system)

### 例外追跡(Exception tracking)
Nimは例外追跡をサポートしています。
raisesプラグマが明示的にproc/iterator/method/converterで発生することを許可された例外を定義するために使用することができます。
コンパイラーはこれを検証します。

```nim
proc p(what: bool) {.raises: [IOError, OSError].} =
  if what: raise newException(IOError, "IO")
  else: raise newException(OSError, "OS")
```

空の`raises`リスト(`raises: []`)は、例外が発生しないことを意味します。

```nim
proc p(): bool {.raises: [].} =
  try:
    unsafeCall()
    result = true
  except:
    result = false
```

`raises`リストは、proc型に付加することもできます。これは型の互換性に影響します。

```nim
type
  Callback = proc (s: string) {.raises: [IOError].}
var
  c: Callback

proc p(x: string) =
  raise newException(OSError, "OS")

c = p # type error
```

ルーチン`p`に対して、コンパイラは推論規則を使用して、発生する可能性のある例外のセットを決定します。
アルゴリズムは`p`の呼び出しグラフで動作します

- proc型`T`を介したすべての間接呼び出しは、`system.Exception`（例外階層の基本型）を発生させると想定されているため、`T`に明示的な発生リストがない限り、例外は発生しません。
ただし、呼び出しの形式が`f(...)`の場合、`f`は現在解析されているルーチンのパラメーターであり、無視されます。
呼び出しは楽観的に作用がないと見なされます。ルール2はこのケースを補います。
- 呼び出し自体ではない（かつnilではない）呼び出し内にあるproc型のすべての式は、何らかの方法で間接的に呼び出されると想定されるため、その発生リストは`p`のraisesリストに追加されます。
- （forward宣言またはimportcプラグマによる）未知のbodyを持つproc `q`の呼び出しはすべて、`q`に明示的な発生リストがない限り、`system.Exception`を発生させると想定されます。
- メソッド`m`へのすべての呼び出しは、`m`に明示的な`raises`リストがない限り、`system.Exception`を発生させると想定されます。
- 他のすべての呼び出しについて、解析は正確な`raises`リストを決定できます。
- `raises`リストを決定するために、`p`の`raise`および`try`ステートメントが考慮されます。

ルール1-2は、以下の機能を保証します。

```nim
proc noRaise(x: proc()) {.raises: [].} =
  # unknown call that might raise anything, but valid:
  x()

proc doRaise() {.raises: [IOError].} =
  raise newException(IOError, "IO")

proc use() {.raises: [].} =
  # doesn't compile! Can raise IOError!
  noRaise(doRaise)
```

そのため、多くの場合、コールバックによってコンパイラーのエフェクト分析が過度に保守的になることはありません。

### タグトラッキング(Tag tracking)
例外追跡は、Nimのエフェクトシステムの一部です。
例外を発生させることはeffectです。
他のeffectも定義できます。
ユーザー定義のeffectは、ルーチンにタグを付け、このタグに対してチェックを実行する手段です。

```nim
type IO = object ## input/output effect
proc readLine(): string {.tags: [IO].} = discard

proc no_IO_please() {.tags: [].} =
  # the compiler prevents this:
  let x = readLine()
```

タグは型名でなければなりません。`tags`リスト（`raises`リストのような）もprocタイプに付加できます。これは型の互換性に影響します。

### エフェクトプラグマ(Effects pragma)
`effects`プラグマは、プログラマーによるエフェクト解析を支援するように設計されています。
これは、コンパイラーがすべての推論されたエフェクトを`efects`の位置まで出力するステートメントです。

```nim
proc p(what: bool) =
  if what:
    raise newException(IOError, "IO")
    {.effects.}
  else:
    raise newException(OSError, "OS")
```

コンパイラは、`IOError`が発生する可能性があるというヒントメッセージを生成します。
`OSError`は、エフェクトプラグマが表示されるブランチで発生できないため、リストされていません。

## ジェネリック(Generics)
ジェネリックproc,イテレーター,型を型パラメータによってパラメータ化するNimの手法です。
コンテキストに応じて、型パラメーターを導入するかジェネリックproc,イテレーター,型をインスタンス化するために角カッコ`[]`が使用されます。

次の例はジェネリック2分木がモデル化できることを示しています。

```nim
type
  BinaryTree*[T] = ref object # BinaryTree is a generic type with
                              # generic param ``T``
    le, ri: BinaryTree[T]     # left and right subtrees; may be nil
    data: T                   # the data stored in a node

proc newNode*[T](data: T): BinaryTree[T] =
  # constructor for a node
  result = BinaryTree[T](le: nil, ri: nil, data: data)

proc add*[T](root: var BinaryTree[T], n: BinaryTree[T]) =
  # insert a node into the tree
  if root == nil:
    root = n
  else:
    var it = root
    while it != nil:
      # compare the data items; uses the generic ``cmp`` proc
      # that works for any type that has a ``==`` and ``<`` operator
      var c = cmp(it.data, n.data)
      if c < 0:
        if it.le == nil:
          it.le = n
          return
        it = it.le
      else:
        if it.ri == nil:
          it.ri = n
          return
        it = it.ri

proc add*[T](root: var BinaryTree[T], data: T) =
  # convenience proc:
  add(root, newNode(data))

iterator preorder*[T](root: BinaryTree[T]): T =
  # Preorder traversal of a binary tree.
  # Since recursive iterators are not yet implemented,
  # this uses an explicit stack (which is more efficient anyway):
  var stack: seq[BinaryTree[T]] = @[root]
  while stack.len > 0:
    var n = stack.pop()
    while n != nil:
      yield n.data
      add(stack, n.ri)  # push right subtree onto the stack
      n = n.le          # and follow the left pointer

var
  root: BinaryTree[string] # instantiate a BinaryTree with ``string``
add(root, newNode("hello")) # instantiates ``newNode`` and ``add``
add(root, "world")          # instantiates the second ``add`` proc
for str in preorder(root):
  stdout.writeLine(str)
```

`T`は、ジェネリック型パラメーターまたは型変数と呼ばれます。

### Is operator
`is`演算子は、型の等価性をチェックするためにセマンティック解析中に評価されます。
したがって、ジェネリックコード内の型の特化に非常に役立ちます。

```nim
type
  Table[Key, Value] = object
    keys: seq[Key]
    values: seq[Value]
    when not (Key is string): # empty value for strings used for optimization
      deletedKeys: seq[bool]
```

### 型クラス(Type Classes)
型クラスは、オーバーロード解決または`is`演算子のコンテキストで型と照合するために使用できる特別な擬似型です。
Nimは次の組み込み型クラスをサポートしています。

|型クラス|マッチ|
|:---|:---|
|`object`|any object type|
|`tuple`|any tuple type|
|`enum`|any enumeration|
|`proc`|any proc type|
|`ref`|any `ref` type|
|`ptr`|any `ptr` type|
|`var`|any `var` type|
|`distinct`|any distinct type|
|`array`|any array type|
|`set`|any set type|
|`seq`|any seq type|
|`auto`|any type|
|`any`|distinct auto (see below)|

さらに、すべてのジェネリック型は、ジェネリック型のインスタンス化と一致する同じ名前の型クラスを自動的に作成します。

標準のブール演算子を使用して型クラスを組み合わせて、より複雑な型クラスを作成できます。

```nim
# create a type class that will match all tuple and object types
type RecordType = tuple or object

proc printFields[T: RecordType](rec: T) =
  for key, value in fieldPairs(rec):
    echo key, " = ", value
```

型クラスの構文はMLに似た言語のADT/代数データ型の構文に似ているように見えますが、型クラスは型のインスタンス化で適用される静的な制約であることを理解する必要があります。
型クラスは実際にはそれらの型ではなく、最終的に何らかの特異な型に解決される一般的な「チェック」を提供するシステムです。
型のクラスでは、オブジェクトのバリアントやメソッドとは異なり、実行時の型のダイナミズムは許可されません。

例として、次はコ​​ンパイルされません。

```nim
type TypeClass = int | string
var foo: TypeClass = 2 # foo's type is resolved to an int here
foo = "this will fail" # error here, because foo is an int
```

Nimでは、ジェネリック型パラメーターの型制約として型クラスと通常の型を指定できます。

```nim
proc onlyIntOrString[T: int|string](x, y: T) = discard

onlyIntOrString(450, 616) # valid
onlyIntOrString(5.0, 0.0) # type mismatch
onlyIntOrString("xy", 50) # invalid as 'T' cannot be both at the same time
```

### 暗黙のジェネリック(Implicit generics)
型クラスは、パラメータの型として直接使用できます。

```nim
# create a type class that will match all tuple and object types
type RecordType = tuple or object

proc printFields(rec: RecordType) =
  for key, value in fieldPairs(rec):
    echo key, " = ", value
```

このような方法で型クラスを利用するプロシージャは、暗黙的にジェネリックであると見なされます。
これらは、プログラム内で使用されるparam型の一意の組み合わせごとに1回インスタンス化されます。

デフォルトでは、オーバーロードの解決中に、各名前付き型クラスは厳密に1つの具象型にバインドされます。
このような型クラスをバインド型と呼びます。これを説明するために、システムモジュールから直接取得した例を次に示します。

```nim
proc `==`*(x, y: tuple): bool =
  ## requires `x` and `y` to be of the same tuple type
  ## generic ``==`` operator for tuples that is lifted from the components
  ## of `x` and `y`.
  result = true
  for a, b in fields(x, y):
    if a != b: result = false
```

あるいは、`distict`型修飾子を型クラスに適用して、型クラスに一致する各パラメーターが異なる型にバインドできるようにすることができます。
このような型クラスは、多くの型のバインドと呼ばれます。

暗黙的なジェネリックスタイルで記述されたProcは、多くの場合、一致したジェネリック型の型パラメーターを参照する必要があります。
これらは、ドット構文を使用して簡単にアクセスできます。

```nim
type Matrix[T, Rows, Columns] = object
  ...

proc `[]`(m: Matrix, row, col: int): Matrix.T =
  m.data[col * high(Matrix.Columns) + row]
```

暗黙のジェネリックを示す他の例は次のとおりです。

```nim
proc p(t: Table; k: Table.Key): Table.Value

# is roughly the same as:

proc p[Key, Value](t: Table[Key, Value]; k: Key): Value
```

```nim
proc p(a: Table, b: Table)

# is roughly the same as:

proc p[Key, Value](a, b: Table[Key, Value])
```

```nim
proc p(a: Table, b: distinct Table)

# is roughly the same as:

proc p[Key, Value, KeyB, ValueB](a: Table[Key, Value], b: Table[KeyB, ValueB])
```

パラメータタイプとして使用される `typedesc`は、暗黙的なジェネリックも導入します。`typedesc`には独自のルールセットがあります。

```nim
proc p(a: typedesc)

# is roughly the same as:

proc p[T](a: typedesc[T])
```

typedescは「bind many」型クラスです。

```nim
proc p(a, b: typedesc)

# is roughly the same as:

proc p[T, T2](a: typedesc[T], b: typedesc[T2])
```

タイプ`typedesc`のパラメーター自体は、型として使用できます。
型として使用される場合は、基になる型です。（つまり、「typedesc」の1つのレベルが取り除かれます）。

```nim
proc p(a: typedesc; b: a) = discard

# is roughly the same as:
proc p[T](a: typedesc[T]; b: T) = discard

# hence this is a valid call:
p(int, 4)
# as parameter 'a' requires a type, but 'b' requires a value.
```

### 一般的な推論の制限(Generic inference restrictions)
タイプ`var T`および`typedesc [T]`は、一般的なインスタンス化では推測できません。以下は許可されていません。

```nim
proc g[T](f: proc(x: T); x: T) =
  f(x)

proc c(y: int) = echo y
proc v(y: var int) =
  y += 100
var i: int

# allowed: infers 'T' to be of type 'int'
g(c, 42)

# not valid: 'T' is not inferred to be of type 'var int'
g(v, i)

# also not allowed: explict instantiation via 'var int'
g[var int](v, i)
```

### ジェネリックでのシンボル検索(Symbol lookup in generics)

#### オープンシンボルとクローズシンボル(Open and Closed symbols)
ジェネリックのシンボルバインディングルールはわずかに微妙です。
「オープン」および「クローズ」シンボルがあります。
「閉じた」シンボルはインスタンス化コンテキストで再バインドできませんが、「開いた」シンボルは再バインドできます。
デフォルトでは、オーバーロードされたシンボルは開いており、他のすべてのシンボルは閉じています。

オープンシンボルは、2つの異なるコンテキストで検索されます。
定義時のコンテキストとインスタンス化時のコンテキストの両方が考慮されます。

```nim
type
  Index = distinct int

proc `==` (a, b: Index): bool {.borrow.}

var a = (0, 0.Index)
var b = (0, 0.Index)

echo a == b # works!
```

例では、タプルのジェネリック`==`（システムモジュールで定義されている）は、タプルのコンポーネントの`==`演算子を使用します。
ただし、インデックス型の`==`はタプルの`==`の後に定義されます。
ただし、インスタンス化では現在定義されているシンボルも考慮されるため、例はコンパイルされます。

### Mixin statement
mixin宣言により、シンボルを強制的に開くことができます。

```nim
proc create*[T](): ref T =
  # there is no overloaded 'init' here, so we need to state that it's an
  # open symbol explicitly:
  mixin init
  new result
  init result
```

`mixin`ステートメントは、テンプレートとジェネリックでのみ意味があります。

### Bind statement
`bind`ステートメントは、`mixin`ステートメントに対応しています。
それは、早期にバインドされるべき識別子を明示的に宣言するために使用できます（つまり、識別子はテンプレート/ジェネリック定義のスコープ内で検索されるべきです）。

```nim
# Module A
var
  lastId = 0

template genId*: untyped =
  bind lastId
  inc(lastId)
  lastId
```

```nim
# Module B
import A

echo genId()
```

ただし、`bind`は定義スコープからのシンボルバインドがデフォルトであるため、ほとんど役に立ちません。

`bind`ステートメントは、テンプレートとジェネリックでのみ意味があります。

## テンプレート(Templates)

テンプレートは、マクロの単純な形式です。
これは、Nimの抽象構文ツリーで動作する単純な置換メカニズムです。コンパイラのセマンティックパスで処理されます。

テンプレートを呼び出す構文は、プロシージャを呼び出すのと同じです。

例：

```nim
template `!=` (a, b: untyped): untyped =
  # this definition exists in the System module
  not (a == b)

assert(5 != 6) # the compiler rewrites that to: assert(not (5 == 6))
```

`!=`, `>`, `>=`, `in`, `notin`, `isnot`演算子は実際にはテンプレートです。

`a > b`は`b < a`に変換され、`a in b`は`contains(b, a)`に変換されます。
`notin`と`isnot`には明らかな意味があります。

テンプレートの「型」には`untyped`,`typed`,`typedesc`シンボルがあります。
これらは「メタタイプ」であり、特定のコンテキストでのみ使用できます。
通常の型も使用できます。これは、`typed`式が期待されることを意味します。

### typedパラメータとuntypedパラメータ

`untyped`パラメータは式がテンプレートに渡される前に、シンボル検索と型解決が実行されないことを意味します。
これは、たとえば、宣言されていない識別子をテンプレートに渡すことができることを意味します。

```nim
template declareInt(x: untyped) =
  var x: int

declareInt(x) # valid
x = 3
```

```nim
template declareInt(x: typed) =
  var x: int

declareInt(x) # invalid, because x has not been declared and so has no type
```

すべてのパラメーターが`untyped`なテンプレートは、即時テンプレートと呼ばれます。
歴史的な理由により、テンプレートには`immediate`プラグマで明示的に注釈を付けることができますが、これらのテンプレートはオーバーロード解決に関与せず、パラメーターの型はコンパイラーによって無視されます。
明示的なimmediateテンプレートは非推奨になりました。

注：歴史的な理由により、`stmt`は`typed`のエイリアスであり、`expr`は`untyped`のエイリアスでしたが、削除されました。

### コードブロックをテンプレートに渡す(Passing a code block to a template)
ステートメントのブロックを、特別な`:`構文に従って最後の引数としてテンプレートに渡すことができます。

```nim
template withFile(f, fn, mode, actions: untyped): untyped =
  var f: File
  if open(f, fn, mode):
    try:
      actions
    finally:
      close(f)
  else:
    quit("cannot open: " & fn)

withFile(txt, "ttempl3.txt", fmWrite):  # special colon
  txt.writeLine("line 1")
  txt.writeLine("line 2")
```

この例では、2つの`writeLine`ステートメントが`actions`パラメーターにバインドされています。

通常、コードのブロックをテンプレートに渡すには、ブロックを受け入れるパラメーターの型が`untyped`である必要があります。
シンボル検索はテンプレートのインスタンス化される時まで遅延されるためです。

```nim
template t(body: typed) =
  block:
    body

t:
  var i = 1
  echo i

t:
  var i = 2  # fails with 'attempt to redeclare i'
  echo i
```

上記のコードは、`i`が既に宣言されているという謎のエラーメッセージで失敗します。
これは`var i = ...`が`body`パラメータに渡される前に（bodyがtypedであるため）型チェックされる必要があり、Nimでの型チェックがシンボル検索を意味するためです。
シンボルの検索を成功させるには、現在の（つまり外側の）スコープに`i`を追加する必要があります。
型チェックの後、これらのシンボルテーブルへの追加はロールバックされません（良かれ悪しかれ）。
同じコードでも`untyped`であれば、テンプレートに渡されるbodyは型チェックされないため機能します。

```nim
template t(body: untyped) =
  block:
    body

t:
  var i = 1
  echo i

t:
  var i = 2  # compiles
  echo i
```

### untypedの可変長引数(Varargs of untyped)
型チェックをを防ぐ`untyped`メタ型に加えて、`varargs[untyped]`もあり、パラメータの数さえ固定されません。

```nim
template hideIdentifiers(x: varargs[untyped]) = discard

hideIdentifiers(undeclared1, undeclared2)
```

ただし、テンプレートは可変引数を反復処理できないため、この機能は一般的にマクロに非常に役立ちます。

### テンプレートでのシンボルバインディング(Symbol binding in templates)
テンプレートは衛生的なマクロなので、新しいスコープを開きます。
ほとんどのシンボルは、テンプレートを定義したスコープからバインドされます。

```nim
# Module A
var
  lastId = 0

template genId*: untyped =
  inc(lastId)
  lastId
```

```nim
# Module B
import A

echo genId() # Works as 'lastId' has been bound in 'genId's defining scope
```

ジェネリックのように、シンボルのバインドは、`mixin`または`bind`ステートメントを介して影響を受ける可能性があります。

### 識別子の構築(Identifier construction)
テンプレートでは、バッククォート`表記を使用して識別子を作成できます。

```nim
template typedef(name: untyped, typ: typedesc) =
  type
    `T name`* {.inject.} = typ
    `P name`* {.inject.} = ref `T name`

typedef(myint, int)
var x: PMyInt
```

この例では、`name`は`myint`でインスタンス化されるため、`T name`は`Tmyint`になります。

### テンプレートパラメータのルックアップルール(Lookup rules for template parameters)

テンプレート内のパラメーター`p`は式`x.p`でも置換されます。
したがって、テンプレート引数をフィールド名として使用でき、完全修飾されている場合でもグローバルシンボルを同じ引数名でシャドウできます。

```nim
# module 'm'

type
  Lev = enum
    levA, levB

var abclev = levB

template tstLev(abclev: Lev) =
  echo abclev, " ", m.abclev

tstLev(levA)
# produces: 'levA levA'
```

しかし、グローバルシンボルは`bind`ステートメントによって適切にキャプチャできます。

```nim
# module 'm'

type
  Lev = enum
    levA, levB

var abclev = levB

template tstLev(abclev: Lev) =
  bind m.abclev
  echo abclev, " ", m.abclev

tstLev(levA)
# produces: 'levA levB'
```

### テンプレートの衛生(Hygiene in templates)
デフォルトのテンプレートは衛生的です。
テンプレートで宣言されたローカル識別子は、インスタンス化コンテキストではアクセスできません。

```nim
template newException*(exceptn: typedesc, message: string): untyped =
  var
    e: ref exceptn  # e is implicitly gensym'ed here
  new(e)
  e.msg = message
  e

# so this works:
let e = "message"
raise newException(IoError, e)
```

テンプレートで宣言されたシンボルがインスタンス化スコープに公開されるかどうかは、injectおよびgensymプラグマによって制御されます。
gensymされたシンボルは公開されませんが、injectされます。

`type`,`var`,`let`,`const`の実体シンボルのデフォルトは`gensym`であり、`proc`,`iterator`,`converter`,`template`,`macro`は`inject`です。
ただし、実体の名前がテンプレートパラメータとして渡される場合、それはinjectされたシンボルです。

```nim
template withFile(f, fn, mode: untyped, actions: untyped): untyped =
  block:
    var f: File  # since 'f' is a template param, it's injected implicitly
    ...

withFile(txt, "ttempl3.txt", fmWrite):
  txt.writeLine("line 1")
  txt.writeLine("line 2")
```

`inject`と`gensym`プラグマはsecond class注釈です。
テンプレート定義以外のセマンティクスはないため、抽象化できません。

```nim
{.pragma myInject: inject.}

template t() =
  var x {.myInject.}: int # does NOT work
```

テンプレートの衛生をなくすには、テンプレートに`dirty`プラグマを使用できます。`inject`と`gensym`は、`dirty`テンプレートでは効果がありません。

`gensym`化シンボルは`x.field`構文の`field`として使用できません。
また、`ObjectConstruction(field: value)`および`namedParameterCall(field = value)`構文構造体でも使用できません。

その理由は、次のようなコードです。

```nim
type
  T = object
    f: int

template tmp(x: T) =
  let f = 34
  echo x.f, T(f: 4)
```

これは期待どおりに動作するはずです。

ただし、これはメソッド呼び出し構文が`gensym`化シンボルに対して使用できないことを意味します。

```nim
template tmp(x) =
  type
    T {.gensym.} = int
  
  echo x.T # invalid: instead use:  'echo T(x)'.

tmp(12)
```

注：バージョン1より前のNimコンパイラーは、この要件に関してより寛大でした。
移行期間には`--useVersion:0.19`スイッチを使用します。

### メソッド呼び出し構文の制限(Limitations of the method call syntax)
`x.f`の式`x`は`f(x)`書き換える必要があると判断する前に、セマンティックをチェックする必要があります（つまり、シンボル検索と型チェック）。
したがって、ドット構文には、テンプレート/マクロの呼び出しに使用する場合、いくつかの制限があります。

```nim
template declareVar(name: untyped) =
  const name {.inject.} = 45

# Doesn't compile:
unknownIdentifier.declareVar
```

別の一般的な例は次のとおりです。

```nim
from sequtils import toSeq

iterator something: string =
  yield "Hello"
  yield "World"

var info = something().toSeq
```

ここでの問題は、`toSeq`がシーケンスに変換する機会を得る前に、このコンテキストではiteratorとしての`something()`がこのコンテキストで呼び出し可能でないことをコンパイラが既に決定していることです。

## マクロ(Macros)
マクロは、コンパイル時に実行される特別な関数です。
通常、マクロの入力は、渡されるコードの抽象構文木（AST）です。
その後、マクロはそれに対して変換を行い、変換されたASTを返すことができます。
これを使用して、カスタム言語機能を追加し、ドメイン固有の言語を実装できます。

マクロ呼び出しでは、セマンティック解析が完全に上から下、左から右に進みません。
代わりに、セマンティック解析は少なくとも2回行われます。

- セマンティック解析は、マクロ呼び出しを認識して解決します。
- コンパイラーはマクロ本体を実行します（他のprocを呼び出す場合があります）。
- マクロ呼び出しのASTを、マクロによって返されたASTに置き換えます。
- コードのその領域のセマンティック解析を繰り返します。
- マクロによって返されたASTに他のマクロ呼び出しが含まれている場合、このプロセスが繰り返されます。

マクロは高度なコンパイル時コード変換を可能にしますが、Nimの構文を変更することはできません。
ただし、Nimの構文はいずれにせよ十分な柔軟性があるため、これは実際の制限ではありません。

### デバッグ例(Debug Example)
次の例は、可変数の引数を受け入れる強力な`debug`コマンドを実装しています。

```nim
# Nim構文ツリーを使用するには、`` macros``モジュールで定義されているAPIが必要です。
import macros

macro debug(args: varargs[untyped]): untyped =
  # `args`は、マクロの引数に対するASTをそれぞれ含む` NimNode`値のコレクションです。
  # マクロは常に `NimNode`を返さなければなりません。
  # 種類が `nnkStmtList`のノードは、このユースケースに適しています。
  result = nnkStmtList.newTree()
  # このマクロに渡される引数を繰り返し処理します:
  for n in args:
    # 式を書き込む呼び出しをステートメントリストに追加します;
    # `toStrLit`はASTをその文字列表現に変換します:
    result.add newCall("write", newIdentNode("stdout"), newLit(n.repr))
    # ": "を記述する呼び出しをステートメントリストに追加します:
    result.add newCall("write", newIdentNode("stdout"), newLit(": "))
    # 式の値を書き込む呼び出しをステートメントリストに追加します:
    result.add newCall("writeLine", newIdentNode("stdout"), n)

var
  a: array[0..10, int]
  x = "some string"
a[0] = 42
a[1] = 45

debug(a[0], a[1], x)
```

このマクロ呼び出しは次のように展開されます。


```nim
write(stdout, "a[0]")
write(stdout, ": ")
writeLine(stdout, a[0])

write(stdout, "a[1]")
write(stdout, ": ")
writeLine(stdout, a[1])

write(stdout, "x")
write(stdout, ": ")
writeLine(stdout, x)
```

`varargs`パラメーターに渡される引数は、配列コンストラクター式にラップされます。
これが、`debug`がすべての`n`の子に対して繰り返される理由です。

### BindSym
上記の`debug`マクロは、`write`,`writeLine`および`stdout`がシステムモジュールで宣言されているという事実に依存しているため、インスタンス化コンテキストで表示されます。
バインドされていない識別子を使用する代わりに、バインドされた識別子（別名、シンボル）を使用する方法があります。そのために、`bindSym`ビルトインを使用できます。

```nim
import macros

macro debug(n: varargs[typed]): untyped =
  result = newNimNode(nnkStmtList, n)
  for x in n:
    # we can bind symbols in scope via 'bindSym':
    add(result, newCall(bindSym"write", bindSym"stdout", toStrLit(x)))
    add(result, newCall(bindSym"write", bindSym"stdout", newStrLitNode(": ")))
    add(result, newCall(bindSym"writeLine", bindSym"stdout", x))

var
  a: array[0..10, int]
  x = "some string"
a[0] = 42
a[1] = 45

debug(a[0], a[1], x)
```

このマクロ呼び出しは次のように展開されます。

```nim
write(stdout, "a[0]")
write(stdout, ": ")
writeLine(stdout, a[0])

write(stdout, "a[1]")
write(stdout, ": ")
writeLine(stdout, a[1])

write(stdout, "x")
write(stdout, ": ")
writeLine(stdout, x)
```

ただし、シンボル`write`,`writeLine`および`stdout`は既にバインドされており、再度検索されません。
例が示すように、`bindSym`はオーバーロードされたシンボルを暗黙的に処理します。

### Case-Ofマクロ(Case-Of Macro)
Nimでは、すべてのブランチがマクロ実装に渡されて処理されるという違いがあるだけで、case-of式の構文を持つマクロを持つことができます。
その後、マクロの実装により、分岐を有効なNimステートメントに変換します。
次の例は、この機能を字句アナライザに使用する方法を示しています。

```nim
import macros

macro case_token(args: varargs[untyped]): untyped =
  echo args.treeRepr
  # creates a lexical analyzer from regular expressions
  # ... (implementation is an exercise for the reader ;-)
  discard

case_token: # this colon tells the parser it is a macro statement
of r"[A-Za-z_]+[A-Za-z_0-9]*":
  return tkIdentifier
of r"0-9+":
  return tkInteger
of r"[\+\-\*\?]+":
  return tkOperator
else:
  return tkUnknown
```

スタイルに関する注意：コードを読みやすくするために、要求を満たす中で最も強力でないプログラミングテクニックを使用することをお勧めします。
したがって、「チェックリスト」は次のとおりです。

- 可能であれば、通常のproc/iteratorを使用します
- Else：可能であれば、ジェネリックproc/iteratorを使用します
- Else：可能であれば、テンプレートを使用します。
- Else：マクロを使用します。

### プラグマとしてのマクロ(Macros as pragmas)
ルーチン全体（プロシージャ、イテレータなど）を、プラグマ表記を介してテンプレートまたはマクロに渡すこともできます。

```nim
template m(s: untyped) = discard

proc p() {.m.} = discard
```

これは、次の単純な構文変換です。

```nim
template m(s: untyped) = discard

m:
  proc p() = discard
```

### Forループマクロ(For Loop Macro)
特別な型`system.ForLoopStmt`の式を唯一の入力パラメーターとして使用するマクロは、`for`ループ全体を書き換えることができます。

```nim
import macros
{.experimental: "forLoopMacros".}

macro enumerate(x: ForLoopStmt): untyped =
  expectKind x, nnkForStmt
  # 最初のforループ変数を取り除き、整数カウンターとして使用します:
  result = newStmtList()
  result.add newVarStmt(x[0], newLit(0))
  var body = x[^1]
  if body.kind != nnkStmtList:
    body = newTree(nnkStmtList, body)
  body.add newCall(bindSym"inc", x[0])
  var newFor = newTree(nnkForStmt)
  for i in 1..x.len-3:
    newFor.add x[i]
  # enumerate(X)を 'X'に変換します
  newFor.add x[^2][1]
  newFor.add body
  result.add newFor
  # マクロ全体をブロックでラップして、新しいスコープを作成します
  result = quote do:
    block: `result`

for a, b in enumerate(items([1, 2, 3])):
  echo a, " ", b

# マクロをブロックにラップせずに、再定義エラーを避けるために、
# ここで「a」と「b」に異なる名前を選択する必要があります
for a, b in enumerate([1, 2, 3, 5]):
  echo a, " ", b
```

現在、forループマクロは`{.experimental: "forLoopMacros".}`を使用して明示的に有効にする必要があります。

## 特別な型(Special Types)

### static[T]
その名前が示すように、静的パラメーターは定数式でなければなりません。

```nim
proc precompiledRegex(pattern: static string): RegEx =
  var res {.global.} = re(pattern)
  return res

precompiledRegex("/d+") # Replaces the call with a precompiled
                        # regex, stored in a global variable

precompiledRegex(paramStr(1)) # Error, command-line options
                              # are not constant expressions
```

コード生成のため、すべての静的パラメーターはジェネリックパラメーターとして扱われます。
procは、指定された一意の値（または値の組み合わせ）ごとに個別にコンパイルされます。

静的パラメータは、ジェネリック型のシグネチャにも表示できます。

```nim
type
  Matrix[M,N: static int; T: Number] = array[0..(M*N - 1), T]
    # Note how `Number` is just a type constraint here, while
    # `static int` requires us to supply an int value
  
  AffineTransform2D[T] = Matrix[3, 3, T]
  AffineTransform3D[T] = Matrix[4, 4, T]

var m1: AffineTransform3D[float]  # OK
var m2: AffineTransform2D[string] # Error, `string` is not a `Number`
```

`static T`は、基礎となるジェネリック型`static [T]`の構文上の利便性にすぎないことに注意してください。
型パラメーターを省略して、すべての定数式の型クラスを取得できます。
`static`を別の型クラスでインスタンス化することにより、より具体的な型クラスを作成できます。

対応する`static`型に強制することで、コンパイル時に定数式として式を評価することができます。

```nim
import math

echo static(fac(5)), " ", static[bool](16.isPowerOfTwo)
```

コンパイラーは、式の評価の失敗または型の不一致エラーの可能性を報告します。

### typedesc[T]
多くのコンテキストで、Nimでは型の名前を通常の値として扱うことができます。
これらの値はコンパイル段階でのみ存在しますが、すべての値には型が必要であるため、`typedesc`は特別な型と見なされます。

`typedesc`はジェネリック型のように機能します。
たとえば、シンボル`int`の型は`typedesc[int]`です。
通常のジェネリック型と同様に、ジェネリックパラメータが省略されると、`typedesc`はすべての型の型クラスを示します。
構文上の利便性として、`typedesc`を修飾子として使用することもできます。

`typedesc`パラメータを備えたProcは、暗黙的にジェネリックと見なされます。
これらは、提供された型の一意の組み合わせごとにインスタンス化され、procの本体内で、各パラメーターの名前はバインドされた具象型を参照します。

```nim
proc new(T: typedesc): ref T =
  echo "allocating ", T.name
  new(result)

var n = Node.new
var tree = new(BinaryTree[int])
```

複数の型パラメーターが存在する場合、それらは異なる型に自由にバインドします。
bind-onceの動作を強制するために、明示的なジェネリックパラメーターを使用できます。

```nim
proc acceptOnlyTypePairs[T, U](A, B: typedesc[T]; C, D: typedesc[U])
```

バインドされると、procシグネチャの残りの部分に型paramsを表示できます。

```nim
template declareVariableWithType(T: typedesc, value: T) =
  var x: T = value

declareVariableWithType int, 42
```

型パラメーターと一致する型のセットを制約することにより、オーバーロードの解決にさらに影響を与えることができます。
これは実際には、テンプレートを介して型に属性をアタッチするために機能します。制約は、具象型または型クラスにすることができます。

```nim
template maxval(T: typedesc[int]): int = high(int)
template maxval(T: typedesc[float]): float = Inf

var i = int.maxval
var f = float.maxval
when false:
  var s = string.maxval # error, maxval is not implemented for string

template isNumber(t: typedesc[object]): string = "Don't think so."
template isNumber(t: typedesc[SomeInteger]): string = "Yes!"
template isNumber(t: typedesc[SomeFloat]): string = "Maybe, could be NaN."

echo "is int a number? ", isNumber(int)
echo "is float a number? ", isNumber(float)
echo "is RootObj a number? ", isNumber(RootObj)
```

マクロは一般的にインスタンス化されないという違いがありますが、`typedesc`を渡すことはほとんど同じです。
type expressionは他のすべてと同様に、`NimNode`としてマクロに単に渡されます。

```nim
import macros

macro forwardType(arg: typedesc): typedesc =
  # ``arg`` is of type ``NimNode``
  let tmp: NimNode = arg
  result = tmp

var tmp: forwardType(int)
```

### typeof operator
注：`typeof(x)`は歴史的な理由から`type(x)`と書くこともできますが、type(x)`は推奨されません。

式の型を取得するには、その式から`typeof`値を作成します（他の多くの言語では、これはtypeof演算子として知られています）。

```nim
var x = 0
var y: typeof(x) # y has type int
```

`typeof`を使用してproc/iterator/converterを呼び出す`c(x)`（`X`は空の可能性のある引数リストを表します）の結果の型を決定する場合、
`c`がイテレーターである解釈が他の解釈より優先されますが、`typeof`の2番目の引数として`typeOfProc`を渡すことにより、動作を変更できます。

```nim
iterator split(s: string): string = discard
proc split(s: string): seq[string] = discard

# since an iterator is the preferred interpretation, `y` has the type ``string``:
assert typeof("a b c".split) is string

assert typeof("a b c".split, typeOfProc) is seq[string]
```

## モジュール(Modules)
Nimは、モジュールコンセプトによってプログラムを複数の部分に分割することをサポートしています。
各モジュールは、独自のファイルに存在する必要があり、独自の名前空間を持っています。
モジュールは、情報の隠蔽と分割コンパイルを可能にします。
モジュールは、importステートメントによって別のモジュールのシンボルにアクセスできます。
再帰的なモジュールの依存関係は許可されますが、ちょっと微妙です。
アスタリスク(`*`)でマークされた最上位のシンボルのみがエクスポートされます。
有効なモジュール名は、有効なNim識別子のみです（したがって、ファイル名は`identifier.nim`です）。

モジュールをコンパイルするためのアルゴリズムは次のとおりです。

- importステートメントを再帰的にたどって、通常どおりモジュール全体をコンパイルします。
- サイクルがある場合は、既に解析されたシンボルのみをインポートします（エクスポートされます）。不明な識別子が発生した場合は中止します。

これは例によって最もよく説明されます：

```nim
# Module A
type
  T1* = int  # Module A は型``T1``をexportします
import B     # コンパイラはBのパースを開始します

proc main() =
  var i = p(3) # Bは既に完全に解析されているため機能します

main()
```

```nim
# Module B
import A  # Aはここではパースされません!
          # Aの既知のシンボルのみインポートされます。

proc p*(x: A.T1): A.T1 =
  # T1は既にAのインターフェースシンボルテーブルに追加されているため、
  # このプロシージャーは機能します。
  result = x + 1
```

#### Import statement
`import`ステートメントの後にモジュール名のリストを続けることができます。
また、単一のモジュール名の後に`except`リストを続けて、いくつかのシンボルがインポートされないようにすることができます。

```nim
import strutils except `%`, toUpperAscii

# doesn't work then:
echo "$1" % "abc".toUpperAscii
```

`except`リストが実際にモジュールからエクスポートされているかどうかはチェックされません。
この機能により、これらの識別子をエクスポートしない古いバージョンのモジュールに対してコンパイルできます。

#### Include statement
`include`ステートメントは、モジュールのインポートとは根本的に異なることを行います。
ファイルの内容を含めるだけです。`include`ステートメントは、大きなモジュールをいくつかのファイルに分割するのに役立ちます。

```nim
include fileA, fileB, fileC
```

#### Module names in imports
モジュールエイリアスは、`as`キーワードを介して導入できます。

```nim
import strutils as su, sequtils as qu

echo su.format("$1", "lalelu")
```

元のモジュール名にはアクセスできません。
`path/to/module`または`"path/to/module"`という表記を使用して、サブディレクトリ内のモジュールを参照できます。

```nim
import lib/pure/os, "lib/pure/times"
```

モジュール名は`strutil`であって`lib/pure/strutils`ではなく、以下のようには**できない**ことに注意して下さい。

```nim
import lib/pure/strutils
echo lib/pure/strutils.toUpperAscii("abc") # 無効
```

同様に名前は既に`strutil`であるため、以下は意味がありません。

```nim
import lib/pure/strutils as strutils
```

#### ディレクトリからの一括インポート(Collective imports from a directory)
`import dir / [moduleA, moduleB]`構文を使用して、同じディレクトリから複数のモジュールをインポートできます。

パス名は、構文的にはNim識別子または文字列リテラルです。
パス名が有効なNim識別子でない場合は、文字列リテラルである必要があります。

```nim
import "gfx/3d/somemodule" # '3d'は有効なNim識別子ではないため、引用符で囲みます
```

#### 疑似インポート/インクルードパス(Pseudo import/include paths)
ディレクトリは、いわゆる「擬似ディレクトリ」にすることもできます。
同じパスを持つモジュールが複数ある場合、それらを使用してあいまいさを回避できます。

There are two pseudo directories:

1. `std`:`std`疑似ディレクトリは、Nimの標準ライブラリの抽象的な場所です。たとえば、構文`import std / strutils`は、標準ライブラリのstrutilsモジュールを明確に参照するために使用されます。
1. `pkg`:`pkg`疑似ディレクトリは、Nimbleパッケージを明確に参照するために使用されます。
ただし、このドキュメントの範囲外の技術的な詳細については、そのセマンティクスは次のとおりです。
検索パスを使用してモジュール名を検索しますが、標準ライブラリの場所は無視します。つまり、`std`の反対です。

#### From import statement
`from`ステートメントの後、モジュール名の後に`import`を続けて、明示的な完全修飾なしで使用したいシンボルをリストします。

```nim
from strutils import `%`

echo "$1" % "abc"
# always possible: full qualification:
echo strutils.replace("abc", "a", "z")
```

モジュールをインポートしたいが、モジュール内のすべてのシンボルへの完全修飾アクセスを強制したい場合は`from module import nil`も使用可能です。

#### Export statement
クライアントモジュールがモジュールの依存関係をインポートする必要がないように、`export`ステートメントをシンボル転送に使用できます。

```nim
# module B
type MyObject* = object
```

```nim
# module A
import B
export B.MyObject

proc `$`*(x: MyObject): string = "my object"
```

```nim
# module C
import A

# B.MyObject has been imported implicitly here:
var x: MyObject
echo $x
```

エクスポートされたシンボルが別のモジュールである場合、そのすべての定義が転送されます。
`except`リストを使用して、一部のシンボルを除外できます。

エクスポートするときは、モジュール名のみを指定する必要があることに注意してください。

```nim
import foo/bar/baz
export baz
```

### スコープルール(Scope rules)
識別子は、宣言の時点から、宣言が発生したブロックの終わりまで有効です。
識別子が既知となるのは、識別子の範囲内です。識別子の正確なスコープは、宣言された方法によって異なります。

#### ブロックスコープ(Block scope)
ブロックの宣言部分で宣言された変数のスコープは、宣言のポイントからブロックの終わりまで有効です。
ブロックに識別子が再宣言される2番目のブロックが含まれている場合、このブロック内では2番目の宣言が有効になります。
内部ブロックを離れると、最初の宣言が再び有効になります。
プロシージャまたはイテレータのオーバーロードの目的で有効な場合を除き、識別子を同じブロックで再定義することはできません。

#### タプルまたはオブジェクトのスコープ(Tuple or object scope)
タプルまたはオブジェクト定義内のフィールド識別子は、次の場所で有効です。

- タプル/オブジェクト定義の終わりまで
- 指定されたタプル/オブジェクトタイプの変数のフィールド指定子
- オブジェクト型のすべての子孫型

#### モジュールスコープ(Module scope)
モジュールのすべての識別子は、宣言の時点からモジュールの終わりまで有効です。
間接的に依存するモジュールの識別子は利用できません。
システムモジュールは、自動的にすべてのモジュールにインポートされます。

モジュールが2つの異なるモジュールによって識別子をインポートする場合、オーバーロード解決が行われるオーバーロードプロシージャまたはイテレータでない限り、識別子の各occurrenceを修飾する必要があります。

```nim
# Module A
var x*: string
```

```nim
# Module B
var x*: int
```

```nim
# Module C
import A, B
write(stdout, x) # エラー: x はあいまい
write(stdout, A.x) # 正常: 修飾されている

var x = 4
write(stdout, x) # あいまいでない: モジュールCのxが使用される
```

#### コードの並べ替え(Code reordering)
注：コードの並べ替えは実​​験的なものであり、`{.experimental.}`プラグマで有効にする必要があります。

コードの並べ替え機能は、プロシージャ、テンプレート、マクロ定義をトップレベルのスコープでの変数宣言と初期化とともに暗黙的に並べ替えることができるため、プログラマーは定義を正しく並べ替えたり、
前方宣言を強制させられる心配をせずに済みます。

例：

```nim
{.experimental: "codeReordering".}

proc foo(x: int) =
  bar(x)

proc bar(x: int) =
  echo(x)

foo(10)
```

変数も並べ替えることができます。
初期化される変数（宣言と代入が同じ行でされる変数）は初期化ステートメント全体を並べ替えることができます。。
トップレベルで実行されるコードに注意してください。

```nim
{.experimental: "codeReordering".}

proc a() =
  echo(foo)

var foo = 5

a() # outputs: "5"
```

並べ替えは、最上位スコープのシンボルに対してのみ機能することに注意することが重要です。
したがって、以下はコンパイルに失敗します。

```nim
{.experimental: "codeReordering".}

proc a() =
  b()
  proc b() =
    echo("Hello!")

a() #エラー
```

## コンパイラメッセージ(Compiler Messages)
Nimコンパイラーは、ヒント、警告、エラーメッセージなど、さまざまな種類のメッセージを出力します。コンパイラが静的エラーを検出すると、エラーメッセージが表示されます。

## プラグマ(Pragmas)
プラグマは、膨大な数の新しいキーワードを導入することなく、コンパイラに追加情報/
コマンドを提供するNimのメソッドです。プラグマは、セマンティックチェック中にその場で
処理されます。プラグマは特別な `{.` と `.}` の中括弧で囲まれています。プラグマは、
機能にアクセスするためのより適切な構文が利用可能になる前に、言語機能を試す最初
の実装としてもよく使用されます。

### 非推奨プラグマ(deprecated pragma)
deprecatedプラグマはシンボルに非推奨としてのマークを付与します。

```nim
proc p() {.deprecated.}
var x {.deprecated.}: char
```

このプラグマは、開発者に伝えるためにオプションの警告文字列を取り込むこともできます。

```nim
proc thing(x: bool) {.deprecated: "thongを代わりに使用してください".}
```

### 副作用なしプラグマ(noSideEffect pragma)
`noSideEffect` プラグマは、procやiteratorが副作用を持たないことをマークするために使用されます。
これは、procやiteratorがパラメーターから到達可能な場所のみを変更し、戻り値が引数のみに依存することを意味します。
`var T` または `ref T` または `ptr T` のタイプを持つパラメーターがない場合、これは副作用がないことを意味します。
コンパイラがこれを検証できない場合にprocまたはiteratorに副作用なしのマークを付与すると静的なエラーとなります。

特別なセマンティックルールとして、組み込みの [`debugEcho`](https://nim-lang.org/docs/system.html#debugEcho%2Cvarargs%5Btyped%2C%5D) は副作用がないように見せかけるため、
`noSideEffect` としてマークされたルーチンのデバッグに使用できます。

`func` は、副作用のないprocの糖衣構文です。

```nim
func `+` (x, y: int): int
```

コンパイラの副作用解析を上書きするには、 `{.noSideEffect.}` プラグマブロックを使用できます。

```nim
func f() =
  {.noSideEffect.}:
    echo "test"
```

### コンパイルタイムプラグマ(compileTime pragma)
`compileTime` プラグマは、コンパイル時にのみ実行されるプロシージャまたは変数をマークするために使用されます。コードは生成されません。
コンパイル時プロシージャは、マクロのヘルパーとして役立ちます。
言語のバージョン0.12.0以降、パラメータータイプ内で `system.NimNode` を使用するプロシージャは、 `compileTime` で暗黙的に宣言されます。

```nim
proc astHelper(n: NimNode): NimNode =
  result = n
```

上のコードは以下と同じです。

```nim
proc astHelper(n: NimNode): NimNode {.compileTime.} =
  result = n
```

compileTime変数は実行時にも利用できます。
これにより、コンパイル時に代入されるが（lookup tablesなど）、
実行時にアクセスされる変数の特定のイディオムが簡素化されます。

```nim
import macros

var nameToProc {.compileTime.}: seq[(string, proc (): string {.nimcall.})]

macro registerProc(p: untyped): untyped =
  result = newTree(nnkStmtList, p)
  
  let procName = p[0]
  let procNameAsStr = $p[0]
  result.add quote do:
    nameToProc.add((`procNameAsStr`, `procName`))

proc foo: string {.registerProc.} = "foo"
proc bar: string {.registerProc.} = "bar"
proc baz: string {.registerProc.} = "baz"

doAssert nameToProc[2][1]() == "baz"
```

### 戻り値なしプラグマ(noReturn pragma)
`noreturn` プラグマはプロシージャに戻り値がないことをマークします。

### 非循環プラグマ(acyclic pragma)
`acyclic` プラグマは、型宣言に適用されます。非推奨であり、無視されます。

### ファイナルプラグマ(final pragma)
`final` のプラグマをオブジェクトタイプに使用して、継承できないことを指定できます。
継承は、既存のオブジェクトから（ `object of SuperType` 構文を介して）継承するオブジェクト、または `inheritable` としてマークされているオブジェクトでのみ使用できます。

### shallowプラグマ(shallow pragma)
`shallow` プラグマは、型のセマンティクスに影響します。
コンパイラーは、浅いコピー(shallow copy)の作成を許可します。
これは重大な意味上の問題を引き起こし、メモリ安全性を破壊する可能性があります！
ただし、Nimのセマンティクスではシーケンスと文字列の深いコピー(deep copy)が必要なため、割り当てを大幅に高速化できます。
これは、特にシーケンスを使用してツリー構造を構築する場合、高コストになる可能性があります。

```nim
type
  NodeKind = enum nkLeaf, nkInner
  Node {.shallow.} = object
    case kind: NodeKind
    of nkLeaf:
      strVal: string
    of nkInner:
      children: seq[Node]
```

### 純粋プラグマ(pure pragma)
オブジェクト型は、実行時の型識別に使用される型フィールドが省略されるように、`pure` プラグマでマークできます。
これは、他のコンパイル言語とのバイナリ互換性のために必要でした。

列挙型は、`pure` としてマークできます。次に、そのフィールドにアクセスするには、常に列挙型名を省略しない完全な修飾が必要です。

### アセンブリスタックフレームなしプラグマ(asmNoStackFrame pragma)
procには、 `asmNoStackFrame` プラグマを使用して、procのスタックフレームを生成しないようコンパイラーに指示できます。
また、`return result;` などのexitステートメントは生成されず、
生成されたC関数は `__declspec(naked)`または `__attribute__((naked))` として宣言されます（使用されるCコンパイラに応じて）。

注：このプラグマは、アセンブラーステートメントのみで構成されるプロシージャでのみ使用してください。

### エラープラグマ(error pragma)
`error` プラグマは、指定された内容のエラーメッセージをコンパイラに出力させるために使用されます。
ただし、エラーが発生してもコンパイルは必ずしも中断しません。

`error` プラグマは、シンボル（イテレータやプロシージャなど）に注釈を付けるためにも使用できます。
シンボルを使用すると、静的エラーが発生します。
これは、オーバーロードと型変換が原因で一部の操作が有効であることを除外するのに特に役立ちます。

```nim
## ポインターではなく、基になるint値が比較されることを確認します。
proc `==`(x, y: ptr int): bool {.error.}
```

### 致命的なプラグマ(fatal pragma)
`fatal` プラグマは、指定された内容のエラーメッセージをコンパイラに出力させるために使用されます。
`error` プラグマとは対照的に、コンパイルはこのプラグマによって中止されることが保証されています。

例：

```nim
when not defined(objc):
  {.fatal: "このプログラムはobjcコマンドでコンパイルします！".}
```

### 警告プラグマ(warning pragma)
`warning` プラグマは、指定された内容の警告メッセージをコンパイラーに出力させるために使用されます。
警告の後にコンパイルが続行されます。

### ヒントプラグマ(hint pragma)
`hint` プラグマは、指定された内容のヒントメッセージをコンパイラに出力させるために使用されます。
ヒントの後にコンパイルが続行されます。

### 行プラグマ(line pragma)
`line` プラグマは、スタックバックトレースで見られるように、注釈付きステートメントの行情報に影響を与えるために使用できます。

```nim
template myassert*(cond: untyped, msg = "") =
  if not cond:
    # 'raise'ステートメントのランタイム行情報を変更
    {.line: instantiationInfo().}:
      raise newException(EAssertionFailed, msg)
```

`line` プラグマをパラメーターとともに使用する場合、パラメーターは `tuple[filename: string, line: int]` である必要があります。
パラメーターなしで使用する場合は、 `system.InstantiationInfo()` が使用されます。

### linearScanEndプラグマ(linearScanEnd pragma)
`linearScanEnd` プラグマを使用して、Nimのcaseステートメントをコンパイルする方法をコンパイラーに指示できます。
構文的には、ステートメントとして使用する必要があります。

```nim
case myInt
of 0:
  echo "most common case"
of 1:
  {.linearScanEnd.}
  echo "second most common case"
of 2: echo "unlikely: use branch table"
else: echo "unlikely too: use branch table for ", myInt
```

この例では、ケース分岐0と1は他のケースよりもはるかに一般的です。
したがって、生成されたアセンブラコードは、これらの値を最初にテストする必要があります。
これにより、CPUの分岐予測が成功する可能性が高くなります（高価なCPUパイプラインストールを回避します）。
他のケースは、O（1）オーバーヘッドのジャンプテーブルに入れられる可能性がありますが、パイプラインが停止する可能性が非常に高くなります。

`linearScanEnd` プラグマは、線形スキャンでテストする必要がある最後の分岐に配置する必要があります。 case文全体の最後の分岐に配置すると、case文全体で線形スキャンが使用されます。

### computedGotoプラグマ(computedGoto pragma)
`computedGoto` プラグマを使用して、 `while true` ステートメントでNimケースをコンパイルする方法をコンパイラーに指示できます。
構文的には、ループ内のステートメントとして使用する必要があります。

```nim
type
  MyEnum = enum
    enumA, enumB, enumC, enumD, enumE

proc vm() =
  var instructions: array[0..100, MyEnum]
  instructions[2] = enumC
  instructions[3] = enumD
  instructions[4] = enumA
  instructions[5] = enumD
  instructions[6] = enumC
  instructions[7] = enumA
  instructions[8] = enumB
  
  instructions[12] = enumE
  var pc = 0
  while true:
    {.computedGoto.}
    let instr = instructions[pc]
    case instr
    of enumA:
      echo "yeah A"
    of enumC, enumD:
      echo "yeah CD"
    of enumB:
      echo "yeah B"
    of enumE:
      break
    inc(pc)

vm()
```

例が示すように、 `computedGoto` はインタープリターに最も役立ちます。
基になるバックエンド（Cコンパイラ）が計算されたgoto拡張機能をサポートしない場合、
プラグマは単に無視されます。

### unrollプラグマ(unroll pragma)
`unroll` プラグマは、実行効率のためにforループまたはwhileループを展開する必要があることをコンパイラーに伝えるために使用できます。

```nim
proc searchChar(s: string, c: char): int =
  for i in 0 .. s.high:
    {.unroll: 4.}
    if s[i] == c: return i
  result = -1
```

上記の例では、検索ループは係数4によって展開されます。
展開係数も省略できます。
その後、コンパイラーは適切なアンロール係数を選択します。

注：現在、コンパイラはこのプラグマを認識しますが、無視します。

### 即時プラグマ(immediate pragma)
`immediate` プラグマは廃止されました。
[typedパラメータとuntypedパラメータ](#typedパラメータとuntypedパラメータ)を参照してください。

### コンパイルオプションプラグマ(compilation option pragmas)
ここにリストされているプラグマは、proc / method / converterのコード生成オプションをオーバーライドするために使用できます。

| pragma | allowed values | description |
| ------ | -------------- | ----------- |
| checks | on&#124;off | すべてのランタイムチェックのコード生成をオンまたはオフにします。 |
| boundChecks | on&#124;off | 配列バインドチェックのコード生成をオンまたはオフにします。 |
| overflowChecks | on&#124;off | オーバーフローまたはアンダーフローチェックのコード生成をオンまたはオフにします。 |
| nilChecks | on&#124;off | nilポインターチェックのコード生成をオンまたはオフにします。 |
| assertions | on&#124;off | アサーションのコード生成をオンまたはオフにします。 |
| warnings | on&#124;off | コンパイラの警告メッセージをオンまたはオフにします。 |
| hints | on&#124;off | コンパイラのヒントメッセージをオンまたはオフにします。 |
| optimization | none&#124;speed&#124;size | コードの速度またはサイズを最適化するか、最適化を無効にします。 |
| patterns | on&#124;off | テンプレート/マクロの書き換え項をオンまたはオフにします。 |
| callconv | cdecl&#124;... | 後続のすべてのプロシージャ（およびプロシージャタイプ）のデフォルトの呼び出し規約を指定します。 |

例：

```nim
{.checks: off, optimization: speed.}
# ランタイムチェックなしでコンパイルし、速度を最適化する
```

### pushとpopプラグマ(push and pop pragmas)
`push` / `pop`プラグマは、optionディレクティブに非常に似ていますが、一時的に設定をオーバーライドするために使用されます。

例：

```nim
{.push checks: off.}
# 速度が重要なため、このセクションの実行時チェックなしでコンパイル
# ... なんらかのコード ...
{.pop.} # 元の設定を復元
```

`push` / `pop` は、いくつかの標準ライブラリプラグマのオン/オフを切り替えることができます。

例：

```nim
{.push inline.}
proc thisIsInlined(): int = 42
func willBeInlined(): float = 42.0
{.pop.}
proc notInlined(): int = 9

{.push discardable, boundChecks: off, compileTime, noSideEffect, experimental.}
template example(): string = "https://nim-lang.org"
{.pop.}

{.push deprecated, hint[LineTooLong]: off, used, stackTrace: off.}
proc sample(): bool = true
{.pop.}
```

サードパーティのプラグマの場合、その実装に依存しますが、同じ構文を使用します。

### レジスタプラグマ(register pragma)
`register` プラグマは変数専用です。
変数を `register` として宣言し、アクセスを高速化するために変数をハードウェアレジスタに配置する必要があるというヒントをコンパイラに提供します。
Cコンパイラは通常これを無視しますが、それには十分な理由があります。
多くの場合、Cコンパイラはそれなしでより良い仕事をします。

ただし、非常に特殊な場合（たとえば、バイトコードインタープリターのディスパッチループ）には、利点があります。

### グローバルプラグマ(global pragma)
`global` プラグマをproc内の変数に適用すると、グローバルな場所に保存し、プログラムの起動時に一度初期化するようにコンパイラに指示できます。

```nim
proc isHexNumber(s: string): bool =
  var pattern {.global.} = re"[0-9a-fA-F]+"
  result = s.match(pattern)
```

汎用プロシージャ内で使用される場合、プロシージャのインスタンス化ごとに個別の一意のグローバル変数が作成されます。
モジュール内で作成されたグローバル変数の初期化の順序は定義されていませんが、それらはすべて、元のモジュールの最上位変数の後、それをインポートするモジュールの変数の前に初期化されます。

### pragmaプラグマ(pragma pragma)
`pragma` プラグマは、ユーザー定義のプラグマを宣言するために使用できます。
Nimのテンプレートとマクロはプラグマに影響しないため、これは便利です。
ユーザー定義のプラグマは、他のすべてのシンボルとは異なるモジュール全体のスコープ内にあります。
モジュールからインポートすることはできません。

例：

```nim
when appType == "lib":
  {.pragma: rtl, exportc, dynlib, cdecl.}
else:
  {.pragma: rtl, importc, dynlib: "client.dll", cdecl.}

proc p*(a, b: int): int {.rtl.} =
  result = a+b
```

この例では、ダイナミックライブラリからシンボルをインポートするか、ダイナミックライブラリ生成用にシンボルをエクスポートする、 `rtl` hという名前の新しいプラグマが導入されています。

### 特性のメッセージを無効化(Disabling certain messages)
Nimは、ユーザーを困らせる可能性のある警告とヒント（「行が長すぎます」）を生成します。
特定のメッセージを無効にするメカニズムが提供されています。
各ヒントおよび警告メッセージには、括弧内に記号が含まれています。
これは、有効または無効にするために使用できるメッセージの識別子です。

```nim
{.hint[LineTooLong]: off.} # 長過ぎる行のヒントをOFFにする
```

多くの場合、これはすべての警告を一度に無効にするよりも優れています。

### 使用中プラグマ(used pragma)
Nimは、エクスポートも使用もされていないシンボルに対して警告を生成します。
`used` プラグマをシンボルに添付して、この警告を抑制することができます。
これは、シンボルがマクロによって生成された場合に特に便利です。

```nim
template implementArithOps(T) =
  proc echoAdd(a, b: T) {.used.} =
    echo a + b
  proc echoSub(a, b: T) {.used.} =
    echo a - b

# 未使用の 'echoSub' に対して警告されません
implementArithOps(int)
echoAdd 3, 5
```

`used` は、モジュールを「使用済み」としてマークする最上位ステートメントとしても使用できます。
これにより、「未使用のインポート」警告が防止されます。

```nim
# module: debughelper.nim
when defined(nimHasUsed):
  # 'import debughelper'はデバッグに非常に役立つので、
  # 現在使用されていなくても、Nimはそのインポートに対して警告を生成しないはずです。
  {.used.}
```

### 実験的なプラグマ(experimental pragma)
`experimental` プラグマは、実験的な言語機能を有効にします。
具体的な機能に応じて、これはその機能が他の点では安定したリリースに対して不安定すぎるか、
機能の将来が不確実である（いつでも削除される可能性がある）ことを意味します。

例：

```nim
{.experimental: "parallel".}

proc useParallel() =
  parallel:
    for i in 0..4:
      echo "echo in parallel"
```

最上位のステートメントとして、`experimental` プラグマは、有効になっている残りのモジュールの機能を有効にします。
これは、モジュールスコープを超えるマクロおよび一般的なインスタンス化にとって問題です。
現在、これらの使用法は `.push / pop` 環境に配置する必要があります。

```nim
# client.nim
proc useParallel*[T](unused: T) =
  # use a generic T here to show the problem.
  {.push experimental: "parallel".}
  parallel:
    for i in 0..4:
      echo "echo in parallel"
  
  {.pop.}
```

```nim
import client
useParallel(1)
```

## 実装固有のプラグマ(Implementation Specific Pragmas)
このセクションでは、現在のNim実装がサポートしているが、言語仕様の一部として見なされるべきではない追加のプラグマについて説明します。

### ビットサイズプラグマ(Bitsize pragma)
`bitsize`のプラグマは、オブジェクトフィールドのメンバーのためです。C / C ++でビットフィールドとしてフィールドを宣言します。

```nim
type
  mybitfield = object
    flag {.bitsize:1.}: cuint
```

これは次を生成します。

```nim
struct mybitfield {
  unsigned int flag:1;
};
```

### 揮発性プラグマ(Volatile pragma)
揮発性のプラグマは、変数だけのためです。
C/C ++で意味するものは何でも、変数を`volatile`として宣言します（そのセマンティクスはC/C ++で十分に定義されていません）。

注：このプラグマは、LLVMバックエンドには存在しません。

### NoDeclプラグマ(NoDecl pragma)
`noDecl`のプラグマは、ほぼすべての記号（変数、PROC、型など）に適用でき、Cとの相互運用性のために、時には有用です。
それは、Cコードでシンボルの宣言を生成しないことをNimに伝えます。例えば：

```nim
var
  EACCES {.importc, noDecl.}: cint # EACCESが変数であるふりをする、
                                   # Nimはその値を知らない
```

ただし、多くの場合、`header`プラグマの方が優れています。

注：これはLLVMバックエンドでは機能しません。

### ヘッダープラグマ(Header pragma)
`header`プラグマは`noDecl`プラグマに非常に似ています。
ほとんどすべてのシンボルに適用でき、宣言しないように指定し、代わりに生成コードに`#include`を含める必要があります。

```nim
type
  PFile {.importc: "FILE*", header: "<stdio.h>".} = distinct pointer
    # CのFILE* 型をインポートする。Nimはそれを新しいポインタ型として扱います。
```

`header`プラグマは常に文字列定数を期待しています。
文字列定数にはヘッダーファイルが含まれます。
Cの場合と同様に、システムヘッダーファイルは山括弧`<>`で囲まれています。
山括弧が指定されていない場合、Nimは生成されたCコードのヘッダーファイルを`""`で囲みます。

注：これはLLVMバックエンドでは機能しません。

### IncompleteStructプラグマ(IncompleteStruct pragma)
`incompleteStruct`のプラグマは、`sizeof`式で基になるCの`struct`を使用しないようコンパイラーに指示します。
```nim
type
  DIR* {.importc: "DIR", header: "<dirent.h>",
         pure, incompleteStruct.} = object
```

### コンパイルプラグマ(Compile pragma)
`compile`プラグマは、プロジェクトとC/C ++ソースファイルをコンパイルしてリンクするために使用することができます。

```nim
{.compile: "myfile.cpp".}
```

注：NimはSHA1チェックサムを計算し、ファイルが変更された場合にのみファイルを再コンパイルします。
`-f`コマンドラインオプションを使用して、ファイルを強制的に再コンパイルできます。

### リンクプラグマ(Link pragma)

`link`プラグマは、プロジェクトに追加のファイルをリンクするために使用することができます。

```nim
{.link: "myfile.o".}
```

### PassCプラグマ(PassC pragma)
`passC`プラグマを使用すると、コマンドラインスイッチ`--passC`を使用する場合と同様に、Cコンパイラに追加のパラメーターを渡すことができます。

```nim
{.passC: "-Wall -Werror".}
```

[システムモジュール](https://nim-lang.org/docs/system.html)の`gorge`を使用して、セマンティック解析中に実行される外部コマンドからパラメーターを埋め込むことができます。

```nim
{.passC: gorge("pkg-config --cflags sdl").}
```

### PassLプラグマ(PassL pragma)
`passL`プラグマを使用すると、コマンドラインスイッチ`--passL`を使用する場合と同様に、追加のパラメーターをリンカーに渡すことができます。

```nim
{.passL: "-lSDLmain -lSDL".}
```

[システムモジュール](https://nim-lang.org/docs/system.html)の`gorge`を使用して、セマンティック解析中に実行される外部コマンドからパラメーターを埋め込むことができます。

```nim
{.passL: gorge("pkg-config --libs sdl").}
```

###　エミットプラグマ(Emit pragma)
`emit`プラグマは直接コンパイラのコードジェネレータの出力に影響を与えるために使用することができます。
そのため、コードを他のコードジェネレーター/バックエンドに移植できなくなります。
その使用は非常に推奨されません！
ただし、C++またはObjective Cコードとのインターフェイスには非常に便利です。

例：

```nim
{.emit: """
static int cvariable = 420;
""".}

{.push stackTrace:off.}
proc embedsC() =
  var nimVar = 89
  # access Nim symbols within an emit section outside of string literals:
  # 文字列リテラルの外のemitセクション内のNimシンボルにアクセスします:
  {.emit: ["""fprintf(stdout, "%d\n", cvariable + (int)""", nimVar, ");"].}
{.pop.}

embedsC()
```

`nimbase.h`は、`nim c`と`nim cpp`の両方で動作する`extern "C"`コードに使用できる`NIM_EXTERNC C`マクロを定義します。
例：

```nim
proc foobar() {.importc:"$1".}
{.emit: """
#include <stdio.h>
NIM_EXTERNC
void fun(){}
""".}
```

後方互換性のために、emitステートメントへの引数が単一の文字列リテラルである場合、バッククォートを介してNimシンボルを参照できます。ただし、この使用法は非推奨です。

トップレベルのemitステートメントの場合、生成されたC/C++ファイルでコードを発行するセクションは、
接頭辞`/*TYPESECTION*/`または`/*VARSECTION*/`または`/*INCLUDESECTION*/`によって影響を受ける可能性があります。

```nim
{.emit: """/*TYPESECTION*/
struct Vector3 {
public:
  Vector3(): x(5) {}
  Vector3(float x_): x(x_) {}
  float x;
};
""".}

type Vector3 {.importcpp: "Vector3", nodecl} = object
  x: cfloat

proc constructVector3(a: cfloat): Vector3 {.importcpp: "Vector3(@)", nodecl}
```

### ImportCppプラグマ(ImportCpp pragma)
注：[c2nim](https://github.com/nim-lang/c2nim/blob/master/doc/c2nim.rst)はC ++の大規模なサブセットを解析でき、
`importcpp`プラグマパターン言語について認識しています。ここで説明されているすべての詳細を知る必要はありません。

[Cの`importc`プラグマ](#ImportcプラグマImportc-pragma)と同様に、`importcpp`プラグマを使用して、一般にC ++メソッドまたはC++シンボルをインポートできます。
生成されたコードは、構文を呼び出すC++メソッド`obj->method(arg)`を使用します。
`header`と`emit`プラグマを組み合わせることで、C++で記述されたライブラリとの雑なインターフェイスが可能になります。

```nim
# C++エンジンとのインターフェース方法の恐ろしい例 ... ;-)

{.link: "/usr/lib/libIrrlicht.so".}

{.emit: """
using namespace irr;
using namespace core;
using namespace scene;
using namespace video;
using namespace io;
using namespace gui;
""".}

const
  irr = "<irrlicht/irrlicht.h>"

type
  IrrlichtDeviceObj {.header: irr,
                      importcpp: "IrrlichtDevice".} = object
  IrrlichtDevice = ptr IrrlichtDeviceObj

proc createDevice(): IrrlichtDevice {.
  header: irr, importcpp: "createDevice(@)".}
proc run(device: IrrlichtDevice): bool {.
  header: irr, importcpp: "#.run(@)".}
```

これが機能するには、コンパイラーにC++（コマンド`cpp`）を生成するように指示する必要があります。
コンパイラがC++コードを発行するときに、条件付きシンボル`cpp`が定義されます。


## 外部関数インターフェース(Foreign function interface)

### Importcプラグマ(Importc pragma)

## スレッド(Threads)
スレッドサポートを有効にするには、コマンドラインスイッチ`--threads:on`を使用する必要があります。
`system`モジュールには、いくつかのスレッドプリミティブが含まれています。
低レベルのスレッドAPIについては、[threads](https://nim-lang.org/docs/threads.html)と[channels](https://nim-lang.org/docs/channels.html)モジュールを参照してください。
また、高レベルの並列処理構造も利用できます。詳細については、[spawn](https://nim-lang.org/docs/manual_experimental.html#parallel-amp-spawn)を参照してください。

Nimのスレッドのメモリモデルは、他の一般的なプログラミング言語(C, Pascal, Java)とはまったく異なります。
各スレッドには独自の（ガベージコレクション）ヒープがあり、メモリの共有はグローバル変数に制限されます。
これは、競合状態の防止に役立ちます。GCが他のスレッドを停止して参照を調べる必要がないため、GCの効率が大幅に向上します。

### スレッドプラグマ(Thread pragma)
新しい実行スレッドとして実行されるプロシージャは、読みやすくするために`thread`プラグマでマークする必要があります。
コンパイラは、ヒープ共有の制限の違反をチェックします。
この制限は、異なる（スレッドローカル）ヒープから割り当てられたメモリで構成されるデータ構造の構築が無効であることを意味します。

スレッドプロシージャは`createThread`または`spawn`に渡され、間接的に呼び出されます。したがって、`thread`プラグマは`procvar`を暗黙指定します。

### GCの安全性(GC safety)
proc `p`が直接的または間接的（GC安全でないprocの呼び出しを介して）にGCされるメモリ（`string`, `seq`, `ref`またはクロージャー）を含むグローバル変数にアクセスしない場合、`p`をGC安全なプロシージャーと呼びます。

gcsafe注釈を使用して、procをgcsafeとしてマークできます。
それ以外の場合、このプロパティはコンパイラーによって推測されます。
`noSideEffect`は`gcsafe`を意味することに注意してください。
スレッドを作成する唯一の方法は、`spawn`または`createThread`を使用することです。

呼び出されるprocは`var`パラメーターを使用してはならず、そのパラメーターのいずれにも`ref`型またはクロージャー型が含まれていてはなりません。
これにより、ヒープ共有の制限がなくなります。

Cからインポートされるルーチンは、常に`gcsafe`であると想定されます。
GCセーフチェックを無効にするために、`--threadAnalysis:off`コマンドラインスイッチを使用できます。
これは、古いコードから新しいスレッドモデルへの移植作業を容易にするための一時的な回避策です。

コンパイラのgcsafety解析をオーバーライドするために、`{.gcsafe.} `プラグマブロックを使用できます。

```nim
var
  someGlobal: string = "some string here"
  perThread {.threadvar.}: string

proc setPerThread() =
  {.gcsafe.}:
    deepCopy(perThread, someGlobal)
```

今後の方向性：

- GCされる共有メモリが提供されるかもしれません。

### Threadvarプラグマ(Threadvar pragma)
変数は`threadvar`プラグマでマークできます。
これにより、変数はスレッドローカル変数になります。
さらに、これは`global`プラグマのすべての効果が適用されます。

```nim
var checkpoints* {.threadvar.}: seq[string]
```

実装の制限により、スレッドローカル変数は`var`セクション内で初期化できません。
（すべてのスレッドローカル変数は、スレッドの作成時に複製される必要があります。）

### スレッドと例外(Threads and exceptions)
スレッドと例外の相互作用は簡単です。
あるスレッドで処理された例外が他のスレッドに影響を与えることはできません。
ただし、あるスレッドで未処理の例外が発生すると、プロセス全体が終了します！
