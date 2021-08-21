# 16章 ベクトル



### 16.0 ライブラリの読み込み

```text
library("tidyverse")
```

### 16.1 はじめに

{% hint style="info" %}
練習問題はありません
{% endhint %}

### 16.2 ベクトルの基本

{% hint style="info" %}
練習問題はありません
{% endhint %}

### 16.3 重要なアトミックベクトル

#### 練習問題1 `is.finite(x)`と`!is.infinite(x)`の違いは何か。

`is.finite()`は`NA`, `NaN`,`Inf`,`-Inf`は、`NA`, `NaN`を`FALSE`として扱います。同じく、`!is.infinite()`は`NA`, `NaN`を`TRUE`として扱います。

```text
x <- c(0, 1, NA, NaN, Inf, -Inf)

is.finite(x)
[1]  TRUE  TRUE FALSE FALSE FALSE FALSE
# !is.finite(x)
# [1] FALSE FALSE  TRUE  TRUE  TRUE  TRUE

# is.infinite(x)
# [1] FALSE FALSE FALSE FALSE  TRUE  TRUE
!is.infinite(x)
[1]  TRUE  TRUE  TRUE  TRUE FALSE FALSE
```

|  | is.finite\(\) | is.infinite\(\) |
| :--- | :--- | :--- |
| NA | FALSE | FALSE |
| NaN | FALSE | FALSE |
| Inf | FALSE | TRUE |
| -Inf | FALSE | TRUE |

#### 練習問題2 `dplyr::near()`のソースコードを読んで、機能を説明しなさい。

`near()`は、完全に等価かどうかをチェックする代わりに、2つの数値の差が一定の許容範囲内であることをチェックします。デフォルトでは、許容誤差はコンピューターが表現できる最小の浮動小数点数\(`.Machine$double.eps`\)の平方根に設定されています。

```text
dplyr::near
　function (x, y, tol = .Machine$double.eps^0.5) 
　{
　    abs(x - y) < tol
　}
　<bytecode: 0x1edb640>
　<environment: namespace:dplyr>

.Machine$double.eps^0.5
[1] 1.490116e-08> 

.Machine$double.eps
[1] 2.220446e-16
```

#### 練習問題3 論理ベクトルは3つの可能な値を取る。`integer`に取り得る値はいくつあるのか。`doble`に取り得る値はいくつあるか。

整数ベクトルの場合、`TRUE`、`FALSE`、`NA_integer_integer`の3つです。整数ベクトルで表現できる整数値の範囲は`(2^31)-1`です。

```text
c(TRUE, FALSE, NA_integer_)
[1]  1  0 NA

.Machine $ integer.max
[1] 2147483647

.Machine$double.xmax
[1] 1.8e+308
```

#### 練習問題4 `double`を整数に変換できるようにする少なくとも4つの関数を考えなさい。

整数に切り捨てるか丸めることによって、`double`を整数に変換できます。

```text
x <- c(-2.5, -1.5, 0, 1.5, 2.5)

round(x)
[1] -2 -2  0  2  2
floor(x)
[1] -3 -2  0  1  2
ceiling(x)
[1] -2 -1  0  2  3
trunc(x)
[1] -2 -1  0  1  2
signif(x, digits = 1)
[1] -2 -2  0  2  2
```

この例を見てわかるように、`round()`は四捨五入ではありません。四捨五入であれば、`2.5`は`3`であるべきです。

```text
round2 <- function(x, to_even = TRUE) {
  q <- x %/% 1
  r <- x %% 1
  q + (r >= 0.5)
}

round(x)
[1] -2 -2  0  2  2

round2(x)
[1] -2 -1  0  2  3
```

| 丸め関数 | 処理内容 |
| :--- | :--- |
| round\(x, digits = 0\) | IEEE式で丸めを行なう |
| ceiling\(x\) | x未満でない最小の整数を返す |
| floor\(x\) | x以上でない最大の整数を返す |
| signif\(x, digits = 6\) | digitsで指定された有効桁数に丸める |
| trunc\(x\) | xを0 へ向かって整数化する |

#### 練習問題5 `{readr}パッケージ`のどの関数を使って文字列を`logical`、`integer`、`double`のベクトルに変えることができるか？

`parse_logical()`、`parse_integer()`、`parse_number()`で変換できます。`parse_logical()`は、`1`や`t`なども`TRUE`とみなします。`parse_integer()`は、0から始まっている文字型の数えあれば問題なくパースできますが、記号や小数点があると正確にパースできません。

```text
parse_logical(c("TRUE", "FALSE", "1", "0", "true", "t", "NA"))
[1]  TRUE FALSE  TRUE FALSE  TRUE  TRUE    NA

parse_integer(c("1235", "0134", "NA"))
[1] 1235  134   NA

parse_integer(c("1000", "$1,000", "$1,000.11", "10.00", "NA"))
 警告:  3 parsing failures.
row col               expected    actual
  2  -- an integer             $1,000   
  3  -- an integer             $1,000.11
  4  -- no trailing characters .00      

[1] 1000   NA   NA   NA   NA
```

このような場合は、`parse_number()`で変換できます。

```text
parse_number(c("1000", "$1,000", "$1,000.11", "10.00", "NA"))
[1] 1000.00 1000.00 1000.11   10.00      NA
```

### 16.4 アトミックベクトルを使う

#### 練習問題1 `mean(is.na(x))`はどのように機能するのか。`sum(!is.finite(x))`とどう違うのか。

`mean(is.na(x))`は、ベクトル内の欠損値の割合を計算します。

```text
x <- c(-1, 0, 1,-Inf, Inf, NA, NaN)

mean(is.na(x))
[1] 0.2857143
```

`sum(!is.finite(x))`は、`NA`、`NaN`、`Inf`に等しいベクトル内の要素数を計算します。

```text
sum(!is.finite(x))
[1] 4

!is.finite(x)
[1] FALSE FALSE FALSE  TRUE  TRUE  TRUE  TRUE
```

#### 練習問題2 `is.vector()`のドキュメントを読んで、`is.atomic()`のアトミックベクトルの定義と異なるのか説明しなさい。

`is.vector()`は、オブジェクトに`name`以外の属性\(`attr`\)がないかどうかを確認します。したがって、 `list`は`TRUE`として扱われます。

```text
is.vector(list(a = 1, b = 1))
[1] TRUE

vec <- 1:10
attr(vec, 'foo') <- 'bar'
is.vector(vec);
[1] FALSE
```

`is.atomic()`は "logical"、 "integer"、 "numeric"、 "complex"、 "character"、 "raw"どうかを論理判定します。属性を持つかどうかは関係ありません。

```text
is.atomic(vec)
[1] TRUE
```

#### 練習問題3 `setNames()`と`purrr::set_names()`を比較しなさい。

この2つの関数は、ベクトルの要素に名前をつけることができます。`purrr::set_names()`は関数を使える点で異なります。

```text
setNames(1:4, c("a", "b", "c", "d"))
a b c d 
1 2 3 4 

purrr::set_names(1:4, c("a", "b", "c", "d"))
a b c d 
1 2 3 4 

purrr::set_names(c(a = 1, b = 2, c = 3, d = 4), toupper)
A B C D 
1 2 3 4 
```

#### 練習問題4 入力としてベクトルを受け取る関数を作成しなさい。

* 最後の値を抽出するために `[`または`[[`を使うべきか？

単一の要素を抽出するために`[[`を使います。`[`または`[[`はリストが絡んでいる場合、注意が必要です。

```text
last_value <- function(x) {
  # check for case with no length
  if (length(x)) {
    x[[length(x)]]
  } else {
    x
  }
}

last_value(1)
[1] 1

last_value(1:10)
[1] 10
```

* 偶数位置の要素

```text
even_indices <- function(x) {
  if (length(x)) {
    x[seq_along(x) %% 2 == 0]
  } else {
    x
  }
}

even_indices(1:10)
[1]  2  4  6  8 10
```

* 最後の値を除くすべての要素

```text
not_last <- function(x) {
  n <- length(x)
  if (n) {
    x[-n]
  } else {
    # n == 0
    x
  }
}

not_last(1:3)
[1] 1 2
```

* 偶数のみ（および欠損値なし）

欠損値を含む場合も多いので、欠損値を含んだ状態で作ります。

```text
even_numbers <- function(x) {
  x[x %% 2 == 0]
}
even_numbers(c(-4:4, NA, NaN, Inf, -Inf))
[1] -4 -2  0  2  4 NA NA NA NA
```

#### 練習問題5 `x[-which(x > 0)]`は`x[x <= 0]`とどう異なるのか。

これらの式は、欠損値を扱う方法が異なります。また、`which()`は条件に一致する要素のインデックスを返し、等号を要素に対して使うと、`TRUE or FALSE`が返されます。

```text
x <- c(-1:1, Inf, -Inf, NaN, NA)

which(x > 0)
[1] 3 4

x[-which(x > 0)]
[1]   -1    0 -Inf  NaN   NA

x <= 0
[1]  TRUE  TRUE FALSE FALSE  TRUE    NA    NA

x[x <= 0]
[1]   -1    0 -Inf   NA   NA
```

#### 練習問題6 ベクトルの長さよりも大きい正の整数でサブセット化するとどうなるか。存在しない名前でサブセットするとどうなるか。

存在しないインデックスや名前でサブセットすると、エラーが表示されます。

```text
x <- 1:10
x[[100]]
[[100]] でエラー:  添え字が許される範囲外です 

x <- c(a = 10, b = 20)
x[["c"]]
x[["c"]] でエラー:  添え字が許される範囲外です 
```

### 16.5 再帰ベクトル\(リスト\)

#### 練習問題1 次の`list`を入れ子集合として描きます。

* `list(a, b, list(c, d), list(e, f))`

`a`と`b`と同じ階層に`c`と`d`が入ったリストと、別に`e`と`f`が入ったリストがある。

```text
list(a, b, list(c, d), list(e, f))
[[1]]
[1] "a"

[[2]]
[1] "b"

[[3]]
[[3]][[1]]
[1] "c"

[[3]][[2]]
[1] "d"


[[4]]
[[4]][[1]]
[1] "e"

[[4]][[2]]
[1] "f"
```

* `list(list(list(list(list(list(a))))))`

6重にネストしているリスト。

```text
list(list(list(list(list(list(a))))))
[[1]]
[[1]][[1]]
[[1]][[1]][[1]]
[[1]][[1]][[1]][[1]]
[[1]][[1]][[1]][[1]][[1]]
[[1]][[1]][[1]][[1]][[1]][[1]]
[1] "a"
```

**練習問題2 `list`をサブセット化しているかのように`tibble`をサブセットするとどうなるのか。リストと`tibble`の主な違いは何か？**

単一の要素をベクトルとして抽出するためには、`[[`を使います。リストと`tibble`の違いは、リストは異なる長さのベクトルやデータフレームを内包できますが、`tibble`はデータフレームと同じで、異なる長さでは保持できません。

```text
x <- tibble(a = 1:2, b = 3:4)

x
# A tibble: 2 x 2
      a     b
  <int> <int>
1     1     3
2     2     4

x[["a"]]
[1] 1 2

x["a"]
# A tibble: 2 x 1
      a
  <int>
1     1
2     2
```

### 16.6 属性

{% hint style="info" %}
練習問題はありません
{% endhint %}

### 16.7 拡張ベクトル

#### 練習問題1 `hms::hms(3600)`は何を返しますか。拡張ベクトルはどのような基本型の上に構築され、どんな属性を使うのか？

`hms` クラスのオブジェクトを返し、時刻を`％H：％M：％S`の形式で出力します。

```text
x <- hms::hms(3600)
class(x)
[1] "hms"      "difftime"

x
01:00:00 # 1 hour = 3600 min
```

#### 練習問題2 長さの異なる列を持つ`tibble`を試してみてください。何が起こるのですか？

長さの異なる2つのベクトルを使用して`tibble`を作成しようとすると、`tibble`はエラーを返します。

```text
x <- tibble(a = 1:2, b = 3:5)
 エラー: Tibble columns must have consistent lengths, only values of length one are recycled:
* Length 2: Column `a`
* Length 3: Column `b`
Call `rlang::last_error()` to see a backtrace
```

長さが短い場合、リサイクル規則によって、足りていな分をリサイクルします。

```text
x <- tibble(a = 1, b = 3:5)
x
# A tibble: 3 x 2
      a     b
  <dbl> <int>
1     1     3
2     1     4
3     1     5
```

#### 練習問題3 上記の定義に基づいて、`list`を`tibble`の列として持つことは問題ありませんか。

上記のエラーメッセージは異なる長さを持つベクトルに関するものです。`doble`、`character`、`integer`、`logical`、`date`などの異なる型のベクトルをリストとして持つことも`tibble`は可能です。

```text
tibble(x = 1:6, 
       y = list(1.11, "a", 1L, TRUE, today(), list(1:3)))

# A tibble: 6 x 2
      x y         
  <int> <list>    
1     1 <dbl [1]> 
2     2 <chr [1]> 
3     3 <int [1]> 
4     4 <lgl [1]> 
5     5 <date [1]>
6     6 <list [1]>
```

