# 15章 関数

### 15.0 ライブラリの読み込み

```text
library("tidyverse")
library("lubridate")
library("MASS")
```

### 15.1 はじめに

{% hint style="info" %}
練習問題はありません。
{% endhint %}

### 15.2 いつ関数を書くべきか？

#### 練習問題1 `TRUE`はなぜ`rescale01()`のパラメタではないのか。`x`に欠損値があり、`na.rm = FALSE`だとどうなるか。

```text
rescale01 <- function(x) {
  rng <- range(x, na.rm = TRUE, finite = TRUE)
  (x - rng[1]) / (rng[2] - rng[1])
}
```

`TRUE`はなぜ`rescale01()`のパラメタではないのか、の部分は意図がよくわからないので、それはさておき…作業しやすいように3連ドットで一部変更します。まず、先程の関数と同じ状態であれば、`NA`を除いて計算されるので、`NA`は`NA`として表示されます。`FALSE`にしても結果が変わりません。

```text
rescale01 <- function(x, ...) {
  rng <- range(x, ...)
  (x - rng[1]) / (rng[2] - rng[1])
}

x <- c(1:10, NA)  
rescale01(x, na.rm = TRUE, finite = TRUE)  
 [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556
 [7] 0.6666667 0.7777778 0.8888889 1.0000000        NA
 
rescale01(x, na.rm = FALSE, finite = TRUE) 
 [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556
 [7] 0.6666667 0.7777778 0.8888889 1.0000000        NA
```

これは`finite = TRUE`が`NA`をドロップしているからです。なので、`na.rm = TRUE`かつ`finite = FALSE`にすることで、計算の中に`NA`が含まれる`NA`のベクトルを返します。

```text
rescale01(x, na.rm = FALSE, finite = FALSE) 
 [1] NA NA NA NA NA NA NA NA NA NA NA
```

#### 練習問題2 `rescale01()`の2つ目の引数によって、無限の値は変更されません。`0`は`-Inf`にマッピングされ、`1`は`Inf`にマップされるように書き直しなさい。

`rescale01()`の中で、条件に一致すれば変換を施すように修正します。

```text
rescale01 <- function(x, ...) {
  rng <- range(x, ...)
  res <- (x - rng[1]) / (rng[2] - rng[1])
  res[res == -Inf] <- 0
  res[res == Inf] <- 1
  res
}

rescale01(c(-Inf, 0, 1, NA, NA, 2,3,4, Inf),
          na.rm = TRUE, finite = TRUE)
[1] 0.00 0.00 0.25   NA   NA 0.50 0.75 1.00 1.00
```

#### 練習問題3 次のコードを関数に変えてください。

`mean(is.na(x))`は、NAの割合を計算する関数です。

```text
x <- c(2,4,6, NA)

mean(is.na(x))
[1] 0.25

# わけて書く
na_ratio <- function(x, na.rm = FALSE){
  res <- is.na(x)
  res <- mean(res)
  res
}

na_ratio(x)
[1] 0.25

# シンプル版
na_ratio <- function(x) {
  mean(is.na(x))
}
na_ratio(x)
[1] 0.25
```

この関数はベクトルの各値が全体のどの程度を占めるかを計算する関数です。

```text
x / sum(x, na.rm = TRUE)

x <- 1:5
prop <- function(x, na.rm = FALSE) {
  x / sum(x, na.rm = na.rm)
}
prop(x)
[1] 0.06666667 0.13333333 0.20000000 0.26666667 0.33333333

prop(rep(1,5))[1] 0.2 0.2 0.2 0.2 0.2
```

これは変動係数の計算式です。`NA`が含まれていると、デフォルトで`NA`を返すようにしておきます。

```text
sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)

cv <- function(x, na.rm = FALSE) {
  sd(x, na.rm = na.rm) / mean(x, na.rm = na.rm)
}

cv(x)
[1] 0.5270463
```

#### 練習問題4 数値ベクトルの分散と歪度を計算するための独自の関数を作成しなさい。

分散を計算する関数を作ります。

```text
x <- 1:6

myvar <- function(x, na.rm = TRUE) {
  n <- length(x)
  m <- mean(x, na.rm = TRUE)
  dev <- (x - m)^2
  sum(dev) / (n - 1)
}

myvar(x)
[1] 3.5 

var(x)
[1] 3.5
```

歪度を計算する関数を作ります。

```text
set.seed(1234)
x <- rnorm(10)

skew <- function(x) {
  v <- sd(x)
  m <- mean(x)
  mean((x-m)^3)/(v^3)  
}

skew(x)
[1] -0.3623033
```

#### 練習問題5 同じ長さの2つのベクトルを取り、両ベクトルの`NA`が一致する組み合わせの合計を返す関数`both_na()`を書きなさい。

```text
both_na <- function(x, y) {
  sum(is.na(x) & is.na(y))
}

x <- c(NA, NA, 1, NA)
y <- c(NA, 1, NA, NA)

both_na(x, y)
[1] 2
```

#### 練習問題6 次の関数は何をするのですか？

`is_directory()`はディレクトリかどうかを返す関数で、`is_readable()`はアクセス可能かどうかの関数です。

```text
is_directory <- function(x) file.info(x)$isdir
is_readable <- function(x) file.access(x, 4) == 0
```

#### 練習問題7 Little Bunny Foo Foo、この曲にはたくさんの重複があるので、関数を使用して重複を減らしなさい。

```text
Little bunny Foo Foo
Hopping through the forest
Scooping up the field mice
And bopping them on the head

Down came the Good Fairy, and she said
"Little bunny Foo Foo
I don’t want to see you   Scooping up the field mice

And bopping them on the head.
I’ll give you three chances,
And if you don’t stop, I’ll turn you into a GOON!"
And the next day…
```

質問の意図がよくわからず、[https://jrnold.github.io/r4ds-exercise-solutions/functions.htm](https://jrnold.github.io/r4ds-exercise-solutions/functions.html)の回答をそのまま記載しておきます。

```text
threat <- function(chances) {
  give_chances(
    from = Good_Fairy,
    to = foo_foo,
    number = chances,
    condition = "Don't behave",
    consequence = turn_into_goon
  )
}

lyric <- function() {
  foo_foo %>%
    hop(through = forest) %>%
    scoop(up = field_mouse) %>%
    bop(on = head)

  down_came(Good_Fairy)
  said(
    Good_Fairy,
    c(
      "Little bunny Foo Foo",
      "I don't want to see you",
      "Scooping up the field mice",
      "And bopping them on the head."
    )
  )
}

lyric()
threat(3)
lyric()
threat(2)
lyric()
threat(1)
lyric()
turn_into_goon(Good_Fairy, foo_foo)
```

### 15.3 関数は人間とコンピュータのためのもの

#### 練習問題1 次の3つの関数を読んで、何をするのか調べなさい。また、良い名前をブレインストーミングしなさい。

これは`prefix`で文字列が始まるかをチェックする関数です。`check_prefix()`なんかでもいいかもしれません。

```text
f1 <- function(string, prefix) {
  substr(string, 1, nchar(prefix)) == prefix
}

f1(c("Apple", "App", "Banana"), "Ap")
[1]  TRUE  TRUE FALSE

f1(c("Apple", "App", "Banana"), "B")
[1] FALSE FALSE  TRUE
```

この関数は、ベクトルの最後の要素を取り除く関数です。シンプルに`del_last()`なんかでもいいかもしれません。

```text
f2 <- function(x) {
  if (length(x) <= 1) {
    return(NULL)
  }
  x[-length(x)]
}
```

この関数は、`x`の要素分、`y`を繰り返す関数です。シンプルに`recycle()`なんかでもいいかもしれません。

```text
f3 <- function(x, y) {
  rep(y, length.out = length(x))
}

f3(1:10, 1)
[1] 1 1 1 1 1 1 1 1 1 1
 
f3(1:10, "a")
[1] "a" "a" "a" "a" "a" "a" "a" "a" "a" "a"
```

#### 練習問題2 あなたが最近書いた関数を取り、5分かけてより良い名前と内容をブレインストーミングします。

読者に譲ります。

#### 練習問題3 `rnorm()`と`MASS::mvrnorm()`を比較してください。それらをより一貫性のあるものにすることができるか？

`MASS::mvrnorm` は、多変量正規分布からのサンプルに対して、`rnorm()`は単変量正規分布からのサンプル。`rnorm()`の主な引数は`n`、`mean`、`sd` です。`MASS::mvrnorm`の主な引数は`n`、`mu`、`Sigma`です。一貫性を保つために、それらの引数は同じ名前であるべきです。

```text
args(rnorm)
function (n, mean = 0, sd = 1) 

args(mvrnorm)
function (n = 1, mu, Sigma, tol = 1e-06, empirical = FALSE, EISPACK = FALSE) 
```

#### 練習問題4 `rnorm()`, `dnorm()`よりも`norm_r()`, `norm_d()`が良いのはなぜか。単体のケースも作りなさい。

`norm_r()`と`norm_d()`の場合、分布によって機能をサブセット化できますが、`rnorm()`と`dnorm()`の場合、機能によって分布をサブセット化することになります。乱数を発生させる、確率密度を書きたい場合、まず思い浮かべるのは「どの分布にするのか」と考える。そうなると、思考順にあわせて、分布によって機能をサブセット化できる`norm_r()`と`norm_d()`のほうがよいのかもしれません。

* `***_r`：`norm_r()`、`binom_r()`、`unif_r()`、`exp_r()`。
* `***_d`：`norm_d()`、`binom_d()`、`unif_d()`、`exp_d()`。

### 15.4 条件付きの実行

#### 練習問題1 `if`とは`ifelse()`の違いは何か。

`if`は単一の条件を論理判定し、`ifelse()`は、各ベクトルの要素を論理判定します。`if`はベクトル化されていない関数なので、ベクトルをインプットするとエラーが表示され、最初の要素だけ使われ、誤った出力が表示されます。

```text
f <- function(x){
  if(x < 10){
    x
    }else{
    10 * x
  }
}

f(5)
[1] 5

f(1:10)
 [1]  1  2  3  4  5  6  7  8  9 10
 警告メッセージ: 
 if (x < 10) { で:   条件が長さが 2 以上なので、最初の 1 つだけが使われます 
```

なので、それをベクトル化したければ、関数内部でベクトル化の処理を施すか、`purrr::map_**()`などで`f`をベクトル化します。

```text
x <- 1:10
ifelse(x < 10, x, 10*x)
 [1]   1   2   3   4   5   6   7   8   9 100
 
 map_dbl(x, f) 
 [1]   1   2   3   4   5   6   7   8   9 100
```

#### 練習問題2 時間に応じて、「おはよう」、「こんにちは」、または「こんばんは」と言う挨拶関数を作成しなさい。

```text
say_hello <- function(time) {
  if(!is.POSIXct(time))
    stop("x must be POSIXct")

  hour <- lubridate::hour(time)
  if (hour < 12) {
    print("good morning")
  } else if (hour < 17) {
    print("good afternoon")
  } else {
    print("good evening")
  }
}

say_hello("2019-06-30 09:29:22") # deny char
 say_hello("2019-06-30 09:29:22") でエラー: x must be POSIXct

say_hello(ymd("2019-06-30 09:29:22")) # deny Date
 say_hello(ymd("2019-06-30 09:29:22")) でエラー: x must be POSIXct
 追加情報:  警告メッセージ: 
All formats failed to parse. No formats found. 

say_hello(ymd_hms("2019-06-30 09:29:22")) # ok
[1] "good morning"
```

#### 練習問題3 `fizzbuzz()`関数を実装しなさい。入力は単一のスカラを想定。

```text
fizzbuzz <- function(x) {
  stopifnot(length(x) == 1)
  stopifnot(is.numeric(x))
  if (!(x %% 3) && !(x %% 5)) {
    "fizzbuzz"
  } else if (!(x %% 3)) {
    "fizz"
  } else if (!(x %% 5)) {
    "buzz"
  } else {
    x
  }
}

fizzbuzz(3)
[1] "fizz"

fizzbuzz(5)
[1] "buzz"

fizzbuzz(15)
[1] "fizzbuzz"
```

ベクトルの入力を許す場合はこんな感じ。

```text
for (i in 1:15) {
  if (i %% 3 == 0 & i %% 5 == 0) {cat("FizzBuzz")}
  else if (i %% 3 == 0) {cat("Fizz")}
  else if (i %% 5 == 0) {cat("Buzz")}
  else cat(i)
  cat("\n")
}

1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
```

#### 練習問題4 このネストされたif-elseステートメントのセットを単純化するために、`cut()`はどのように使用できるか？

```text
if (temp <= 0) {
  "freezing"
} else if (temp <= 10) {
  "cold"
} else if (temp <= 20) {
  "cool"
} else if (temp <= 30) {
  "warm"
} else {
  "hot"
}
```

`cut()`を使用することで、ネストさせなくても区間で区切ることが可能なので、より直感的でわかりやすいコードになります。また、`if(cond)`の場合、ベクトル化されていないので、複数の値を区切る場合には、ベクトル化が必要になります。

```text
temp <- seq(-10, 50, by = 1)
cut(temp, c(-Inf, 0, 10, 20, 30, Inf),
    right = TRUE,
    labels = c("freezing", "cold", "cool", "warm", "hot")
)

 [1] freezing freezing freezing freezing freezing freezing freezing
 [8] freezing freezing freezing freezing cold     cold     cold    
[15] cold     cold     cold     cold     cold     cold     cold    
[22] cool     cool     cool     cool     cool     cool     cool    
[29] cool     cool     cool     warm     warm     warm     warm    
[36] warm     warm     warm     warm     warm     warm     hot     
[43] hot      hot      hot      hot      hot      hot      hot     
[50] hot      hot      hot      hot      hot      hot      hot     
[57] hot      hot      hot      hot      hot     
Levels: freezing cold cool warm hot
```

#### 練習問題5 `switch()`で数値を使用するとどうなるのか？

要素に対応する値が返されますが、整数以外の数値は、切り捨てることに注意してください。

```text
fruit <- function(x){
  switch(x, "apple", "banana", "orange")
}

fruit(1)
[1] "apple"
fruit(3)
[1] "orange"
```

#### 練習問題6 この`switch()`の呼び出しは何をするのか？

この`switch()`関数は、一致した名前の欠損していない引数値を返します。`a = ,`のような欠損値のある引数に遭遇すると、この場合は欠損値ではない次の引数の値を返します。したがって、`"e"`のようにそもそも、どこともマッチしないものは何も返しません。

```text
x <- "e"
switch(x,
  a = ,
  b = "ab",
  c = ,
  d = "cd"
)
```

一致しないなら`NULL`を返し、欠損値の部分にも返り値を記載して明確にしておきます。

```text
switch(x,
  a = "ab",
  b = "ab",
  c = "cd",
  d = "cd",
  NULL 
)

NULL
```

### 15.5 関数の引数

#### 練習問題1 `commas(letters, collapse = "-")`はどのように機能しますか。

このコードは、文字列ベクトルをコンマで結合して1つの値にします。

```text
commas <- function(...) {
  str_c(..., collapse = ", ")
}

commas(letters)
[1] "a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z"
```

`commas(letters, collapse = "-")`を入力すると、エラーが返されます。これは関数内部で定義している`collapse = ", "`と、`commas`で定義している`collapse = "-"`が衝突しているからです。

```text
commas(letters, collapse = "-")
 str_c(..., collapse = ", ") でエラー: 
   仮引数 "collapse" が複数の実引数にマッチしました 
```

このような引数の衝突をさけるために、関数で柔軟に指定できるように書き換えます。

```text
binding <- function(..., collapse = ",") {
  str_c(..., collapse = collapse)
}

binding(letters, collapse = ",")
[1] "a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z"

binding(letters, collapse = "-")
[1] "a-b-c-d-e-f-g-h-i-j-k-l-m-n-o-p-q-r-s-t-u-v-w-x-y-z"
```

#### 練習問題2 `rule("Title", pad = "-+")`は、なぜ現在動作しないのか？`pad`引数で調整しなさい。

この`rule()`関数の`pad`は、目的の幅からタイトルの長さと5文字を引いた長さを引いた数に等しい回数だけ複製します。これは暗黙のうちに`pad`が1文字だけであると仮定します。2つの文字があった場合、出力は2倍長くなります。

```text
rule <- function(..., pad = "-") {
  title <- paste0(...)
  width <- getOption("width") - nchar(title) - 5
  cat(title, " ", str_dup(pad, width), "\n", sep = "")
}

rule("Output")
Output ------------------------------------------------------

rule("Title", pad = "-+")Title -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
+-+-+-+-+-+-+-+-+-+-+-+-+
```

#### 練習問題3 `mean()`の`trim`は何をするか？いつ使用するか。

`trim`は、平均を計算する前に、ベクトルの両端からの一部をトリミングします。これは、外れ値に対してロバストな中心傾向を計算するのに役立ちます。

```text
set.seed(1234)
x <- c(10000000, rnorm(100,0,1), 10000000)

mean(x)
[1] 196078.3

mean(x, trim = 0.1)
[1] -0.1902079
```

#### 練習問題4 `cor()`の`method`は`c("pearson", "kendall", "spearman")`です。どういう意味なのか？デフォルトでどの値が使われるのか？

これは、`method`がこれら3つの値のいずれかを取ることができることを意味します。最初の値である`"pearson"`がデフォルトで使用されます。

```text
args(cor)
function (x, y = NULL, use = "everything", method = c("pearson", "kendall", "spearman")) 
```

### 15.6 戻り値

{% hint style="info" %}
練習問題はありません。
{% endhint %}

### 15.7 環境

{% hint style="info" %}
練習問題はありません。
{% endhint %}

