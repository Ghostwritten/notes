
## 1. 自然语言与查询 Recall

当处理人类自然语言时，有些情况，尽管搜索和原文不完全匹配，但是希望搜到一些内容

 - Quick brown fox 和 fast brown fox / Jumping fox 和 Jumped foxes

一些可采取的优化

 - 归一化词元：清除变音符号，如 role 的的时候 也会匹配 role
 - 抽取词根：清除单复数和时态的差异
 - 包含同义词
 - 拼写错误：拼写错误，或者同音异形词
## 2. 混合多语言的挑战
一些具体的多语言场景
 - 不同的索引使用不同的语言 / 同一索引中，不同的字段使用不同的语言 / 一个文档的一个字段内混合不同的语言

混合语言存在的一些挑战

 - 次干提取：以色列文档，包含了希伯来语，阿拉伯语，俄语和英文
 - 不争取的文档频率 - 英文为主的文章中，德文算分高（稀有）
 - 需要判断用户搜索时使用的语言，语言识别（Compact Language Detecor）
例如，根据语言查询不同的索引

## 3. 分词的挑战
英文分词：You’re 分成一个还是多个？Half -baked
中文分词

 - 分词的标椎：哈工大标椎中，姓和名分开。HanLP 是在一起的。具体情况需制定不同的标椎
 - 歧义（组合型歧义，交际型歧义，真歧义）
中华人民共和国 / 美国会通过对台收武器法案 / 上海仁和服装厂

## 4. 中文分词方法的演变 - 字典法
查字典 - 最容易想到的分词方法（北京航空大学的梁南元教授提出）

 - 一个句子从左到到右扫描一遍。遇到有点词就标识出来。找到复合词，就找最长的
 - 不认识的字符串就分割成单字词

最小词数的分词理论 - 哈工大王晓龙博士吧查字典的方法理论化

 - 一句话应该分词数量最少的词串
 - 遇到二义性的分割，无能为力（例如：“发展中国家”/“上海大学城书店”）
 - 用各种文化规则来解决二义性，都并不成功
## 5. 中文分词方法的演变 - 基于统计法的机器学习算法
统计语言模型 - 1990 年前后 ，清华大学电子工程系郭进博士
 - 解决了二义性问题，将中文分词的错误率降低了一个数据级。概率问题，动态规划 + 利用维特比算法快速找到最佳分词

基于统计的机器学习算法

 - 这类目前常用的算法是 HMM、CRF、SVM、深度学习算法等算法。比如 Hanlp 分词工具是基于 CRF算法为例，基本思路是对汉字进行标注训练，不仅考虑了词语出现的频率，还考虑上下文，具有较好的学习能力，因此其对歧义词和未登录词的识别都具有良好的下效果随着深度学习的兴起，也出现了基于神经网路的分词器，有人尝试使用双向 LSTM + CRF实现分词器，其本质上是序列标注，据报道其分词器字符准确率可高达 97.5%
## 6. 中文分词器现状
中文分词器以统计语言模型为基础，经过几十年的发展，今天基本已经可以看做是一个已经解决的问题
不同分词器的好坏，主要的差别在于数据的使用和工程使用的精度
常见的分词器都是使用机器学期算法和词典相结合，一方面能够提高分词准确率，另一方面能够改善领域适应性

## 7. 一些中文分词器

 - HanLP - 面向生产环境的自然语言处理包
 - IK 分词器
### 7.1 HanLP
./elasticsearch-plugin install [https://github.com/KennFalcon/elasticsearc...](https://github.com/KennFalcon/elasticsearch-analysis-hanlp/releases/download/v7.1.0/elasticsearch-analysis-hanlp-7.1.0.zip)
### 7.2 IK Analysis
/elasticsearch-plugin install [https://github.com/medcl/elasticsearch-ana...](https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.0/elasticsearch-analysis-ik-7.1.0.zip)
### 7.3 拼音
./elasticsearch-plugin install [https://github.com/medcl/elasticsearch-ana...](https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.0/elasticsearch-analysis-ik-7.1.0.zip)

### 7.4 中文分词 DEMO

 - 使用不同分词器测试效果
 - 索引时，尽量切分的短，查询的时候，尽量用长的词
 - 拼音分词器

```bash
#安装插件
./elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.0/elasticsearch-analysis-ik-7.1.0.zip
#安装插件
bin/elasticsearch install https://github.com/KennFalcon/elasticsearch-analysis-hanlp/releases/download/v7.1.0/elasticsearch-analysis-hanlp-7.1.0.zip
```

```bash
docker exec -it es7_01 bash
mkdir /usr/share/elasticsearch/plugins/hanlp
docker cp elasticsearch-analysis-hanlp-7.1.0.zip es7_02:/usr/share/elasticsearch/plugins/hanlp
docker exec -it es7_01 bash
cd /usr/share/elasticsearch/plugins/hanlp
unzip elasticsearch-analysis-hanlp-7.1.0.zip 
rm -rf elasticsearch-analysis-hanlp-7.1.0.zip 
```

```bash
#ik_max_word
#ik_smart
#hanlp: hanlp默认分词
#hanlp_standard: 标准分词
#hanlp_index: 索引分词
#hanlp_nlp: NLP分词
#hanlp_n_short: N-最短路分词
#hanlp_dijkstra: 最短路分词
#hanlp_crf: CRF分词（在hanlp 1.6.6已开始废弃）
#hanlp_speed: 极速词典分词

POST _analyze
{
  "analyzer": "hanlp_standard",
  "text": ["剑桥分析公司多位高管对卧底记者说，他们确保了唐纳德·特朗普在总统大选中获胜"]

}     

#Pinyin
PUT /artists/
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "user_name_analyzer" : {
                    "tokenizer" : "whitespace",
                    "filter" : "pinyin_first_letter_and_full_pinyin_filter"
                }
            },
            "filter" : {
                "pinyin_first_letter_and_full_pinyin_filter" : {
                    "type" : "pinyin",
                    "keep_first_letter" : true,
                    "keep_full_pinyin" : false,
                    "keep_none_chinese" : true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "trim_whitespace" : true,
                    "keep_none_chinese_in_first_letter" : true
                }
            }
        }
    }
}


GET /artists/_analyze
{
  "text": ["刘德华 张学友 郭富城 黎明 四大天王"],
  "analyzer": "user_name_analyzer"
}
```
参考资料：
极客时间：Elasticsearch核心技术与实战
相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
[docker-compose安装elasticsearch集群](https://blog.csdn.net/xixihahalelehehe/article/details/109389780)
[Elasticsearch 7.X之文档、索引、REST API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406518)
[Elasticsearch节点，集群，分片及副本详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406875)
[Elasticsearch倒排索引介绍](https://blog.csdn.net/xixihahalelehehe/article/details/109440345)
[Elasticsearch Analyzer 进行分词详解](https://blog.csdn.net/xixihahalelehehe/article/details/109447777)
[Elasticsearch search API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109449425)
[Elasticsearch URI Search 查询方法详解](https://blog.csdn.net/xixihahalelehehe/article/details/109453253)
[Elasticsearch Request Body 与 Query DSL详解](https://blog.csdn.net/xixihahalelehehe/article/details/109458983)
[Elasticsearch Dynamic Mapping 和常见字段类型详解](https://blog.csdn.net/xixihahalelehehe/article/details/109463294)
[Eelasticsearch 多字段特性及 Mapping 中配置自定义 Analyzer详解](https://blog.csdn.net/xixihahalelehehe/article/details/109476672)
[Elasticsearch index template与dynamic template详解](https://blog.csdn.net/xixihahalelehehe/article/details/109595303)
[Elasticsearch 聚合分析简介](https://blog.csdn.net/xixihahalelehehe/article/details/109625376)
[Elasticsearch 第一阶段总结与测试](https://blog.csdn.net/xixihahalelehehe/article/details/109626903)
[Elasticsearch 基于词项和基于全文的搜索详解](https://blog.csdn.net/xixihahalelehehe/article/details/109669976)
[Elasticsearch 结构化搜索详解](https://blog.csdn.net/xixihahalelehehe/article/details/109674952)
[Elasticsearch 搜索的相关性算分详解](https://blog.csdn.net/xixihahalelehehe/article/details/109720759)
[Elasticsearch Query & Filtering 与 多字符串多字段查询详解](https://blog.csdn.net/xixihahalelehehe/article/details/109773574)
[Elasticsearch 单字符串多字段查询：Dis Max Query详解](https://blog.csdn.net/xixihahalelehehe/article/details/109774891)
[Elasticsearch 单字符串多字段查询: Multi Match详解](https://blog.csdn.net/xixihahalelehehe/article/details/109776199)
[Elasticsearch 多语言及中文分词与检索详解](https://blog.csdn.net/xixihahalelehehe/article/details/109778074)
