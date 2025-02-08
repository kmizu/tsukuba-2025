## プログラミング言語作成概論

### 2025年02月08日（土）
### 筑波大学
###　ソフトウェアサイエンス特別講義A
### 株式会社ネクストビート　水島宏太

--

## 自己紹介

- 水島宏太（みずしまこうた） / @kmizu （けーみず） 
  - 筑波大学情報学類（現情報科学類）2002年度入学
  - 博士（工学）
- 株式会社ネクストビート　
  - テクノロジーエヴァンジェリスト
    - 技術広報的なこと
    - 社内エンジニア教育
  - 最近は大規模言語モデル（LLM）に興味津々
- プログラミング言語&構文解析マニア
  - 自作言語を趣味で作る
- Japan Scala Assocation監事

--

## 会社の宣伝

![Nextbeat](img/nextbeat.jpg)

--

自社Webサービスを色々やってます
- 子育て関係: KIDSNAプラットフォーム
  - 保育士バンク
  - KIDSNAシッター
- 地方創生: おもてなしHR
- グローバル領域: Hospitality Careers

--

## プログラミング言語のお話

--

## 世の中には多くのプログラミング言語がある

- 手続き型： C, BASIC, Pascal, Go, ...
- オブジェクト指向： C++, Java, C#, Ruby, Python, ...
- 関数型： OCaml, Haskell, Scala, Elixir, ...

※分類は暫定的なもの

--

**なんでこんなにたくさんのプログラミング言語があるの？**

--

## たくさんのプログラミング言語がある理由

- あることが得意な言語は別のことが不得意
  - C++は性能やリアルタイム性が必要な環境で有利
  - 安全性が重要な場面ではJavaに比べて不利

**どの言語を選んでもトレードオフが発生する **

- 「全部込み」の言語を作ればいいのでは？
  -  学習コストが異様に高くなる

**時代のニーズにあった、バランスの取れた言語設計が必要**

--

## プログラミング言語を作る前に

- 普通のプログラミング言語を作るのはハードルが高い
  - 言語処理系を作る上での「定番」アプローチの習得
  - 「構文」「型システム」「意味論」への理解

**単純な言語から始めるのが良い**

--

## 数式言語から始める

- いわゆる四則演算だけできるもの
- 具体例:
  - `1 + 2` → `3`
  - `1 - 2` → `-1`
  - `2 + 3 * 4` → `14`
    - 掛け算が優先
  - `(1 + 2 + 3 + 4) /  2` → `5`
- これらを解釈実行できるものを作る

--

## 数式の優先順位の話

- `2 + 3 * 4` → `2 + (3 * 4)` → `14`
  - 掛け算や割り算が足し算や引き算よりも優先
- `(1 + 2 + 3 + 4) /  2` → `5`
  - `()`によって優先順位を変更できる
- けっこうややこしい（構文解析の話）
  - 考えないで済むアプローチを取る
  - 抽象構文

--

## 抽象構文

- 本質的でない部分を除いた形での構文
  - 足し算が`+`であるとかは些細なこと
  - 優先順位も些細なこと
  - スペースをどう処理するのかも些細なこと

--

### 数式の抽象構文を考える

- スペースの処理や優先順位については捨象

```
式   = 式 + 式 // 加算
     | 式 - 式 // 減算
     | 式 * 式 // 乗算 
     | 式 / 式 // 除算
     | 整数
整数 = 0|[1-9][0-9]*
```


--

### 数式の抽象構文木 in Scala 3

- 抽象構文を「木」（Tree）の形で表現したもの

```js
enum Exp {
  case BinExp(op: String, lhs: Exp, rhs: Exp)
  case VInt(value: Int)
}
```

--

### 数式の抽象構文木を組み立てるin Scala 3

- 括弧は入らない（木の構造に埋め込まれている）

```scala
VInt(100) // 100
BinExp("+", VInt(100), VInt(200)) // 100 + 200
BinExp(
  "+",
  VInt(1), BinExp("*", VInt(2), VInt(3))
) // 1 + 2 * 3 == 1 + (2 * 3)
```

--

### 数式の抽象構文木を解釈してみる in Scala 3

```js
VInt(100) // → 100
BinExp('+', VInt(100), VInt(200)) // 100 + 200 → 300
BinExp(
  '+',
  VInt(1),
  BinExp('*', VInt(2), VInt(3))
) // 1 + 2 * 3 == 1 + (2 * 3) → 7
```

これらの具体例を一般化する = インタプリタの作成

--

### 数式インタプリタを作る

- 再帰的に自分自身を呼び出すだけ
- これで全部

```scala
def eval(exp: Exp): Int = exp match {
  case VInt(value)              => value
  case BinExp("+", lhs, rhs)    => eval(lhs) + eval(rhs)
  case BinExp("-", lhs, rhs)    => eval(lhs) - eval(rhs)
  case BinExp("*", lhs, rhs)    => eval(lhs) * eval(rhs)
  case BinExp("/", lhs, rhs)    => eval(lhs) / eval(rhs)
}
```

--

## 数式インタプリタをテストする

--

### その前に

- AST（Abstract Syntax Tree）を組み立てる補助関数を用意する
  - 記述が長くなるので

```scala
extension (a: Exp) {
  // 加算
  def |+|(b: Exp): Exp = BinExp("+", a, b)
  // 減算
  def |-|(b: Exp): Exp = BinExp("-", a, b)
  // 乗算
  def |*|(b: Exp): Exp = BinExp("*", a, b)
  // 除算
  def |/|(b: Exp): Exp = BinExp("/", a, b)
}
// 整数
def tInt(value: Int): Exp = VInt(value)
```

--

### 加算をテストする

- `Munit`ライブラリを使ったユニットテスト

```scala
import munit.FunSuite
class EvaluatorSuite extends FunSuite {
  test("1 + 1 == 2") {
    val e = tInt(1) |+| tInt(1)
    assertEquals(eval(e), 2)
  }
}
```

--

### 減算をテストする

```scala
import munit.FunSuite
class EvaluatorSuite extends FunSuite {
  test("1 - 1 == 0") {
    val e = tInt(1) |-| tInt(1)
    assertEquals(eval(e), 0)
  }

  test("1 - 2 == -1") {
    val e = tInt(1) |-| tInt(2)
    assertEquals(eval(e), -1)
  }
}
```

--

### 乗算をテストする

```scala
import munit.FunSuite
class EvaluatorSuite extends FunSuite {
  test("1 * 1 == 1") {
    val e = tInt(1) |*| tInt(1)
    assertEquals(eval(e), 1)
  }

  test("1 * 0 == 0") {
    val e = tInt(1) |*| tInt(0)
    assertEquals(eval(e), 0)
  }

  test("2 * 2 == 4") {
    val e = tInt(2) |*| tInt(2)
    assertEquals(eval(e), 4)
  }
}
```

--

### 除算をテストする

```scala
import munit.FunSuite
class EvaluatorSuite extends FunSuite {
  test("0 / 1 == 0") {
    val e = tInt(0) |/| tInt(1)
    assertEquals(eval(e), 0)
  }

  test("2 / 1 == 2") {
    val e = tInt(2) |/| tInt(1)
    assertEquals(eval(e), 2)
  }

  test("6 / 2 == 3") {
    val e = tInt(6) |/| tInt(2)
    assertEquals(eval(e), 3)
  }
}
```

--

### 複合的な式をテストする

```scala
import munit.FunSuite
class EvaluatorSuite extends FunSuite {
  test("(1 + (2 * 3) - 1) / 2 == 3") {
    // { (1 + (2 * 3)) - 1 } / 2
    val e = tInt(1) |+| (tInt(2) |*| tInt(3)) |-| tInt(1) |/| tInt(2)
    assertEquals(eval(e), 3)
  }
}
```

--

### 数式言語についてまとめ

- まず抽象構文を考える
- 抽象構文を元に抽象構文木を設計する
- 抽象構文木をScala 3で組み立てて、手で実行してみる
- 一般化して解釈できる関数を書く
  - 俗に言う`eval`関数
  - インタプリタと言い換えてもOK
- 動作することをテストする
  - 計算ごとにテストするのがポイント

--

## 質問タイム

--

## 単純なプログラミング言語を作る - 名前付け

- Skala
  - Scala 3で作ったのでSkala

--

## 欲しい機能

- 四則演算＋α
- 変数
  - 代入と参照
- 連接
  - ２つの式を連続して評価
- 条件分岐
  - いわゆる`if`式
- 繰り返し
  - いわゆる`while`式
- 関数
  - 定義 + 呼び出し

--

## 寄り道 - チューリング完全性について

- ほぼべてのプログラミング言語は能力的に等価
  - CでできてJavaでできないことはないし、逆も同じ
- Skalaもチューリング完全「になるはず」
  - つまり、「なんでもできる」
- 手間の問題は別
  - Skalaは機能が少ない
  - 複雑なアルゴリズムを表現しようとすると長くなる

--

### Skalaの抽象構文を考える

```text
プログラム = 関数定義* 式* // 関数定義が並んだ後に式が並ぶ 
式         = 数式 // これまでに定義したもの
           | 比較式 // a < b , a > b, a <= b, a >= b, a == b, a != b
           | 変数代入 // a = 式
           | 変数参照  // a
           | 連接 // {式1; 式2; .. 式n} 
           | 条件分岐 // if(式) 式 else 式
           | 繰り返し // while(式) 式
```

--

### SKalaの抽象構文木を設計する 

--

### リテラルの抽象構文木を設計する

```scala
enum Exp {
  ...
  case VInt(value: Int)
  ...
}
// 例:
import Exp.*
VInt(100) // 100
VInt(200) // 200
VInt(300) // 300
```

--

### 比較式の抽象構文木を設計する

```scala
enum Exp {
  ...
  case BinExp(op: String, lhs: Exp, rhs: Exp)
  ...
}
import Exp.*
BinExp("<", l, r)  // l <  r
BinExp(">", l, r)  // l >  r
BinExp("<=", l, r) // l <= r
BinExp(">=", l, r) // l >= r
BinExp("==", l, r) // l == r
BinExp("!=", l, r) // l != r
```

--

### 変数代入の抽象構文木を設計する

```scala
enum Exp {
  ...
  case Assignment(name: String, expression: Exp)
  ...
}
// 例:
import Exp.*
Assignment("a", VInt(1)) // a = 1
```

--

### 変数参照の抽象構文木を設計する

```scala
enum Exp {
  ...
  case Ident(name: String)
  ...
}
// 例:
import Exp.*
BinExp("+", Ident("a"), VInt(1)) // a + 1
```

--

### 連接の抽象構文木を設計する

```scala
enum Exp {
  ...
  case SeqExp(expressions: List[Exp])
  ...
}
import Exp.*

SeqExp(List(
  Assignment("a", VInt(1)),
  Ident("a")
)) // { a = 1; a; } →
SeqExp(List(
  Assignment("a", VInt(1)),
  Assignment("a", BinExp("+", Ident("a"), VInt(1)))
)) // { a = 1; a = a + 1; } → 2
```

--

### Skalaのインタプリタを作る

--

## その前に補助関数（１）

```scala
import Exp.*
extension (a: Exp) {
  def |+|(b: Exp): Exp = BinExp("+", a, b)
  def |-|(b: Exp): Exp = BinExp("-", a, b)
  def |*|(b: Exp): Exp = BinExp("*", a, b)
  def |/|(b: Exp): Exp = BinExp("/", a, b)
}
```

--

## その前に補助関数（２）

```scala
import Exp.*extension (a: Exp) {
  def |<|(b: Exp): Exp = BinExp("<", a, b)
  def |>|(b: Exp): Exp = BinExp(">", a, b)
  def |<=|(b: Exp): Exp = BinExp("<=", a, b)
  def |>=|(b: Exp): Exp = BinExp(">=", a, b)
  def |==|(b: Exp): Exp = BinExp("==", a, b)
  def |!=|(b: Exp): Exp = BinExp("!=", a, b)
}
```

--

## その前に補助関数（３）

```scala
import Exp.*
def tInt(value: Int): Exp = VInt(value)
def tAssign(name: String, value: Exp): Exp = Assignment(name, value)
def tId(name: String): Exp = Ident(name)
```

--

## その前に補助関数（４）

```scala
import Exp.*
def tSeq(expressions: Exp*): Exp = {
  SeqExp(expressions.toList)
}
```

--

## Skalaのeval関数を実装する（１） - 数式

- 以降、`evalRec`の実装のみを掲載する

```scala
import Exp.*
import scala.collection.mutable.{Map => MMap}
def eval(e: Exp, varEnv: MMap[String, Int], funcEnv: MMap[String, Func]): Int = {
  // 内部関数
  def evalRec(e: Exp): Int = e match {
    ...
    case BinExp("+", lhs, rhs) => evalRec(lhs) + evalRec(rhs)
    case BinExp("-", lhs, rhs) => evalRec(lhs) - evalRec(rhs)
    case BinExp("*", lhs, rhs) => evalRec(lhs) * evalRec(rhs)
    case BinExp("/", lhs, rhs) => evalRec(lhs) / evalRec(rhs)
    ...
  }
  evalRec(e)
}
```

--

## SKalaのeval関数を実装する（２） - 比較式

```scala
def evalRec(e: Exp): Int = e match {
  ...
  case BinExp("<", lhs, rhs)  => 
    if (evalRec(lhs) < evalRec(rhs)) 1 else 0
  case BinExp(">", lhs, rhs)  => 
    if (evalRec(lhs) > evalRec(rhs)) 1 else 0
  case BinExp("<=", lhs, rhs) => 
    if (evalRec(lhs) <= evalRec(rhs)) 1 else 0
  case BinExp(">=", lhs, rhs) => 
    if (evalRec(lhs) >= evalRec(rhs)) 1 else 0
  case BinExp("==", lhs, rhs) => 
    if (evalRec(lhs) == evalRec(rhs)) 1 else 0
  case BinExp("!=", lhs, rhs) => 
    if (evalRec(lhs) != evalRec(rhs)) 1 else 0
  ...
}
```

--

## SKalaのeval関数を実装する（３） - 連接式

1. `bodies`から一つずつ要素を取り出す
2. 取り出した`e`を引数として再帰的に`eval()`
3. 最後の結果を`result`として返す

```scala
def evalRec(e: Exp): Int = e match {
  ... 
  case SeqExp(bodies) =>
    var result: Int = 0
    bodies.foreach { expr =>
      result = evalRec(expr)
    }
    result
  ...
}
```

--

## Skalaのeval関数を実装する（４） - 代入と識別子

```scala
import Exp.*
import scala.collection.mutable.{Map => MutableMap}
def evalRec(e: Exp): Int = e match {
  ...
  case Assignment(name, expr) =>
    val v = evalRec(expr)
    varEnv(name) = v
    v
  case Ident(name) =>
    env.getOrElse(name, sys.error(s"Undefined identifier: $name"))
  ...
}
```

--

## Skalaのeval関数を実装する（５） - 整数

```scala
def evalRec(e: Exp): Int = {
  ...
  case VInt(value) => value
  ...
}
```

--

### Skalaのインタプリタをテストする- 四則演算

- 基本的に数式言語の時と同じなので説明は略

```scala
test("1 + 1 == 2") {
  val e = tInt(1) |+| tInt(1)
  assertEquals(evalExp(e), 2)
}
test("1 - 2 == -1") {
  val e = tInt(1) |-| tInt(2)
  assertEquals(evalExp(e), -1)
}
test("1 * 1 == 1") {
  val e = tInt(1) |*| tInt(1)
  assertEquals(evalExp(e), 1)
}
test("0 / 1 == 0") {
  val e = tInt(0) |/| tInt(1)
  assertEquals(evalExp(e), 0)
}
test("6 / 2 == 3") {
  val e = tInt(6) |/| tInt(2)
  assertEquals(evalExp(e), 3)
}
```

--

### Skalaのインタプリタをテストする - 比較式

```scala
test("1 < 2 == 1") {
  val e = tInt(1) |<| tInt(2)
  assertEquals(evalExp(e), 1)
}
test("1 >= 1 == 1") {
  val e = tInt(1) |>=| tInt(1)
  assertEquals(evalExp(e), 1)
}
test("1 == 1 == 1") {
  val e = tInt(1) |==| tInt(1)
  assertEquals(evalExp(e), 1)
}
test("1 != 0 == 1") {
  val e = tInt(1) |!=| tInt(0)
  assertEquals(evalExp(e), 1)
}
```

--

### Skalaのインタプリタをテストする - 連接/代入式

```scala
test("{a = 100; a} == 100") {
  val e = tSeq(
    tAssign("a", tInt(100)),
    tId("a")
  )
  assertEquals(evalExp(e), 100)
}
test("{a = 100; b = a + 1; b} == 101") {
  val e = tSeq(
    tAssign("a", tInt(100)),
    tAssign("b", tId("a") |+| tInt(1)),
    tId("b")
  )
  assertEquals(evalExp(e), 101)
} 
```

--

## Skalaを拡張する

--

### if式を追加する - 構文木の追加

```scala
enum Exp {
  ...
  case If(condition: Exp, thenClause: Exp, elseClause: Exp)
  ...
}

def tIf(
  condition: Exp, thenClause: Exp, elseClause: Exp
): Exp = MIf(condition, thenClause, elseClause)

// 例: if(1 < 2) 3 else 4
val ifExample = tIf(tInt(1) |<| tInt(2), tInt(3), tInt(4))
```

--

### if式を追加する - eval()の実装

1. `condition`を`eval()`する
2. `condition`が0でなければ3.、0なら4.を実行する
3.  `thenClause` を `eval()` する
4.  `elseClause` を `eval()` する

```scala
def evalRec(e: Exp): Int = e match {
  ...
  case If(condition, thenClause, elseClause) =>
    if (evalRec(condition) != 0)
      evalRec(thenClause)
    else
      evalRec(elseClause)
  ...
}
```

--

### if式をテストする

```scala
test("(if(i > 2) 2 else 1) == 1") {
  val e = tIf(tInt(1) |<| tInt(2), tInt(2), tInt(1))
  assertEquals(evalExp(e), 1)
}
```

--

### while式を追加する - 構文木の追加

```scala
enum Exp {
  ...
  case While(condition: Exp, bodies: List[Exp])
  ...
}
def tWhile(condition: Exp, bodies: Exp*): Exp = 
  MWhile(condition, bodies.toList)
```

--

### while式を追加する - eval()の実装

1. `condition`を`eval()`する
2. 結果が0でなければ3.、0なら6.を実行する
3. `bodies`の各要素`body`を`eval()`する
4. `condition`を`eval()`する
5. 結果が0でなければ3.、0なら6.を実行する
6. `0`を返す


```scala
def evalRec(e: Exp): Int = e match {
  ...
  case Exp.While(condition, bodies) =>
    while (evalRec(condition) != 0) {
      bodies.foreach(body => evalRec(body))
    }
    0
  ...
}
```

--

### while式をテストする

```scala
test("""
  i = 0;
  while(i < 10) {
    i = i + 1;
  };
  i
""") {
  val e = tSeq(
    tAssign("i", tInt(0)),
    tWhile(tId("i") |<| tInt(10),
      tAssign("i", tId("i") |+| tInt(1))
    ),
    tId("i")
  )
  assertEquals(evalExp(e), 10)
}
```

--

### 関数定義を追加する - 構文木の追加

- `name`: 関数名
- `params`: 仮引数
- `body`: 関数の本体

```scala
enum Exp {
  ...
  case Func(name: String, params: List[String], body: Exp)
  ...
}
import Exp.*
def tFunction(
  name: String, params: List[String], body: Exp
): Func = Func(name, params, body)
```

--

### 関数呼び出しを追加する - 構文木の追加

```scala
enum Exp {
  ...
  case Call(name: String, args: List[Exp])
  ...
}
import Exp.*
def tCall(name: String, args: Exp*): Call = {
  Call(name, args.toList)
}
```

--

### 「プログラム」構文木の追加

- 関数定義はSkalaでは式ではないため
- `functions`: 関数定義の配列
- `bodies`: 式の列（プログラム本体）

```scala
case class Program(
  functions: List[Func], bodies: List[Exp]
)
def tProgram(
  functions: List[Func], bodies: Exp*
): Program = Program(functions, bodies.toList)
```

--

### 関数定義を処理する

- 連想配列`funcEnv`に関数を登録
  - `funcEnv(f.name) = f`
  - 後で`funcEnv`を参照すれば関数が取得できる

```scala
import Exp.*
import scala.collection.mutable.{Map => MMap}
def evalProgram(program: Program): Int = {
  val funcEnv: MMap[String, Func] = MMap.from(map)
  program.functions.foreach { f =>
    funcEnv(f.name) = f
  }
  var result: Int = 0
  program.bodies.foreach{body => 
    result = eval(exp, MMap.empty, funcEnv)
  }
  result
}
```

--

### 関数呼び出しを処理する

1. `funcEnv`から`e.name`に対応する関数を取得
2. 実引数（`args`）と仮引数（`params`）を対応づけた新しい環境（`newEnv`）を作る
3. 関数の本体（`body`）と環境（`newEnv`）を引数に再帰的に`eval()`を呼び出す
  - 新しく作られた環境は戻ってきた後のどっかでGCに回収される

--

### 関数呼び出しを処理する - コード

```scala
def evalRec(e: Exp)): Int = e match {
  ...
  case Call(name, args) => 
    funcEnv.get(name) match {
      case Some(Func(_, params, body)) =>
        val argValues = args.map(evalRec)
        val newEnv = MMap.empty[String, Int] ++ varEnv
        params.zip(argValues).foreach { case (param, argVal) =>
          newEnv(param) = argVal
        }
        // ここではevalRecでなくevalを呼び出す
        eval(body, newEnv, funcEnv)
      case None => sys.error(s"Function $name is not defined")
    }
  ...
}
```

--

### 関数定義/呼び出しをテストする

- `add` 関数を定義: 
  - `function add(a, b) { return a + b; }` 相当

```scala
test("""
  function add(a, b) {
    return a + b;
  }
  add(1, 2) == 3
""") {
  val program = tProgram(
    List(tFunction("add", List("a", "b"), tId("a") |+| tId("b"))),
    tCall("add", tInt(1), tInt(2))
  )
  assertEquals(evalProgram(program), 3)
}
```

--

### 行数の話

- 数式言語のインタプリタ
  - 総行数: 約28行
- Skalaのインタプリタ
  - 総行数: 約154行（テスト除く） 

150行あれば最低限の言語は作れる！

--

## 構文解析の話

--

## 本当はテキストで書きたい

- それには「構文解析」が必要

```
function sum(from, to) {
  result = from;
  while(i <= to) {
    result = result + i;
    i = i + 1;
  }
  result;
}
sum(1, 4) → 10
```

--

## 構文解析は煩雑

- 構文解析アルゴリズムの理解が必要
  - `LL(1)`, `LR(1)`, `PEG`
    - PEGは比較的簡単だけども（我田引水）
- 構文解析機を書くのは退屈
  - 冗長なコードを書かされる≠難しい
- 今回はJSONで構文を記述する

--

## 数式言語のAST in Scala (再)

```scala
enum Exp {
  case BinExp(op: String, lhs: Exp, rhs: Exp)
  case VInt(value: Int)
}
```

- これらのオブジェクトをJSONで表現する

--

## 数式言語 in JSON: 数値

```scala
VInt(100) // 100 
```

↓

```js
1
```

--

## 数式言語 in JSON: 加算

```scala
BinExp("+", VInt(100), VInt(200)) // 100 + 200
```

↓ 

```js
['+', 100, 200] // LispのS式ぽい記述
```

--

## 数式言語 in JSON: 減算

```scala
BinExp("-", VInt(100), VInt(200)) // 100 - 200
```

↓ 

```js
['-', 100, 200]
```

--

## 数式言語 in JSON: 乗算

```scala
VInt(100) |*| VInt(200)) // 100 * 200
```

↓ 

```js
['*', 100, 200]
```

--

## 数式言語 in JSON: 除算

```scala
BinExp("/", VInt(100), VInt(2)) // 100 / 2
```

↓ 

```js
['/', 100, 2]
```

--

## 数式言語 in JSON: 複雑な式

```scala
// 1 + 2 * 3
VInt(1) |+| (VInt(2) |*| VInt(3)) 
```

↓

```js
['+', 1, ['*', 2, 3]]
```

--

## JSONをASTに変換する - 整数

```scala
import play.api.libs.json._
def translateToAst(json: JsValue): Exp = json match {
  case JsNumber(n) =>
    tInt(n.toInt)
}
```

--

## JSONをASTに変換する - 連接

```scala
def translateToAst(json: JsValue): Exp = json match {
  ...
  case JsArray(values) if values.nonEmpty =>
    values.head match {
      case "seq" =>
        SeqExp(values.tail.map(translateToAst)*)
      ...
    }
  ...
}
```

--

## JSONをASTに変換する - if式

```scala
def translateToAst(json: JsValue): Exp = json match {
  ...
  case JsArray(values) if values.nonEmpty =>
    values.head match {
      case "if" =>
        If(
          translateToAst(values(1)), 
          translateToAst(values(2)), translateToAst(values(3))
        )
      ...
    }
  ...
}
```

--

## JSONをASTに変換する - while式

```scala
def translateToAst(json: JsValue): Exp = json match {
  ...
  case JsArray(values) if values.nonEmpty =>
    values.head match {
      case "while" =>
        While(translateToAst(values(1)), translateToAst(values(2)))
      ...
    }
  ...
}
```

--

## JSONをASTに変換する - 代入式

```scala
def translateToAst(json: JsValue): Exp = json match {
  ...
  case JsArray(values) if values.nonEmpty =>
    values.head match {
      case "assign" =>
        // Expecting: ["assign", name, expression]
        tAssign(values(1).as[String], translateToAst(values(2)))
      ...
    }
  ...
}
```

--

## JSONをASTに変換する - 関数呼び出し

```scala
def translateToAst(json: JsValue): Exp = json match {
  ...
  case JsArray(values) if values.nonEmpty =>
    values.head match {
      case "call" =>
        // Expecting: ["call", functionName, arg1, arg2, ...]
        tCall(values(1).as[String], values.drop(2).toSeq.map(translateToAst)*)
      ...
    }
  ...
}
```

--

## JSONをASTに変換する - 識別子

```scala
def translateToAst(json: JsValue): Exp = json match {
  ...
  case JsArray(values) if values.nonEmpty =>
    values.head match {
      case "id" =>
        // Expecting: ["id", name]
        tId(values(1).as[String])
      ...
    }
  ...
}
```

--

## JSONをASTに変換する - 二項演算

```scala
def translateToAst(json: JsValue): Exp = json match {
  ...
  case JsArray(values) if values.nonEmpty =>
    values.head match {
      case "+" =>
        translateToAst(values(1)) |+| translateToAst(values(2))
      case "-" =>
        translateToAst(values(1)) |-| translateToAst(values(2))
      ...
    }
  ...
}
```

--

### JSON文字列を解釈する 

1. `JSON.parse(input)`でJSONに変換
2. JSONに対して`translateToAst()`を呼び出す
3. 講義で作った`evalute()`を呼び出す

```scala
def evalJsonExp(jsonString: String): Int = {
  val json: JsValue = Json.parse(jsonString)
  val ast: Exp      = translateToAst(json)
  evalExp(ast)
}
// 使用例:
evalJsonExp("""["+", 1, 2]""") // -> 3
```

--

### 構文解析を避けた理由

- プログラミング言語開発初心者にとって煩雑
- 動作の本質にはあまり関係ない
- 「UIとしての構文」は依然として重要

--

### 構文解析について詳しく知りたい人は

- [Parsing Teqhniques 2nd Edition](https://www.amazon.co.jp/Parsing-Techniques-Practical-Monographs-Computer-ebook/dp/B0017AMLL8)
  - 構文解析のことが大体なんでも乗ってる
  - 洋書のみでマニアック過ぎる
- [Go言語で作るインタプリタ](https://www.amazon.co.jp/dp/4873118220/)
  - 読んでないけど、最低限の構文解析は扱ってるらしい
  - 評判は良さげ
- [WEB+DB PRESS Vol.125 特集作って学ぶプログラミング言語](https://www.amazon.co.jp/dp/4297124351)
  - 拙著
  - 今回のと似たアプローチ
  - 最後の方で構文解析を扱っている

--

## 型検査

--

- これまでのSkalaは型を持っていない
  - 誤った型の演算はそのままスルー
- 例： `(1 <= 2) + 3` はエラーにならない
  - 本当はエラーになって欲しい

型検査の出番

--

## 今回やりたいこと

- 真偽値と整数を区別する
- 真偽値と整数の演算を**実行前に**エラーにする

--

## 型を定義する

```scala
enum Type {
  case TInt // 整数型
  case TBool // 真偽値型
}
```

--

## 型検査のための関数 - リテラル

- 整数リテラルは整数型
- 以降、`typeOfRec`関数のみ掲載

```scala
import scala.collection.mutable.{Map => MMap}
def typeOf(
  e: Exp, typeEnv: MMap[String, Type], funcenv: MMap[String  Func]
): Type = {
  def typeOfRec(e: Exp): Type = e match {
    case VInt(_) => TInt
  }
  typeOfRec(e)
}
```

--

## 型検査のための関数 - 四則演算

- 四則演算のオペランドは整数型で結果も整数型

```scala
def typeOfRec(e: Exp): Type = e match {
  ...
  case BinExp("+" | "-" | "*" | "/", lhs, rhs) =>
    val lType = typeOfRec(lhs)
    val rType = typeOfRec(rhs)
    if (lType == TInt && rType == TInt) TInt
    else sys.error(s"Type mismatch: $lType and $rType")
}
```

--

## 型検査のための関数 - 比較式

- 比較式のオペランドは整数型で結果は真偽値型

```scala
def typeOfRec(e: Exp): Type = e match {
  ...
  case BinExp("<" | ">" | "<=" | ">=" | "==" | "!=", lhs, rhs) =>
    val lType = typeOfRec(lhs)
    val rType = typeOfRec(rhs)
    if (lType == TInt && rType == TInt) TBool
    else sys.error(s"Type mismatch: $lType and $rType")
}
```

--

## 型検査のための関数 - 識別子

- 識別子の型は変数の型

```scala
def typeOfRec(e: Exp): Type = e match {
  ...
  case Ident(name) =>
    typeEnv.getOrElse(name, sys.error(s"Undefined identifier: $name"))
}
```

--

## 型検査のための関数 - 代入式

- 代入式の左辺は変数で右辺は任意の式
- 代入式の結果は右辺の型

```scala
def typeOfRec(e: Exp): Type = e match {
  ...
  case Assignment(_, expr) => 
    val expType = typeOfRec(expression)
    if (typeEnv.contains(name)) {
      val existingType = typeEnv(name)
      if (existingType != expType)
        sys.error(s"Type mismatch: '$name': has type $existingType, but expression is $expType")
    } else {
      typeEnv(name) = expType  // implicit declaration
    }
    expType
}
```

--

## 型検査のための関数 - 連接式

- 連接式の結果は最後の式の型

```scala
def typeOfRec(e: Exp): Type = e match {
  ...
  case SeqExp(bodies) =>
    var resultType: Type = TInt  // default value; will be updated below
    bodies.foreach {exp =>
      resultType = typeOfRec(exp)
    }
    resultType
}
```

--

## 型検査のための関数 - if式

- if式の条件式は真偽値型でthen節とelse節は同じ型
- if式の結果はthen節とelse節の型
:
```scala
def typeOfRec(e: Exp): Type = e match {
  ...
  case If(condition, thenClause, elseClause) =>
    val condType = typeOfRec(condition)
    if (condType != TBool)
      sys.error(s"Type mismatch: expected Bool, got $condType")
    val thenType = typeOfRec(thenClause)
    val elseType = typeOfRec(elseClause)
    if (thenType != elseType)
      sys.error(s"Type mismatch: expected $thenType, got $elseType")
    thenType
}
```

--

## 型検査のための関数 - while式

- while式の条件式は真偽値型で本体は任意の型
- while式の結果は整数型

```scala
def typeOfRec(e: Exp): Type = e match {
  ...
  case While(condition, bodies) =>
    val condType = typeOfRec(condition)
    if (condType != TBool)
      sys.error(s"Type mismatch: expected Bool, got $condType")
    bodies.foreach { expr =>
      typeOfRec(e)
    }
    TInt
}
```

--

## 型検査のための関数 - 関数呼び出し

```scala
def typeOfRec(e: Exp): Type = e match {
  ...
  case Call(name, args) =>
    funcEnv.get(name) match {
      case Some(Func(_, params, retType, _)) =>
        if (params.length != args.length) sys.error(s"Type mismatch: function $name' expected ${params.length} args, got ${args.length}" args)
        params.zip(args).foreach { case ((paramName, paramType), arg) =>
          val argType = typeOfRec(arg)
          if (argType != paramType) sys.error(s"Type mismh: function '$name' expected $paramType for '$paramName', but got $argType")
        }
        retType
      case None =>
        sys.error(s"Undefined function: '$name'")
    }
}
```

--

## まとめ

- 数式言語のインタプリタを作成した
  - 総行数: 約28行
- プログラミング言語Skalaのインタプリタを作成した
  - JSON形式の具象構文を設計した
    - 総行数: 約270行
  - 型検査を追加した
    - 総行数: 約440行

- URL: [https://github.com/kmizu/skala](https://github.com/kmizu/skala)

--

自分だけのプログラミング言語を作ってみよう！
