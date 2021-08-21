# 0章 はじめに

### Rではじめるデータサイエンス 

この本は[Rではじめるデータサイエンス](https://www.oreilly.co.jp/books/9784873118147/)の演習問題の回答を記載したものです。したがって、章立ては日本語版に準じています。

Rではじめるデータサイエンスの英語版は [https://r4ds.had.co.nz](https://r4ds.had.co.nz/) より無料で閲覧が可能です。

![](.gitbook/assets/51p6-f0gthl._sx388_bo1-204-203-200_.jpg)

### 注意事項

基本的には、`{tidyverse}`を使ったコーディングスタイルですが、引数のありなしや、同じ動作を行う場合であっても、異なる記述形式でコーディングしている箇所があります。

また、常に最適化されたコードを載せているわけでもなく、場合によって意図的に`{tidyverse}`を使った非効率なコードを記述している場合もあります。その点は、ご自身で改善されることを望みます。

翻訳に関しては杜撰かつ解釈の取り違いよる回答の誤りもあるかと思います。疑問点があれば、下記参考文献一覧より原著をあたってください。

### 動作環境

下記の環境のもと、動作することを確認しています。

```text
sessionInfo()
R version 3.5.3 (2019-03-11)
Platform: x86_64-apple-darwin15.6.0 (64-bit)
Running under: macOS Mojave 10.14.5

Matrix products: default
BLAS: /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libBLAS.dylib
LAPACK: /Library/Frameworks/R.framework/Versions/3.5/Resources/lib/libRlapack.dylib

locale:
[1] ja_JP.UTF-8/ja_JP.UTF-8/ja_JP.UTF-8/C/ja_JP.UTF-8/ja_JP.UTF-8

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base     

other attached packages:
[1] forcats_0.4.0        stringr_1.4.0        dplyr_0.8.1          purrr_0.3.2         
[5] readr_1.3.1          tidyr_0.8.3          tibble_2.1.2         ggplot2_3.1.1       
[9] tidyverse_1.2.1.9000

loaded via a namespace (and not attached):
 [1] Rcpp_1.0.1       cellranger_1.1.0 pillar_1.4.1     compiler_3.5.3   dbplyr_1.4.0     plyr_1.8.4      
 [7] tools_3.5.3      lubridate_1.7.4  jsonlite_1.6     nlme_3.1-137     gtable_0.3.0     lattice_0.20-38 
[13] pkgconfig_2.0.2  rlang_0.3.4      reprex_0.3.0     cli_1.1.0        DBI_1.0.0        rstudioapi_0.10 
[19] haven_2.1.0      withr_2.1.2      xml2_1.2.0       httr_1.4.0       fs_1.3.1         generics_0.0.2  
[25] hms_0.4.2        grid_3.5.3       tidyselect_0.2.5 glue_1.3.1       R6_2.4.0         readxl_1.3.1    
[31] modelr_0.1.4     magrittr_1.5     backports_1.1.4  scales_1.0.0     rvest_0.3.4      assertthat_0.2.1
[37] colorspace_1.4-1 stringi_1.4.3    lazyeval_0.2.2   munsell_0.5.0    broom_0.5.2      crayon_1.3.4
```

### 参考文献

[Wickham, Hadley, and Garrett Grolemund. 2017. _R for Data Science: Import, Tidy, Transform, Visualize, and Model Data_. 1st ed. O’Reilly Media.](https://r4ds.had.co.nz/)

[R for Data Science: Exercise Solutions. 2019. _Jeffrey B. Arnold_](https://jrnold.github.io/r4ds-exercise-solutions/)

[Hadley, Grolemund（2017）「Rではじめるデータサイエンス」（黒川 利明・大橋 真也訳） O'Reilly Japan．](https://www.oreilly.co.jp/books/9784873118147/)

[  
](https://jrnold.github.io/r4ds-exercise-solutions/)

