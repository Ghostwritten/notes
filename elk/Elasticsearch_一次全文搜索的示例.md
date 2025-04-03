

----
## 1. 目的
目标：用一个具体案例，巩固知识点

 - 写入数据、设置mapping、设置analysis
 - 查询并高亮显示结果
 - 分析查询结果，通过修改配置和查询，优化搜索的相关性
 - 分析问题，结合原理，分析思考并加以实践

## 2. TMDB数据库
创建2008年，电影的meta Data库
46万本电影，12万本电视剧，230万张图片，每周20万次编辑

提供API

## 3. 数据导入
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6ab361f72c3b13fbc8549d307debf664.png)


 - 数据特征----标题信息短、概述相对较长
 - 通过TMDB Search API
 - 将查询数据保存在本地CSV文件中
 - 使用python导入并查询数据
 - 索引的主分片数设置为1，使用默认的Dynamic Mapping


## 4. use case --查找Space Jam

空中大灌篮（Space JAM）
华纳公司动画明星、篮球巨星乔丹、外星小怪物

案例：用户不记得电影名，而希望通过关键词，搜索到电影的详细信息

搜索关键词： 
Basketball with Cartoon Aliens


## 5. 操作
介质下载：
```bash
[root@master elk]# tree test31/
test31/
├── ingest_tmdb_from_file.py
├── mapping
│   ├── english_analyzer.json
│   └── english_english_3_shards.json
├── query
│   ├── query_space_jam_improved.json
│   ├── query_space_jam.json
│   └── query_space_jam_most_fields.json
├── query_tmdb.py
└── tmdb.json
```

安装python 模块

```bash
yum -y install python-pip 
pip install requests
```
脚本
 cat ingest_tmdb_from_file.py 

```bash
import requests
import json
import os

indexName ="tmdb"   #索引名
mappingFolder ="./mapping"  #映射目录
headers={"Content-Type":"application/json","Accept":"application/json"}

def extract():
    f = open('./tmdb.json') #打开tmdb.json数据源
    if f:
         return json.loads(f.read());读取数据源
    return {}


def reindex(settings, movieDict={}):#获取

    resp = requests.delete("http://192.168.211.60:9200/"+indexName) #D删除已存在的该索引名
    data = json.dumps(settings,indent=4, sort_keys=True)对数据源信息进行json处理
    print "settings:\n%s" %data  #打印
    resp = requests.put("http://192.168.211.60:9200/"+indexName,
        headers=headers, data=data)  #将数据源导入tmdb索引名内

    print "Response for createing the index with the settings and mappings. %s" %resp.text   #打印返回信息

    bulkMovies = ""
    for id, movie in movieDict.iteritems():#循环电影列表，进行简化初始化电影排序
        addCmd = {"index": {"_index": indexName,
                            "_type": "_doc",
                            "_id": movie["id"]}}
        bulkMovies += json.dumps(addCmd) + "\n" + json.dumps(movie) + "\n"

    print "Start ingesting data......"
    resp = requests.post("http://192.168.211.60:9200/_bulk", headers={"content-type":"application/json"}, data=bulkMovies)
    #print resp.content

def select_mapping():
    print "\r\n>> Please select the mapping file. Choose 0 for empty mapping\r\n"
    mappingList=os.listdir(mappingFolder) #列出mapping分词器
    print "[0] empty mapping. It will use dynamic mapping with default settings"
    for idx, mappingItem in enumerate(mappingList):
        print "[%d] %s" %(idx+1,mappingItem)  #显示个数---mapping分词器文件
    userInput = raw_input() #或控制台输出
    try:
        selectIndex = int(userInput) #整数初始化输入内容
    except ValueError:
        selectIndex =-1

    if(selectIndex==-1 or selectIndex > len(mappingList)+1): #假如输入不符合数字或者数字大于分词器文件数量报错
        print '\033[31mPlease provide a valid integer \033[0m'
        msg = "from 0 to %d." %(len(mappingList))
        print msg
        exit()
    if selectIndex == 0:  #假如等于-，不使用分词器
        print "return empty"
        return {}
    mappingName = mappingList[selectIndex-1]  #安装索引取值分词器文件
    fileName="%s/%s" %(mappingFolder,mappingName) #打印确认分词器文件是否正确
    f = open(fileName) #打开文件
    mapping = {}
    if f:
        mapping = json.loads(f.read()); #读取分词器方法
    return mapping


def main():
    movieDict = extract()
    mapping = select_mapping()
    reindex(settings=mapping, movieDict=movieDict)  #获取分词器方法与初始化的数据内容
    print "Done for ingesting TMDB data into Elasticsearch"

if __name__== "__main__":
  main()
```
cat mapping/english_analyzer.json 

```bash
{
  "settings": {   #设置
    "number_of_shards": 1  #主分片为1
  },
  "mappings": {  #映射
    "properties": { #属性
      "overview": {  #概述
        "type": "text", #格式为text文本
        "analyzer": "english", #英文语言分词器
        "fields": {  #字段
          "std": {
            "type": "text",  # 文本
            "analyzer": "standard" #默认标准分词器
          }
        }
      },
      "popularity": { #流行度，人气
        "type": "float" #浮点数
      },
      "title": {  #标题
        "type": "text",  #类型为text
        "analyzer": "english",分词器为英文
        "fields": {  #字段
          "keyword": {  #关键词
            "type": "keyword",  #类型关键词
            "ignore_above": 256  #设置ignore_above属性(默认是10) ，表示最大的字段值长度，超出这个长度的字段将不会被索引，但是会存储。
          }
        }
      }
    }
  }
}
```

 cat mapping/english_english_3_shards.json 

```bash
{
  "settings": {
    "number_of_shards": 3,  #主分片为3
    "number_of_replicas": 1  #副本数量为1
  },
  "mappings": {
    "properties": {
      "overview": {
        "type": "text",
        "analyzer": "english",
        "fields": {
          "std": {
            "type": "text",
            "analyzer": "standard"
          }
        }
      },
      "popularity": {
        "type": "float"
      },
      "title": {
        "type": "text",
        "analyzer": "english",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```

cat query_tmdb.py 

```bash
import requests
import json
import os
import sys

indexName ="tmdb"
queryFolder ="./query"
headers={"Content-Type":"application/json","Accept":"application/json"}

def search(query, printHighlight):
    print query  #打印请求方法
    url = "http://localhost:9200/%s/_search" %indexName  #设置带有索引名的url
    resp = requests.get(url,headers=headers,data=json.dumps(query)) #根据query方法请求url数据
    hits = json.loads(resp.text)["hits"] #对获取的数据进行json处理
    print "\r\n######################## Results ################################\r\n"
    print "No\tScore\t\t\tTitle"
    for idx, hit in enumerate(hits["hits"]):  #循环处理返回数据
        print "%s\t%s\t\t\t%s" %(idx+1, hit["_score"], hit["_source"]["title"])#从1打印序号、人气、电影名 
        print "---------------------------------------------------------------"
        if printHighlight:
            if (("highlight" in hit.keys()) and ("title" in hit["highlight"].keys())):
                hl = ";".join(hit["highlight"]["title"]).replace("<em>","\033[1;31;40m").replace("</em>","\033[0m")
                print "title: \033[0;32;40m%d hit(s)\033[0m \r\n%s\r\n--" %(len(hit["highlight"]["title"]) ,hl)
            if (("highlight" in hit) and ("overview" in hit["highlight"])):
                hl = ";".join(hit["highlight"]["overview"]).replace("<em>","\033[1;31;40m").replace("</em>","\033[0m")
                print "overview: \033[0;32;40m%d hit(s)\033[0m \r\n%s\r\n--" %( len(hit["highlight"]["overview"]), hl)


def select_query():
    print "\r\n>> Please select the query file.\r\n"  #请选择请求方法
    queryList=os.listdir(queryFolder)  #列出请求方法
    for idx, queryItem in enumerate(queryList): #安装序号控制台列出请求方法json文本
        print "[%d] %s" %(idx,queryItem)
    userInput = raw_input()  #控制台输入选择的序号
    try:
        selectIndex = int(userInput)
    except ValueError:
        selectIndex =-1
    if(selectIndex==-1 or selectIndex > len(queryList)): #判断序号是否符合规范
        print '\033[31mPlease provide a valid integer \033[0m'
        msg = "from 0 to %d." %(len(queryList))
        print msg
        exit()
    queryName = queryList[selectIndex]
    fileName="%s/%s" %(queryFolder,queryName)
    f = open(fileName)  #打开文本
    query = {}
    if f:
        query = json.loads(f.read());  #读取文本
    return query

def main():
    highlight = False  #高亮默认关闭
    for arg in sys.argv: #参数输入如果出现“h”、“hl”、“highlight”则打开高亮模式
        if arg == "h" or arg == "hl" or arg == "highlight":
            highlight = True
    query = select_query()  #查询函数
    search(query, highlight)

if __name__== "__main__":
  main()
```

cat query/query_space_jam_improved.json 

```bash
{
      "_source": ["title","overview"], #原始文档包含标题与概述
      "size":20,  #大小不超20个
      "query": {
          "multi_match": { #混合字段匹配，默认"type": "best_fields",当字段之间相互竞争，又相互关联。例如 title 和 body 这样的字段，评分来自最匹配字段
              "query": "basketball with cartoon aliens",
              "fields": ["title","overview"]
          }
      },
      "highlight" : {
            "fields" : {
              "overview" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] },
              "title" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] }
            }
        }
  }
```
 cat query/query_space_jam.json 

```bash
{
      "_source": ["title","overview"],
      "size":20,
      "query": {
          "multi_match": {
              "query": "basketball with cartoon aliens",
              "fields": ["title^10","overview"]  #正则匹配
          }
      },
      "highlight" : {
            "fields" : {
                "overview" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] },
                "title" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] }
            }
        }

  }
```

cat query/query_space_jam_most_fields.json 

```bash
{
      "_source": ["title","overview"],
      "size":20,
      "query": {
          "multi_match": {
              "type": "most_fields",  #处理英文内容时：一种常见的手段是，在主字段（English Analyzer），抽取词干，加入同义词，以匹配更多的文档。相同的文本，加入子字段（Standard Analyzer），以提供更加精确的匹配。其他字段作为匹配文档提高性相关度的信号。匹配字段越多越好
              "query": "basketball with cartoon aliens",
              "fields": ["title","overview"]
          }
      },
      "highlight" : {
            "fields" : {
              "overview" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] },
              "title" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] }
            }
        }
  }
```


### 5.1 不使用分词器三种请求的结果类比

```bash
[root@master test31]# python ingest_tmdb_from_file.py 

>> Please select the mapping file. Choose 0 for empty mapping

[0] empty mapping. It will use dynamic mapping with default settings
[1] english_analyzer.json
[2] english_english_3_shards.json
0  <控制台输入>
return empty
settings:
{}
Response for createing the index with the settings and mappings. {"acknowledged":true,"shards_acknowledged":true,"index":"tmdb"}
Start ingesting data......
Done for ingesting TMDB data into Elasticsearch
[root@master test31]# python query_tmdb.py 

>> Please select the query file.

[0] query_space_jam_improved.json
[1] query_space_jam.json
[2] query_space_jam_most_fields.json
0  <控制台输入>
{u'highlight': {u'fields': {u'overview': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}, u'title': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}}}, u'query': {u'multi_match': {u'query': u'basketball with cartoon aliens', u'fields': [u'title', u'overview']}}, u'size': 20, u'_source': [u'title', u'overview']}

######################## Results ################################

No	Score			Title
1	9.305035			Meet Dave
---------------------------------------------------------------
2	8.556931			Aliens
---------------------------------------------------------------
3	8.257416			Speed Racer
---------------------------------------------------------------
4	8.007193			Aliens in the Attic
---------------------------------------------------------------
5	7.9753795			Space Jam   #想要的结果
---------------------------------------------------------------
6	7.7324815			Grown Ups
---------------------------------------------------------------
7	7.505871			Semi-Pro
---------------------------------------------------------------
8	7.390269			The Flintstones
---------------------------------------------------------------
9	7.3710775			The Basketball Diaries
---------------------------------------------------------------
10	7.2194405			Coach Carter
---------------------------------------------------------------
11	7.1320205			Cowboys & Aliens
---------------------------------------------------------------
12	6.7105947			White Men Can't Jump
---------------------------------------------------------------
13	6.6692567			Alien: Resurrection
---------------------------------------------------------------
14	6.5318284			Teen Wolf
---------------------------------------------------------------
15	6.5275307			District 9
---------------------------------------------------------------
16	6.491797			Bedazzled
---------------------------------------------------------------
17	6.458376			The Watch
---------------------------------------------------------------
18	6.155629			Galaxy Quest
---------------------------------------------------------------
19	6.1139226			Monsters vs Aliens
---------------------------------------------------------------
20	5.746967			Batteries Not Included
---------------------------------------------------------------
[root@master test31]# python query_tmdb.py 

>> Please select the query file.

[0] query_space_jam_improved.json
[1] query_space_jam.json
[2] query_space_jam_most_fields.json
1
{u'highlight': {u'fields': {u'overview': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}, u'title': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}}}, u'query': {u'multi_match': {u'query': u'basketball with cartoon aliens', u'fields': [u'title^10', u'overview']}}, u'size': 20, u'_source': [u'title', u'overview']}

######################## Results ################################

No	Score			Title
1	85.56929			Aliens
---------------------------------------------------------------
2	73.71077			The Basketball Diaries
---------------------------------------------------------------
3	71.3202			Cowboys & Aliens
---------------------------------------------------------------
4	61.13922			Monsters vs Aliens
---------------------------------------------------------------
5	53.501827			Aliens in the Attic
---------------------------------------------------------------
6	53.501827			Aliens vs Predator: Requiem
---------------------------------------------------------------
7	45.221096			Dances with Wolves
---------------------------------------------------------------
8	45.221096			Friends with Kids
---------------------------------------------------------------
9	45.221096			Friends with Benefits
---------------------------------------------------------------
10	45.221096			Fire with Fire
---------------------------------------------------------------
11	39.572163			From Paris with Love
---------------------------------------------------------------
12	39.572163			Sleeping with the Enemy
---------------------------------------------------------------
13	39.572163			Interview with the Vampire
---------------------------------------------------------------
14	39.572163			Just Go With It
---------------------------------------------------------------
15	39.572163			To Rome with Love
---------------------------------------------------------------
16	39.572163			Gone with the Wind
---------------------------------------------------------------
17	39.572163			My Week with Marilyn
---------------------------------------------------------------
18	39.572163			Hobo with a Shotgun
---------------------------------------------------------------
19	39.572163			From Russia With Love
---------------------------------------------------------------
20	39.572163			Trouble with the Curve
---------------------------------------------------------------
[root@master test31]# python query_tmdb.py 

>> Please select the query file.

[0] query_space_jam_improved.json
[1] query_space_jam.json
[2] query_space_jam_most_fields.json
2  <控制台输入>
{u'highlight': {u'fields': {u'overview': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}, u'title': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}}}, u'query': {u'multi_match': {u'query': u'basketball with cartoon aliens', u'type': u'most_fields', u'fields': [u'title', u'overview']}}, u'size': 20, u'_source': [u'title', u'overview']}

######################## Results ################################

No	Score			Title
1	13.357376			Aliens in the Attic
---------------------------------------------------------------
2	9.468831			Aliens
---------------------------------------------------------------
3	9.305035			Meet Dave
---------------------------------------------------------------
4	8.406412			Cowboys & Aliens
---------------------------------------------------------------
5	8.257416			Speed Racer
---------------------------------------------------------------
6	7.9753795			Space Jam  #想要的结果
---------------------------------------------------------------
7	7.7324815			Grown Ups
---------------------------------------------------------------
8	7.505871			Semi-Pro
---------------------------------------------------------------
9	7.390269			The Flintstones
---------------------------------------------------------------
10	7.3710775			The Basketball Diaries
---------------------------------------------------------------
11	7.2194405			Coach Carter
---------------------------------------------------------------
12	7.0533514			Monsters vs Aliens
---------------------------------------------------------------
13	6.7105947			White Men Can't Jump
---------------------------------------------------------------
14	6.6692567			Alien: Resurrection
---------------------------------------------------------------
15	6.5318284			Teen Wolf
---------------------------------------------------------------
16	6.5275307			District 9
---------------------------------------------------------------
17	6.491797			Bedazzled
---------------------------------------------------------------
18	6.458376			The Watch
---------------------------------------------------------------
19	6.155629			Galaxy Quest
---------------------------------------------------------------
20	5.746967			Batteries Not Included
---------------------------------------------------------------
```

### 5.2 带有英文分词器的三种请求结果

```bash
[root@master test31]# python ingest_tmdb_from_file.py 

>> Please select the mapping file. Choose 0 for empty mapping

[0] empty mapping. It will use dynamic mapping with default settings
[1] english_analyzer.json
[2] english_english_3_shards.json
1
settings:
{
    "mappings": {
        "properties": {
            "overview": {
                "analyzer": "english", 
                "fields": {
                    "std": {
                        "analyzer": "standard", 
                        "type": "text"
                    }
                }, 
                "type": "text"
            }, 
            "popularity": {
                "type": "float"
            }, 
            "title": {
                "analyzer": "english", 
                "fields": {
                    "keyword": {
                        "ignore_above": 256, 
                        "type": "keyword"
                    }
                }, 
                "type": "text"
            }
        }
    }, 
    "settings": {
        "number_of_shards": 1
    }
}
Response for createing the index with the settings and mappings. {"acknowledged":true,"shards_acknowledged":true,"index":"tmdb"}
Start ingesting data......
Done for ingesting TMDB data into Elasticsearch
[root@master test31]# python query_tmdb.py 

>> Please select the query file.

[0] query_space_jam_improved.json
[1] query_space_jam.json
[2] query_space_jam_most_fields.json
0
{u'highlight': {u'fields': {u'overview': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}, u'title': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}}}, u'query': {u'multi_match': {u'query': u'basketball with cartoon aliens', u'fields': [u'title', u'overview']}}, u'size': 20, u'_source': [u'title', u'overview']}

######################## Results ################################

No	Score			Title
1	12.882349			Space Jam  #想要的结果
---------------------------------------------------------------
2	7.876023			The Basketball Diaries
---------------------------------------------------------------
3	7.53847			Grown Ups
---------------------------------------------------------------
4	7.499677			Speed Racer
---------------------------------------------------------------
5	7.409075			Alien
---------------------------------------------------------------
6	7.409075			Aliens
---------------------------------------------------------------
7	7.409075			Alien³
---------------------------------------------------------------
8	7.244087			Semi-Pro
---------------------------------------------------------------
9	7.162643			The Flintstones
---------------------------------------------------------------
10	6.943389			Coach Carter
---------------------------------------------------------------
11	6.765371			White Men Can't Jump
---------------------------------------------------------------
12	5.9676995			Cowboys & Aliens
---------------------------------------------------------------
13	5.9676995			Aliens in the Attic
---------------------------------------------------------------
14	5.9676995			Alien: Resurrection
---------------------------------------------------------------
15	5.8452215			Meet Dave
---------------------------------------------------------------
16	5.8005633			Aliens vs Predator: Requiem
---------------------------------------------------------------
17	5.440302			Bedazzled
---------------------------------------------------------------
18	5.3304057			High School Musical
---------------------------------------------------------------
19	5.3242			The Thing
---------------------------------------------------------------
20	5.1603985			Invasion of the Body Snatchers
---------------------------------------------------------------
[root@master test31]# python query_tmdb.py 

>> Please select the query file.

[0] query_space_jam_improved.json
[1] query_space_jam.json
[2] query_space_jam_most_fields.json
1
{u'highlight': {u'fields': {u'overview': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}, u'title': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}}}, u'query': {u'multi_match': {u'query': u'basketball with cartoon aliens', u'fields': [u'title^10', u'overview']}}, u'size': 20, u'_source': [u'title', u'overview']}

######################## Results ################################

No	Score			Title
1	78.76022			The Basketball Diaries
---------------------------------------------------------------
2	74.090744			Alien
---------------------------------------------------------------
3	74.090744			Aliens
---------------------------------------------------------------
4	74.090744			Alien³
---------------------------------------------------------------
5	59.676994			Cowboys & Aliens
---------------------------------------------------------------
6	59.676994			Aliens in the Attic
---------------------------------------------------------------
7	59.676994			Alien: Resurrection
---------------------------------------------------------------
8	49.95806			Monsters vs Aliens
---------------------------------------------------------------
9	42.961407			AVP: Alien vs. Predator
---------------------------------------------------------------
10	42.961407			Aliens vs Predator: Requiem
---------------------------------------------------------------
11	12.882349			Space Jam  #想要的结果
---------------------------------------------------------------
12	7.53847			Grown Ups
---------------------------------------------------------------
13	7.499677			Speed Racer
---------------------------------------------------------------
14	7.244087			Semi-Pro
---------------------------------------------------------------
15	7.162643			The Flintstones
---------------------------------------------------------------
16	6.943389			Coach Carter
---------------------------------------------------------------
17	6.765371			White Men Can't Jump
---------------------------------------------------------------
18	5.8452215			Meet Dave
---------------------------------------------------------------
19	5.440302			Bedazzled
---------------------------------------------------------------
20	5.3304057			High School Musical
---------------------------------------------------------------
[root@master test31]# python query_tmdb.py 

>> Please select the query file.

[0] query_space_jam_improved.json
[1] query_space_jam.json
[2] query_space_jam_most_fields.json
2
{u'highlight': {u'fields': {u'overview': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}, u'title': {u'pre_tags': [u'<em>'], u'post_tags': [u'</em>']}}}, u'query': {u'multi_match': {u'query': u'basketball with cartoon aliens', u'type': u'most_fields', u'fields': [u'title', u'overview']}}, u'size': 20, u'_source': [u'title', u'overview']}

######################## Results ################################

No	Score			Title
1	12.882349			Space Jam  #想要的结果
---------------------------------------------------------------
2	11.015935			Aliens
---------------------------------------------------------------
3	10.936138			Aliens in the Attic
---------------------------------------------------------------
4	10.797355			Alien³
---------------------------------------------------------------
5	10.730265			Alien
---------------------------------------------------------------
6	10.0967045			Aliens vs Predator: Requiem
---------------------------------------------------------------
7	9.960973			Alien: Resurrection
---------------------------------------------------------------
8	8.02322			AVP: Alien vs. Predator
---------------------------------------------------------------
9	7.876023			The Basketball Diaries
---------------------------------------------------------------
10	7.53847			Grown Ups
---------------------------------------------------------------
11	7.499677			Speed Racer
---------------------------------------------------------------
12	7.244087			Semi-Pro
---------------------------------------------------------------
13	7.162643			The Flintstones
---------------------------------------------------------------
14	6.943389			Coach Carter
---------------------------------------------------------------
15	6.765371			White Men Can't Jump
---------------------------------------------------------------
16	5.9676995			Cowboys & Aliens
---------------------------------------------------------------
17	5.8452215			Meet Dave
---------------------------------------------------------------
18	5.440302			Bedazzled
---------------------------------------------------------------
19	5.3304057			High School Musical
---------------------------------------------------------------
20	5.3242			The Thing
---------------------------------------------------------------
```
第三个分词器方法只是主分片与副本的结果，查询结果与第二个分词器方法一样。

## 6. 思考与分析
精确值还是全文
搜索是怎么样的？ 不同的字段需要配置怎么样的分词器

测试不同的选项：
 分词期、多字段属性、是否需要g-grams、what are some critical synnonyms 、为字段设置不同权重
 测试不同的选项，测试不同的搜索条件 


## 7. 测试相关性
理解原理 + 多分析 + 多调整测试

技术分为道和术两种

 - 道----原理和原则
 - 术----具体的方法，具体的解法

关于搜索，为了一个好的搜索结果，除了真正理解背后的原理，更需要实践分析

 - 单纯追求“术”，会很辛苦，只有掌握了本质和精髓之“道”，做事才能游刃有余。
 - 要做好搜索，除了理解原理，也需要坚持分析一些不好的搜索结果，只有通过一定的时间的积累，才能真正有所感觉。
 - 总希望一个模型，一个算法，就能毕其功于一役，是不现实的。


## 8. 监控并理解用户行为
不要过度调试相关度
而要监控搜索结果，监控用户点击最顶端结果的频次
将搜索结果提高到极高水平，唯一途径是：

 - 需要具有度量用户行为的强大能力
 - 可以后台实现统计数据，比如，用户的查询和结果，有多少被点击了。
 - 哪些搜索，没有返回结果

相关链接；
[Elasticsearch 单字符串多字段查询: multi_match详解](https://ghostwritten.blog.csdn.net/article/details/109776199)
[elasticsearch的_source、_all、store和index](https://ghostwritten.blog.csdn.net/article/details/113943171)




