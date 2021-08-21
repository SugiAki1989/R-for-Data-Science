# 8章 readrによるデータインポート

### 8.0 ライブラリーの読み込み

```text
library("tidyverse")
```

### 8.1 はじめに

{% hint style="info" %}
練習問題はありません
{% endhint %}

### 8.2 作業をはじめるにあたって

#### 練習問題1 フィルードが「`｜`」で区切られたファイルを読み込むには、どの関数を使うのか。

`read_delim()`を使用します。delimitationの頭文字の引数である`delim`に区切り文字を指定します。詳しくは[こちら](https://qiita.com/matsuou1/items/78fedad2766ce4bbf861)に記載されています。

```text
read_delim(file, delim = "|")
```

#### 練習問題2 `read_csv()`と`read_tsv()`の共通引数は何か。

`formals()`で引数の一覧を取得し、共通部分を`intersect()`で抽出します。

```text
intersect(names(formals(read_csv)), 
          names(formals(read_tsv)))

[1] "file"            "col_names"       "col_types"       "locale"         
[5] "na"              "quoted_na"       "quote"           "comment"        
[9] "trim_ws"         "skip"            "n_max"           "guess_max"      
[13] "progress"        "skip_empty_rows"
```

#### 練習問題3 `read_fwf()`で最も重要な引数は何か。

`read_fwf()`の重要な引数は`col_positions`です。「固定幅フォーマット」のデータ列の開始位置と終了位置を関数に指定することです。

#### 練習問題4 `"x,y\n1,'a,b'"`という文字列を読み込むにはどうすればよいか。

`read_csv()`でそのまま読み込むと、文字列のカンマで区切られてしまい、正しく読み込めません。したがって、`quote`で`'`を指定し、文字列を認識させます。

```text
x <- "x,y\n1,'a,b'"
read_csv(x)
 警告:  1 parsing failure.
row col  expected    actual         file
  1  -- 2 columns 3 columns literal data

# A tibble: 1 x 2
      x y    
  <dbl> <chr>
1     1 'a   

read_csv(x, quote = "'")
# A tibble: 1 x 2
      x y    
  <dbl> <chr>
1     1 a,b  
```

#### 練習問題5 下記の文字列のどこがまいずのかを示しなさい。

```text
read_csv("a,b\n1,2,3\n4,5,6")
read_csv("a,b,c\n1,2\n1,2,3,4")
read_csv("a,b\n\"1")
read_csv("a,b\n1,2\na,b")
read_csv("a;b\n1;3")
read_csv2("a;b\n1;3")
```

この場合、カラム名と値の列数が一致しておらず、カラム数に合わせて、列がなくなってしまいます。

```text
read_csv("a,b\n1,2,3\n4,5,6")
 警告:  2 parsing failures.
row col  expected    actual         file
  1  -- 2 columns 3 columns literal data
  2  -- 2 columns 3 columns literal data

# A tibble: 2 x 2
      a     b
  <dbl> <dbl>
1     1     2
2     4     5
```

データ内の列数とカラムの列数と一致していません。1行目には2つの値しかないので、column `c`はmissingに設定され、2行目には追加の値があり、その値はドロップされます。

```text
read_csv("a,b,c\n1,2\n1,2,3,4")
 警告:  2 parsing failures.
row col  expected    actual         file
  1  -- 3 columns 2 columns literal data
  2  -- 3 columns 4 columns literal data

# A tibble: 2 x 3
      a     b     c
  <dbl> <dbl> <dbl>
1     1     2    NA
2     1     2     3
```

`"`で`1`を閉じている部分かな…よくわからない。

```text
read_csv("a,b\n\"1")
 警告:  2 parsing failures.
row col                     expected    actual         file
  1  a  closing quote at end of file           literal data
  1  -- 2 columns                    1 columns literal data

# A tibble: 1 x 2
      a b    
  <dbl> <chr>
1     1 NA   
```

これを読み込むには、`read_delim()`で`delim = ";"`を指定します。

```text
read_csv ("a;b\n1;3")
# A tibble: 1 x 1
  `a;b`
  <chr>
1 1;3 
 
read_delim("a;b\n1;3", delim = ";")
# A tibble: 1 x 2
      a     b
  <dbl> <dbl>
1     1     3
```

### 8.3 ベクトルをパースする

#### 練習問題1 `locale()`で最も重要な引数は何か。

`locale()`には、以下を設定するための引数があります。

* 日付時刻形式：`date_names`、`date_format`、および`time_format`
* タイムゾーン： `tz`
* マーク：`decimal_mark`、`grouping_mark`
* エンコーディング： `encoding`

#### 練習問題2 `decimal_mark`を`,`に設定すると`grouping_mark`に何が起こるのか。

`decimal_mark`が`,`に設定されている場合、グループ化マークは`.`が設定されます。同じにはできません。

```text
locale(decimal_mark = ".", grouping_mark = ".")
Error: `decimal_mark` and `grouping_mark` must be different
```

`decimal_mark = ","`は下記の設定となります。

```text
locale(decimal_mark = ",")
<locale>
Numbers:  123.456,78
Formats:  %AD / %AT
Timezone: UTC
Encoding: UTF-8
<date_names>
Days:   Sunday (Sun), Monday (Mon), Tuesday (Tue), Wednesday (Wed),
        Thursday (Thu), Friday (Fri), Saturday (Sat)
Months: January (Jan), February (Feb), March (Mar), April (Apr), May (May),
        June (Jun), July (Jul), August (Aug), September (Sep),
        October (Oct), November (Nov), December (Dec)
AM/PM:  AM/PM
```

`grouping_mark = ","`は下記の設定となります。

```text
locale(grouping_mark = ",")
<locale>
Numbers:  123,456.78
Formats:  %AD / %AT
Timezone: UTC
Encoding: UTF-8
<date_names>
Days:   Sunday (Sun), Monday (Mon), Tuesday (Tue), Wednesday (Wed),
        Thursday (Thu), Friday (Fri), Saturday (Sat)
Months: January (Jan), February (Feb), March (Mar), April (Apr), May (May),
        June (Jun), July (Jul), August (Aug), September (Sep),
        October (Oct), November (Nov), December (Dec)
AM/PM:  AM/PM
```

#### 練習問題3 `date_format`と`time_format`の`locale()`の役割について教えなさい。

デフォルトの日付と時刻のフォーマットを提供します。

```text
locale("ja")
<locale>
Numbers:  123,456.78
Formats:  %AD / %AT
Timezone: UTC
Encoding: UTF-8
<date_names>
Days:   日曜日 (日), 月曜日 (月), 火曜日 (火), 水曜日 (水), 木曜日 (木),
        金曜日 (金), 土曜日 (土)
Months: 1月, 2月, 3月, 4月, 5月, 6月, 7月, 8月, 9月, 10月, 11月, 12月
AM/PM:  午前/午後
```

```text
parse_date("2019年6月18日", "%Y年 %m月 %d日", 
           locale = locale("ja"))
[1] "2019-06-18"

parse_date("18日6月2019年", "%d日 %m月 %Y年", 
           locale = locale("ja"))
[1] "2019-06-18"
```

#### 練習問題4 米国外に住んでいる場合は、最もよく読むファイルの種類の設定をカプセル化した新しいロケールオブジェクトを作成しなさい。

ここでは日本を例に考える。`"%Y年%m月%d日"`が一般的なフォーマットなので、これを読む込むための`jp_locale`を設定します。

```text
jp_locale <- locale(date_format = "%Y年%m月%d日")
parse_date("2019年6月18日", locale = jp_locale)
[1] "2019-06-18"
```

#### 練習問題5 `read_csv()`と`read_csv2()`の違いについて教えなさい。

`read_csv()`はカンマを区切り文字としており、`read_csv2()`は、セミコロン\(`;`\)を使用します。

#### 練習問題6 ヨーロッパまたはアジアで使用されている最も一般的なエンコーディングは何ですか。

標準はUTF-8、日本語独特なものとして、JIS X 0208、シフトJIS、ISO-2022-JPなどがある。エンコーディングの詳細は、いまいちわかってません。

#### 練習問題7 次の文字列を解析する正しいフォーマット文字列を作りなさい。

```text
d1 <- "January 1, 2010"
parse_date(d1, "%B %d, %Y")
[1] "2010-01-01"

d2 <- "2015-Mar-07"
parse_date(d2, "%Y-%b-%d")
[1] "2015-03-07"

d3 <- "06-Jun-2017"
parse_date(d3, "%d-%b-%Y")
[1] "2017-06-06"

d4 <- c("August 19 (2015)", "July 1 (2015)")
parse_date(d4, "%B %d (%Y)")
[1] "2015-08-19" "2015-07-01"

d5 <- "12/30/14" # Dec 30, 2014
parse_date(d5, "%m/%d/%y")
[1] "2014-12-30"

t1 <- "1705"
parse_time(t1, "%H%M")
17:05:00

t2 <- "11:15:10.12 PM"
parse_time(t2, "%H:%M:%OS %p")
23:15:10.12
```

### 8.4 ファイルをパースする

{% hint style="info" %}
練習問題はありません
{% endhint %}

### 8.5 ファイルへの書き出し

{% hint style="info" %}
練習問題はありません
{% endhint %}

### 8.6 他の種類のデータ

{% hint style="info" %}
練習問題はありません
{% endhint %}

