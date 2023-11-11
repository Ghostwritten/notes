

----
## 1. Analysis 简介
理解elasticsearch的ngram首先需要了解elasticsearch中的analysis。在此我们快速回顾一下基本原理：

当一个文档被索引时，每个field都可能会创建一个倒排索引（如果mapping的时候没有设置不索引该field）。倒排索引的过程就是将文档通过analyzer分成一个一个的term,每一个term都指向包含这个term的文档集合。当查询query时，elasticsearch会根据搜索类型决定是否对query进行analyze，然后和倒排索引中的term进行相关性查询，匹配相应的文档

```bash
analyzer = CharFilters（0个或多个） + Tokenizer(恰好一个) + TokenFilters(0个或多个)
```
## 2. index analyzer VS search analyzer
如果`mapping`中只设置了一个analyzer，那么这个`analyzer`会同时用于索引文档和搜索`query`。当然索引文档和对query进行`analysis`也可以使用不同的analyzer

一个特殊的情况是有的query是需要被analyzed，有的并不需要。例如`match query`会先用`search analyzer`进行分析，然后去相应`field`的倒排索引进行匹配。而`term query`并不会对query内容进行分析，而是直接和相应field的倒排索引去匹配

## 3. Analyze API
Analyze API是一个有效的方式查看分析后的结果：

```bash
POST _analyze
{
    "analyzer" : "standard",
    "text" : "hello, world"
}
```

输出的结果如下所示，即`[hello, world]`

## 4. Ngram
在机器学习和数据挖掘领域，ngram通常指的是n个词的序列。不过在elasticsearch中，ngram代表的是n个字符的序列。可以把ngram理解成长度为n的滑动窗口，在文档中滑动产生的序列
## 5. Ngram Tokenizer

```bash
POST _analyze
{
  "tokenizer": "ngram",
  "text": "Quick Fox"
}
```

ngram分词器默认会产生最小长度为1，最大长度为2的N-grams序列。上述查询语句的输出是
[ Q, Qu, u, ui, i, ic, c, ck, k, "k ", " ", " F", F, Fo, o, ox, x ]

也可以自定义ngram tokenizer的一些配置：

 - min_gram： 指定产生的最小长度的字符序列，默认为1
 - max_gram： 指定产生的最大长度的字符序列，默认为2
 - token_chars: 指定生成的token应该包含哪些字符.对没有包含进的字符进行分割，默认为[],即保留所有字符

 - - letter - eg: a,b,字
- - digit - eg: 3,7
- - whitespace - eg: " ", "\n"
- - punctuation - eg: !, "
- - symbol - eg: $,√

定义`min_gram`和`max_gram`应该按照使用场景来定。使用ngram的一个常见场景就是自动补全。如果单个字符也进行自动补全，那么可能匹配的suggestion太多，导致没有太大意义。
另一个需要考虑的便是性能，产生大量的ngram占用空间更大，搜索时花费的事件也更多。

假如现在需要将ngrams生成的token都转换成小写形式，可以使用如下形式


```bash
PUT my_index
{
   "settings": {
      "analysis": {
         "tokenizer": {
            "ngram_tokenizer": {
               "type": "nGram",
               "min_gram": 4,
               "max_gram": 4,
               "token_chars": [ "letter", "digit" ]
            }
         },
         "analyzer": {
            "ngram_tokenizer_analyzer": {
               "type": "custom",
               "tokenizer": "ngram_tokenizer",
               "filter": [
                  "lowercase"
               ]
            }
         }
      }
   }
}
POST my_index/_analyze
{
  "analyzer": "ngram_tokenizer_analyzer",
  "text": "Hello, World!"
}
生成的结果序列是[ello, hell, orld, worl]
```
## 6. Ngram Token Filter
上述的例子也可以使用`Ngram Token Filter`，配上standard的分词器和`lower-case`的过滤器。
原文本被standard分词器以whitespace和punctuation分割成token，然后通过lowercase过滤器转换成小写形式，最后通过ngram filter生成长度为4的字符序列

```bash
PUT /my_index
{
   "settings": {
      "analysis": {
         "filter": {
            "ngram_filter": {
               "type": "nGram",
               "min_gram": 4,
               "max_gram": 4
            }
         },
         "analyzer": {
            "ngram_filter_analyzer": {
               "type": "custom",
               "tokenizer": "standard",
               "filter": [
                  "lowercase",
                  "ngram_filter"
               ]
            }
         }
      }
   }
}

POST my_index/_analyze
{
  "analyzer": "ngram_filter_analyzer",
  "text": "Hello, World!"
}
```
上述mapping也会和ngram tokenizer产生同样的效果，具体实际应用如何选择应该视场景而定。假如想匹配的term中可以包含特殊字符，那么你应该使用ngram tokenizer。因为standard的tokenizer会过滤掉这些特殊字符。


## 7. Edge Ngram
Edge Ngram和ngram类似，只不过Edge Ngram产生的序列都是从代索引文档的开头延伸出去的。

```bash
POST _analyze
{
  "tokenizer": "edge_ngram",
  "text": "Quick Fox"
}
```

会产生`[ Q, Qu ]`

Edge Ngram也有着和ngram相同的配置

 - min_gram： 指定产生的最小长度的字符序列，默认为1
 - max_gram： 指定产生的最大长度的字符序列，默认为2
 - token_chars: 指定生成的token应该包含哪些字符.对没有包含进的字符进行分割，默认为[],即保留所有字符

- - letter - eg: a,b,字
- - digit - eg: 3,7
- - whitespace - eg: " ", "\n"
- - punctuation - eg: !, "
- - symbol - eg: $,√
对于如下的mapping

```bash
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
  }
}
POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "2 Quick Foxes."
}
```

产生的token序列为`[ Qu, Qui, Quic, Quick, Fo, Fox, Foxe, Foxes ]`


参考连接：
[https://njucz.github.io/2017/12/20/elasticsearch%20ngram,edgengram%E7%AC%94%E8%AE%B0/](https://njucz.github.io/2017/12/20/elasticsearch%20ngram,edgengram%E7%AC%94%E8%AE%B0/)
