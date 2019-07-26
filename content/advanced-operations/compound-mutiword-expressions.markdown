---
title: Compound multi-word expressions
weight: 20
draft: false
---


```r
require(quanteda)
require(quanteda.corpora)
```

This corpus contains 6,000 Guardian news articles from 2012 to 2016.


```r
corp_news <- download('data_corpus_guardian')
```




```r
ndoc(corp_news)
```

```
## [1] 6000
```

```r
range(docvars(corp_news, 'date'))
```

```
## [1] "2012-01-02" "2016-12-31"
```

Unlike in earlier examples, we remove punctuations in `tokens_remove()` with `padding = TRUE` to keep the original positions of tokens. `[\\p{P}\\p{S}]` is a regular expression for punctuations or separators.


```r
toks_news <- tokens(corp_news) %>% 
    tokens_remove(stopwords('english'), padding = TRUE) %>% 
    tokens_remove('[\\p{P}\\p{S}]', valuetype = 'regex', padding = TRUE)
```

## Collocation analysis

Through collocation analysis, we can identify multi-word expressions that are very frequent in newspaper articles. One of the most common type of multi-word expressions is proper names, which we can select simply based on capitalization in English texts.


```r
toks_news_cap <- tokens_select(toks_news, 
                               pattern = '^[A-Z]',
                               valuetype = 'regex',
                               case_insensitive = FALSE, 
                               padding = TRUE)
head(toks_news_cap[[1]], 50)
```

```
##  [1] "London"    ""          ""          ""          ""         
##  [6] ""          ""          ""          ""          ""         
## [11] ""          ""          ""          ""          ""         
## [16] ""          ""          ""          "March"     ""         
## [21] ""          ""          ""          ""          ""         
## [26] "James"     "Randerson" ""          ""          ""         
## [31] ""          ""          ""          ""          ""         
## [36] "London"    ""          ""          ""          ""         
## [41] ""          ""          ""          ""          ""         
## [46] ""          ""          ""          ""          ""
```

```r
tstat_col_cap <- textstat_collocations(toks_news_cap, min_count = 10, tolower = FALSE)
head(tstat_col_cap, 20)
```

```
##           collocation count count_nested length    lambda         z
## 1       David Cameron   860            0      2  8.312460 150.05619
## 2        Donald Trump   774            0      2  8.483165 124.99787
## 3      George Osborne   362            0      2  8.803972 109.38983
## 4     Hillary Clinton   525            0      2  9.249932 104.32137
## 5            New York  1016            0      2 10.604025 101.76536
## 6       Islamic State   330            0      2  9.958312  99.70885
## 7         White House   478            0      2 10.067494  97.81579
## 8      European Union   348            0      2  8.394973  96.47134
## 9       Jeremy Corbyn   244            0      2  8.885667  92.41832
## 10      Boris Johnson   245            0      2  9.820289  86.13744
## 11     Bernie Sanders   394            0      2 10.057546  85.95220
## 12 Guardian Australia   237            0      2  6.481933  85.73425
## 13   Northern Ireland   204            0      2 10.025227  84.51173
## 14        Home Office   216            0      2  9.838012  80.06445
## 15        Ed Miliband   173            0      2 10.008368  79.61399
## 16       South Africa   172            0      2  7.724807  79.21351
## 17       Barack Obama   343            0      2  9.915703  79.18011
## 18           Ted Cruz   417            0      2 10.912504  79.00898
## 19       Black Friday   190            0      2  8.615140  78.21715
## 20     South Carolina   271            0      2  9.560902  78.07657
```

### Compound multi-word expressions

The result of collocation analysis is not only very interesting but useful: you can be use it to compound tokens. Compounding makes tokens less ambiguous and significantly improves quality of statistical analysis in the downstream. We will only compound strongly associated (p<0.005) multi-word expressions here by sub-setting `tstat_col_cap$collocation`.

{{% notice note %}}
Collocations are automatically recognized as multi-word expressions by `tokens_compound()` in *case-sensitive fixed pattern matching*. This is the fastest way to compound large numbers of multi-word expressions, but make sure that `tolower = FALSE` in `textstat_collocations()` to do this.
{{% /notice %}}


```r
toks_comp <- tokens_compound(toks_news, pattern = tstat_col_cap[tstat_col_cap$z > 3])
toks_news[['text7005']][370:450] # before compounding
```

```
##  [1] ""              ""              ""              "need"         
##  [5] ""              ""              "incriminate"   ""             
##  [9] ""              "anyone"        "else"          ""             
## [13] "Scotland"      ""              "investigation" ""             
## [17] ""              ""              "rejected"      "conspiracy"   
## [21] "theories"      ""              ""              "old"          
## [25] "boss"          ""              "Rupert"        "Murdoch"      
## [29] ""              ""              "deals"         ""             
## [33] "people"        "like"          ""              "last"         
## [37] "boss"          ""              "David"         "Cameron"      
## [41] ""              ""              ""              ""             
## [45] "George"        "Osborne"       ""              ""             
## [49] "keen"          ""              "recruit"       ""             
## [53] ""              ""              "Murdoch"       "apparatchik"  
## [57] ""              "despite"       ""              "resignation"  
## [61] ""              ""              "News"          ""             
## [65] ""              "World"         ""              ""             
## [69] "royal"         "hacking"       "case"          ""             
## [73] "Coulson"       ""              ""              ""             
## [77] ""              "innuendo"      ""              ""             
## [81] ""
```

```r
toks_comp[['text7005']][370:450] # after compounding
```

```
##  [1] ""               ""               "incriminate"    ""              
##  [5] ""               "anyone"         "else"           ""              
##  [9] "Scotland"       ""               "investigation"  ""              
## [13] ""               ""               "rejected"       "conspiracy"    
## [17] "theories"       ""               ""               "old"           
## [21] "boss"           ""               "Rupert_Murdoch" ""              
## [25] ""               "deals"          ""               "people"        
## [29] "like"           ""               "last"           "boss"          
## [33] ""               "David_Cameron"  ""               ""              
## [37] ""               ""               "George_Osborne" ""              
## [41] ""               "keen"           ""               "recruit"       
## [45] ""               ""               ""               "Murdoch"       
## [49] "apparatchik"    ""               "despite"        ""              
## [53] "resignation"    ""               ""               "News"          
## [57] ""               ""               "World"          ""              
## [61] ""               "royal"          "hacking"        "case"          
## [65] ""               "Coulson"        ""               ""              
## [69] ""               ""               "innuendo"       ""              
## [73] ""               ""               ""               ""              
## [77] ""               "win"            ""               ""              
## [81] "support"
```

Alternatively, wrap the whitespace-separated character vector by `phrase()` to compound multi-word expressions:


```r
toks_comp <- tokens_compound(toks_news, 
                             pattern =  phrase(tstat_col_cap$collocation[tstat_col_cap$z > 3]))
toks_news[['text7005']][370:450] # before compounding
```

```
##  [1] ""              ""              ""              "need"         
##  [5] ""              ""              "incriminate"   ""             
##  [9] ""              "anyone"        "else"          ""             
## [13] "Scotland"      ""              "investigation" ""             
## [17] ""              ""              "rejected"      "conspiracy"   
## [21] "theories"      ""              ""              "old"          
## [25] "boss"          ""              "Rupert"        "Murdoch"      
## [29] ""              ""              "deals"         ""             
## [33] "people"        "like"          ""              "last"         
## [37] "boss"          ""              "David"         "Cameron"      
## [41] ""              ""              ""              ""             
## [45] "George"        "Osborne"       ""              ""             
## [49] "keen"          ""              "recruit"       ""             
## [53] ""              ""              "Murdoch"       "apparatchik"  
## [57] ""              "despite"       ""              "resignation"  
## [61] ""              ""              "News"          ""             
## [65] ""              "World"         ""              ""             
## [69] "royal"         "hacking"       "case"          ""             
## [73] "Coulson"       ""              ""              ""             
## [77] ""              "innuendo"      ""              ""             
## [81] ""
```

```r
toks_comp[['text7005']][370:450] # after compounding
```

```
##  [1] ""               ""               "incriminate"    ""              
##  [5] ""               "anyone"         "else"           ""              
##  [9] "Scotland"       ""               "investigation"  ""              
## [13] ""               ""               "rejected"       "conspiracy"    
## [17] "theories"       ""               ""               "old"           
## [21] "boss"           ""               "Rupert_Murdoch" ""              
## [25] ""               "deals"          ""               "people"        
## [29] "like"           ""               "last"           "boss"          
## [33] ""               "David_Cameron"  ""               ""              
## [37] ""               ""               "George_Osborne" ""              
## [41] ""               "keen"           ""               "recruit"       
## [45] ""               ""               ""               "Murdoch"       
## [49] "apparatchik"    ""               "despite"        ""              
## [53] "resignation"    ""               ""               "News"          
## [57] ""               ""               "World"          ""              
## [61] ""               "royal"          "hacking"        "case"          
## [65] ""               "Coulson"        ""               ""              
## [69] ""               ""               "innuendo"       ""              
## [73] ""               ""               ""               ""              
## [77] ""               "win"            ""               ""              
## [81] "support"
```
