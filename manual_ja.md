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
ただし、実装はこれらのランタイムチェックを無効にする手段を提供します。詳細については、[Pragma](#Pragmas)セクションを参照してください。

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
- reference (pointer) types
- the FFI

これらの制限の一部または全ては今後、解除される可能性があります。

## Pragmas






