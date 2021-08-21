# 11章 stringrによる文字列

{% hint style="warning" %}
正規表現に明るくないため、この章の正規表現の部分については、[r4ds-exercise-solutions/strings.html](https://jrnold.github.io/r4ds-exercise-solutions/strings.html)の回答を、基本的にはそのまま記載しています。
{% endhint %}

### 11.0 ライブラリの読み込み

```text
library("tidyverse")
library("stringi")
```

### 11.1 はじめに

{% hint style="info" %}
練習問題はありません
{% endhint %}

### 11.2 文字列の基本

#### 練習問題1 `paste()`と`paste0()`の機能の違いは何ですか？

`paste()`は、デフォルトで文字列をスペースで区切りますが、`paste0()`はスペースで区切りません。

```text
paste("hoge", "fuga", "piyo")
[1] "hoge fuga piyo"

paste0("hoge", "fuga", "piyo")
[1] "hogefugapiyo"
```

`str_c()`は`paste0()`は似ています。

```text
str_c("hoge", "fuga", "piyo")
[1] "hogefugapiyo"
```

#### 練習問題2 `str_c()`の`sep`と`collapse`の違いを説明してください

`sep`は、文字の間に挿入される文字列であり、`collapse`は文字ベクトルを分離する際に使用されるの任意の文字列です。

#### 練習問題3 `str_length()`と`str_sub()`を使用して文字列から中間文字を抽出します。文字列の文字数が偶数の場合どうしますか。

ここでは偶数の場合、中間文字の先頭をとります。

```text
x <- c("a", "ab", "abc", "abcd", "abcde", "abcdef", "abcdefg")
len <- str_length(x)
m <- ceiling(len / 2)
str_sub(x, m, m)
[1] "a" "a" "b" "b" "c" "c" "d"
```

#### 練習問題4 `str_wrap()`はいつ使用しますか。

`str_wrap()`は、テキストが一定の幅に収まるように折り返します。これは、長い文字列を折り返して植字する場合に便利です。

```text
x <- str_c(
  "The Economist is an English-language weekly magazine-format newspaper owned by the Economist Group and edited at offices in London.",
  "Continuous publication began under its founder James Wilson in September 1843.",
  "In 2015, its average weekly circulation was a little over 1.5 million, about half of which were sold in the United States."
)
cat(str_wrap(x, width = 80))

The Economist is an English-language weekly magazine-format newspaper owned by
the Economist Group and edited at offices in London.Continuous publication began
under its founder James Wilson in September 1843.In 2015, its average weekly
circulation was a little over 1.5 million, about half of which were sold in the
United States.
```

#### 練習問題5 `str_trim()`は何か。逆の操作は何か。

`str_trim()`は文字列から空白を削除します。

```text
x <- " hoge fuga piyo "

str_trim(x)
[1] "hoge fuga piyo"

str_trim(x, side = "left")
[1] "hoge fuga piyo "

str_trim(x, side = "right")
[1] " hoge fuga piyo"
```

逆の操作は`str_pad()`です。

```text
str_pad(x, 25, side = "both")
[1] "     hoge fuga piyo      "
str_pad(x, 25, side = "right")
[1] " hoge fuga piyo          "
str_pad(x, 25, side = "left")
[1] "          hoge fuga piyo "
```

#### 練習問題6 `c("a", "b", "c")`を文字列a, b and cに変換する関数を作成します。

```text
str_commasep <- function(x, delim = ",") {
 n <- length(x)
 if (n == 0) {
   ""
 } else if (n == 1) {
   x
 } else if (n == 2) {
   str_c(x[[1]], "and", x[[2]], sep = " ")
 } else {
   not_last <- str_c(x[seq_len(n - 1)], delim)
   last <- str_c("and", x[[n]], sep = " ")
   tmp <- str_c(c(not_last, last), collapse = " ")
   str_replace(tmp, ", and", " and")
  }
}
str_commasep("")
[1] ""
str_commasep("a")
[1] "a"
str_commasep(c("a", "b"))
[1] "a and b"
str_commasep(c("a", "b", "c"))
[1] "a, b and c"
str_commasep(c("a", "b", "c", "d"))
[1] "a, b, c and d"
```

### 11.3 正規表現でパターンマッチ

`str_view()は`RStudioのViewerの部分でマッチしている箇所を確認できます。Viewerに表示される出力を参考までに記載しています。

#### 練習問題1 "\"が"\"、"\\"、"\\\"とマッチしないのはなぜか。

* `"\"`：Rは次の文字をエスケープします。
* `"\\"`：正規表現の\に解決され、正規表現の次の文字をエスケープします。
* `"\\\"`注：最初の2つのバックスラッシュは正規表現のリテラルバックスラッシュに解決され、3番目のバックスラッシュは次の文字をエスケープします。正規表現では、これはエスケープ文字をエスケープします。

#### 練習問題2 文字列「""\」とはどうマッチするのか。

```text
str_view("\"'\\", "\"'\\\\", match = TRUE)

"'\
```

#### 練習問題3 正規表現"......"は、どのようなパターンとマッチするのか。

```text
str_view(c(".a.b.c", ".a.b", "....."), c("\\..\\..\\.."), match = TRUE)

.a.b.c
```

#### 練習問題3 文字列"$^$"はどのようにマッチしますか

```text
str_view(c("$^$", "ab$^$sfas"), "^\\$\\^\\$$", match = TRUE)

$^$
```

#### 練習問題4 一般的な単語のコーパスを考慮して、以下の`stringr::words`すべての単語を検索する正規表現を作成します。

* 「y」で始まる。

```text
str_view(stringr::words, "^y", match = TRUE)

year
yes
yesterday
yet
you
young
```

* 「x」で終わる。

```text
str_view(stringr::words, "x$", match = TRUE)

box
sex
six
tax
```

* ちょうど3文字の長さ。

```text
str_view(stringr::words, "^...$", match = TRUE)

act
add
age
（略）
yes
yet
you
```

* 7文字以上。

```text
str_view(stringr::words, ".......", match = TRUE)

absolute
account
achieve
（略）
whether
without
yesterday
```

**練習問題5 次のワードを探す正規表現を作りなさい。**

* 母音で始まる。

```text
str_view(stringr::words, "^[aeiou]", match = TRUE)

a
able
about
（略）
upon
use
usual
```

* 子音だけが含まれる。

```text
str_view(stringr::words, "^[^aeiou]+$", match = TRUE)

by
dry
fly
mrs
try
why
```

* edで終わるが、eedではない。

```text
str_view(stringr::words, "^ed$|[^e]ed$", match = TRUE)

bed
hundred
red
```

* ingまたはizeで終わる。

```text
str_view(stringr::words, "i(ng|se)$", match = TRUE)

advertise
bring
during
(略)
sing
surprise
thing
```

#### 練習問題6 「cのあとを除いてeの前にi」という規則を検証しなさい。

```text
str_view(stringr::words, "(cei|[^c]ie)", match = TRUE)

achie
belie
brie
(略)
receive
tie
view
```

#### 練習問題7 「qのあとにはuが続く」という規則を検証しなさい。

```text
str_view(stringr::words, "q[^u]", match = TRUE)

ない
```

#### 練習問題8 イギリス英語にマッチする正規表現を作りなさい。

英語自体そこまで得意ではないの詳しく調べてませんが、一般的な場合では、これは困難だそうです。それ用の辞書を必要とする可能性があります。

#### 練習問題9 自分の国の電話番号の正規表現を作りなさい。

```text
x <- c("123-4567-7890", "1235-2351")
str_view(x, "\\d\\d\\d-\\d\\d\\d\\d-\\d\\d\\d\\d")
str_view(x, "[0-9][0-9][0-9]-[0-9][0-9][0-9][0-9]-[0-9][0-9][0-9][0-9]")

123-4567-7890
```

#### 練習問題10 "?"、"+"、"\*"と等価な{m,n}を作りなさい。

正規表現に明るくないので、質問の意味がイマイチわかりませんでした。[r4ds-exercise-solutio](https://jrnold.github.io/r4ds-exercise-solutions/strings.html)の回答をそのまま記載しておきます。

```text
x <- "1888 is the longest year in Roman numerals: MDCCCLXXXVIII"
str_view(x, "CC?")


str_view(x, "CC?")
str_view(x, "CC{0,1}")

str_view(x, "CC+")
str_view(x, "CC{1,}")

str_view_all(x, "C[LX]+")
str_view_all(x, "C[LX]{0,1}")

str_view_all(x, "C[LX]*")
str_view_all(x, "C[LX]{0,}")

```

#### 練習問題11 これらの正規表現が一致するものを言葉で説明してください。

* ^.\*$

任意の文字列に一致します。

* "\{.+\}"

少なくとも1文字を囲む中括弧を含む任意の文字列に一致します。

* \d{4}-\d{2}-\d{2}

4桁の数字の後にハイフン、2桁の数字、ハイフン、さらに2桁の数字が一致します。

* "\\{4}"

4つのバックスラッシュと一致するとおもわれます。

#### 練習問題12 以下のすべての単語を見つけるための正規表現を作成しなさい。

* 3つの子音から始める。

```text
str_view(words, "^[^aeiou]{3}", match = TRUE)

Christ
Christmas
dry
(略)
try
type
why
```

* 連続して3つ以上の母音を持つ。

```text
str_view(words, "[aeiou]{3,}", match = TRUE)

beauty
obvious
previous
quiet
serious
various
```

* 連続して2つ以上の母音 - 子音ペアを持つ。

```text
str_view(words, "([aeiou][^aeiou]){2,}", match = TRUE)

absolute
agent
along
(略)
visit
water
woman
```

**練習問題13** [**https://regexcrossword.com/challenges/**](https://regexcrossword.com/challenges/)**で初心者用の正規表現クロスワードを解く**

自分で遊んでみてください！

**練習問題14 これらの表現が一致するものを言葉で説明してください。**

* `(.)\1\1` 

同じ文字が3回続けて表示されます。

* `"(.)(.)\2\1"`

文字のペアの後に同じ文字のペアが逆の順序で続きます。

* `(..)\1`

任意の2文字が繰り返されます。

* `"(.).\1.\1"`

任意の文字、元の文字、他の文字、元の文字が続く文字。

* `"(.)(.)(.).*\3\2\1"`

3文字の後に任意の種類の0個以上の文字が続き、その後に同じ3文字が続きますが、逆の順序です。

#### 練習問題15 以下の単語に一致するように正規表現を作成します。

* 同じ文字で始まり、同じ文字で終わる。

```text
str_view(stringr::words, "^(.)((.*\\1$)|\\1?$)", match = TRUE)

a
america
area
(略)
trust
window
yesterday
```

* 繰り返された文字のペアを含む。

```text
str_view(words, "([A-Za-z][A-Za-z]).*\\1", match = TRUE)

appropriate
church
condition
(略)
therefore
understand
whether
```

* 少なくとも3つの場所で繰り返される1つの文字を含む。

```text
str_subset(str_to_lower(words), "([a-z]).*\\1.*\\1")

[1] "appropriate" "available"   "believe"     "between"     "business"   
[6] "degree"      "difference"  "discuss"     "eleven"      "environment"
[11] "evidence"    "exercise"    "expense"     "experience"  "individual" 
[16] "paragraph"   "receive"     "remember"    "represent"   "telephone"  
[21] "therefore"   "tomorrow"
```

### 11.4 ツール

#### 練習問題1 単一の正規表現と複数のstr\_detect\(\)の組み合わせの両方を使用してそれを解決しなさい。

* xで始まるまたは終わる単語をすべて探す。

```text
words[str_detect(words, "^x|x$")]
[1] "box" "sex" "six" "tax"

start_with_x <- str_detect(words, "^x")
end_with_x <- str_detect(words, "x$")
words[start_with_x | end_with_x]
[1] "box" "sex" "six" "tax"
```

* 母音で始まり子音で終わる単語をすべて見つける。

```text
str_subset(words, "^[aeiou].*[^aeiou]$") %>% head()
[1] "about"   "accept"  "account" "across"  "act"     "actual"

start_with_vowel <- str_detect(words, "^[aeiou]")
end_with_consonant <- str_detect(words, "[^aeiou]$")
words[start_with_vowel & end_with_consonant] %>% head()
[1] "about"   "accept"  "account" "across"  "act"     "actual"
```

* それぞれ異なる母音を少なくとも1つ含む単語はあるか。

```text
pattern <-
  cross(rerun(5, c("a", "e", "i", "o", "u")),
    .filter = function(...) {
      x <- as.character(unlist(list(...)))
      length(x) != length(unique(x))
    }
  ) %>%
  map_chr(~ str_c(unlist(.x), collapse = ".*")) %>%
  str_c(collapse = "|")
  
str_subset("aseiouds", pattern)
[1] "aseiouds"

str_subset(words, pattern)
character(0)
```

**練習問題2 母音の数が多いのはなんの単語ですか？母音の割合が最も高いのはどの単語ですか。**

```text
vowels <- str_count(words, "[aeiou]")
words[which(vowels == max(vowels))]

[1] "appropriate" "associate"   "available"   "colleague"   "encourage"  
[6] "experience"  "individual"  "television"

prop_vowels <- str_count(words, "[aeiou]") / str_length(words)
words[which(prop_vowels == max(prop_vowels))]
[1] "a"
```

**練習問題3 前の例では、正規表現が「flickered」と一致していることに気付いたかもしれないが、これは色ではない。正規表現を修正しなさい。**

```text
colours <- c("red", "orange", "yellow", "green", "blue", "purple")
colour_match <- str_c(colours, collapse = "|")

colour_match2 <- str_c("\\b(", str_c(colours, collapse = "|"), ")\\b")
colour_match2
[1] "\\b(red|orange|yellow|green|blue|purple)\\b"

more2 <- sentences[str_count(sentences, colour_match) > 1]
str_view_all(more2, colour_match2, match = TRUE
```

#### 練習問題4 ハーバードのセンテンスデータから、次を抽出しなさい。

* 各文の最初の単語

```text
str_extract(sentences, "[a-zA-Z]+") %>% head()
[1] "The"   "Glue"  "It"    "These" "Rice"  "The"
```

* すべての単語がで終わりますing。

```text
pattern <- "\\b[A-Za-z]+ing\\b"
sentences_with_ing <- str_detect(sentences, pattern)
unique(unlist(str_extract_all(sentences[sentences_with_ing], pattern))) %>%
  head()
[1] "spring"  "evening" "morning" "winding" "living"  "king"
```

* すべて複数形

```text
unique(unlist(str_extract_all(sentences, "\\b[A-Za-z]{3,}s\\b"))) %>%
  head()
#> [1] "planks" "days"   "bowls"  "lemons" "makes"  "hogs"
```

#### 練習問題5 「one」、「two」、「three」などの「number」の後に続くすべての単語を見つけなさい。

```text
numword <- "(one|two|three|four|five|six|seven|eight|nine|ten) +(\\S+)"
sentences[str_detect(sentences, numword)] %>%
  str_extract(numword)
 [1] "ten served"    "one over"      "seven books"   "two met"      
 [5] "two factors"   "one and"       "three lists"   "seven is"     
 [9] "two when"      "one floor."    "ten inches."   "one with"     
[13] "one war"       "one button"    "six minutes."  "ten years"    
[17] "one in"        "ten chased"    "one like"      "two shares"   
[21] "two distinct"  "one costs"     "ten two"       "five robins." 
[25] "four kinds"    "one rang"      "ten him."      "three story"  
[29] "ten by"        "one wall."     "three inches"  "ten your"     
[33] "six comes"     "one before"    "three batches" "two leaves."
```

#### 練習問題6 アポストロフィを使った短縮形をさがしなさい。

```text
contraction <- "([A-Za-z]+)'([A-Za-z]+)"
sentences[str_detect(sentences, contraction)] %>%
  str_extract(contraction)
 [1] "It's"       "man's"      "don't"      "store's"    "workmen's" 
 [6] "Let's"      "sun's"      "child's"    "king's"     "It's"      
[11] "don't"      "queen's"    "don't"      "pirate's"   "neighbor's"
```

#### 練習問題7 文字列内のすべてのスラッシュをバックスラッシュに置き換えます。

```text
str_replace_all("past/present/future", "/", "\\\\")
[1] "past\\present\\future"
```

#### 練習問題8 `replace_all()`を使用し、`str_to_lower()`のシンプル版を作成しなさい。

```text
replacements <- c(
  "A" = "a", "B" = "b", "C" = "c", "D" = "d", "E" = "e",
  "F" = "f", "G" = "g", "H" = "h", "I" = "i", "J" = "j",
  "K" = "k", "L" = "l", "M" = "m", "N" = "n", "O" = "o",
  "P" = "p", "Q" = "q", "R" = "r", "S" = "s", "T" = "t",
  "U" = "u", "V" = "v", "W" = "w", "X" = "x", "Y" = "y",
  "Z" = "z"
)
lower_words <- str_replace_all(words, pattern = replacements)
head(lower_words)
[1] "a"        "able"     "about"    "absolute" "accept"   "account"
```

#### 練習問題9 `words`の最初と最後の文字を入れ替えなさい

```text
swapped <- str_replace_all(words, "^([A-Za-z])(.*)([a-z])$", "\\3\\2\\1")
intersect(swapped, words)
 [1] "a"          "america"    "area"       "dad"        "dead"      
 [6] "lead"       "read"       "depend"     "god"        "educate"   
[11] "else"       "encourage"  "engine"     "europe"     "evidence"  
[16] "example"    "excuse"     "exercise"   "expense"    "experience"
[21] "eye"        "dog"        "health"     "high"       "knock"     
[26] "deal"       "level"      "local"      "nation"     "on"        
[31] "non"        "no"         "rather"     "dear"       "refer"     
[36] "remember"   "serious"    "stairs"     "test"       "tonight"   
[41] "transport"  "treat"      "trust"      "window"     "yesterday"
```

#### 練習問題10 文字列"apples, pears, and bananas"を個々のコンポーネントに分割します。

```text
x <- c("apples, pears, and bananas")
str_split(x, ", +(and +)?")[[1]]
[1] "apples"  "pears"   "bananas"
```

#### 練習問題11 " "よりもboundary\("word"\)で分割するほうがいいのはなぜか。

スペースで文字列を分割すると、句読点と単語がグループ化されます。

```text
sentence <- "The quick (“brown”) fox can’t jump 32.3 feet, right?"

str_split(sentence, " ")
[[1]]
[1] "The"       "quick"     "(“brown”)" "fox"       "can’t"     "jump"     
[7] "32.3"      "feet,"     "right?"

str_split(sentence, boundary("word"))
[[1]]
[1] "The"   "quick" "brown" "fox"   "can’t" "jump"  "32.3"  "feet"  "right"
```

#### 練習問題12 \(""\)空の文字列で分割するとどうなりますか？

文字列を個々の文字に分割します。

```text
str_split("ab. cd|agt", "")[[1]]
#>  [1] "a" "b" "." " " "c" "d" "|" "a" "g" "t"
```

### 11.5 他の種類のパターン

#### 練習問題1 "\"を探すために、regex\(\)とfixed\(\)とでそれぞれどうすればいいか。

```text
str_subset(c("a\\b", "ab"), "\\\\")
[1] "a\\b"

str_subset(c("a\\b", "ab"), fixed("\\"))
[1] "a\\b"
```

#### 練習問題2 `sentences`で利用されている5つの最も一般的な言葉は何ですか？

`str_extract_all()`を引数とともに使用すると、`boundary("word")`ですべての単語が抽出されます。

```text
tibble(word = unlist(str_extract_all(sentences, boundary("word")))) %>%
  mutate(word = str_to_lower(word)) %>%
  count(word, sort = TRUE) %>%
  head(5)
  
# A tibble: 5 x 2
  word      n
  <chr> <int>
1 the     751
2 a       202
3 of      132
4 to      123
5 and     118
```

### 11.6 正規表現の別の用途

{% hint style="info" %}
練習問題はありません
{% endhint %}

### 11.7 stringi

#### 練習問題1 以下を`stringi`関数で探しなさい。

* 単語数を数えます。

```text
stri_count_words(head(sentences))
[1] 8 8 9 9 7 7
```

* 重複した文字列を見つけます。

```text
stri_duplicated(c(
  "the", "brown", "cow", "jumped", "over",
  "the", "lazy", "fox"
))
#> [1] FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE
```

* ランダムなテキストを生成します。

```text
stri_rand_strings(4, 5)
#> [1] "5pb90" "SUHjl" "sA2JO" "CP3Oy"
```

#### 練習問題2 `stri_sort()`が整列に使う言語をどのように制御しますか？

`stri_sort(..., opts_collator=stri_opts_collator(locale = ...))`または`stri_sort(..., locale = ...)`のいずれかでソートするときに使用するロケールを設定できます。

```text
string1 <- c("hladny", "chladny")
stri_sort(string1, locale = "pl_PL")
[1] "chladny" "hladny"

stri_sort(string1, opts_collator = stri_opts_collator(locale = "pl_PL"))
[1] "chladny" "hladny"

stri_sort(string1, opts_collator = stri_opts_collator(locale = "sk_SK"))
[1] "hladny"  "chladny"

stri_sort(string1, locale = "sk_SK")
[1] "hladny"  "chladny"
```

`stri_opts_collator()`により、文字列のソート方法を細かく制御できます。

```text
string2 <- c("number100", "number2")
stri_sort(string2)
#> [1] "number100" "number2"
stri_sort(string2, opts_collator = stri_opts_collator(numeric = TRUE))
#> [1] "number2"   "number100"
```

