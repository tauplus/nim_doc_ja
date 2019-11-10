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
セット型は、セットの数学的な概念をモデル化します。
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
|`A == B`|セット等価性|
|`A <= B`|サブセット関係(AはBのサブセットまたは等価)|
|`A < B`|サブセット関係(AはBのサブセットであり等価でない)|
|`e in A`|セットメンバーシップ(Aは要素eを含む)|
|`e notin A`|Aは要素eを含まない|
|`contains(A, e)`|Aは要素eを含む|
|`card(A)`|Aの基数(Aの要素の数)|
|`incl(A, elem)`|`A = A + {elem}`と同じ|
|`excl(A, elem)`|`A = A - {elem}`と同じ|

### ビットフィールド(Bit fields)

### 変換可能な関係(Convertible relation)













## Pragmas






