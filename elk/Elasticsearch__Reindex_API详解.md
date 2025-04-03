


---
## 1. 应用场景：
1. 当你的数据量过大，而你的索引最初创建的分片数量不足，导致数据入库较慢的情况，此时需要扩大分片的数量，此时可以尝试使用Reindex。

2. 当数据的mapping需要修改，但是大量的数据已经导入到索引中了，重新导入数据到新的索引太耗时；但是在ES中，一个字段的mapping在定义并且导入数据之后是不能再修改的，

3.  集群中数据迁移、跨集群的数据迁移

所以这种情况下也可以考虑尝试使用Reindex。

Reindex：
ES提供了_reindex这个API。相对于我们重新导入数据肯定会快不少，实测速度大概是bulk导入数据的5-10倍。

## 1. Reindex API
Reindex API 支持把文档从一个索引拷贝到另外一个索引

```bash
# Reindx API，version Type Internal
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix",
    "version_type": "internal"
  }
}

# 文档版本号增加
GET  blogs_fix/_doc/1

# Reindx API，version Type Internal
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix",
    "version_type": "external"
  }
}

# Reindx API，version Type Internal
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix",
    "version_type": "external"
  },
  "conflicts": "proceed"
}
```
### 3.1 两个注意点

 - 索引的 `mapping _source` 要开启
 - 先创建一个新索引，然后在执行 reindex
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/43ad049b6b9acae7ab00e4cdc6e471c9.png)
### 3.2. OP Type
 - `_reindex` 指挥创建不存在的文档
 - 文档如果存在，会导致版本冲突
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc39dd37dc8be5039d3672a2da13b394.png)
### 3.3. 跨集群 ReIndex
 - 需要修改 `elasticsearch.yml` , 并且重启节点
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/03f4381a5b07a09a65b1162a9c74293d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/13fe8ca5d0740b71216e3193683b0650.png)
### 3.4 查看 Task API
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8378135df62c410d0dfd4227d689a4cc.png)

 - Reindex API 支持一步操作，执行只返回 Task Id
 - POST _reindex?wait_for_completion=false
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe8ab30bdec80f2f58342d8f4193a892.png)
## 4. 数据迁移
1、创建新的索引(可以通过java程序也可以直接在head插件上创建)

注意：在创建索引的时候要把表结构也要创建好(也就是mapping)
2、复制数据

最简单、基本的方式：

```bash
POST _reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  }
}
```
利用命令：

```bash
curl _XPOST 'ES数据库请求地址:9200/_reindex' -d{"source":{"index":"old_index"},"dest":{"index":"new_index"}}
```
但如果新的index中有数据，并且可能发生冲突，那么可以设置`version_type"version_type": "internal"`或者不设置，则Elasticsearch强制性的将文档转储到目标中，覆盖具有相同类型和ID的任何内容：

```bash
POST _reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index",
    "version_type": "internal"
  }
}
```

## 5. 数据迁移效率
问题发现：

常规的如果我们只是进行少量的数据迁移利用普通的reindex就可以很好的达到要求，但是当我们发现我们需要迁移的数据量过大时，我们会发现reindex的速度会变得很慢

数据量几十个G的场景下，elasticsearch reindex速度太慢，从旧索引导数据到新索引，当前最佳方案是什么？
原因分析：

reindex的核心做跨索引、跨集群的数据迁移。
慢的原因及优化思路无非包括：
    1）批量大小值可能太小。需要结合堆内存、线程池调整大小；
    2）reindex的底层是scroll实现，借助scroll并行优化方式，提升效率；
    3）跨索引、跨集群的核心是写入数据，考虑写入优化角度提升效率。


可行方案：

1）提升批量写入大小值

默认情况下，`_reindex`使用1000进行批量操作，您可以在source中调整`batch_size`。

```bash
POST _reindex
{
  "source": {
    "index": "source",
    "size": 5000
  },
  "dest": {
    "index": "dest",
    "routing": "=cat"
  }
}
```
批量大小设置的依据：

1、使用批量索引请求以获得最佳性能。

批量大小取决于数据、分析和集群配置，但一个好的起点是每批处理5-15 MB。

注意，这是物理大小。文档数量不是度量批量大小的好指标。例如，如果每批索引1000个文档:

1）每个1kb的1000个文档是1mb。

2）每个100kb的1000个文档是100 MB。

这些是完全不同的体积大小。

2、逐步递增文档容量大小的方式调优。

 - 1）从大约5-15 MB的大容量开始，慢慢增加，直到你看不到性能的提升。然后开始增加批量写入的并发性(多线程等等)。
 - 2）使用kibana、cerebro或iostat、top和ps等工具监视节点，以查看资源何时开始出现瓶颈。如果您开始接收EsRejectedExecutionException，您的集群就不能再跟上了:至少有一个资源达到了容量。

要么减少并发性，或者提供更多有限的资源(例如从机械硬盘切换到ssd固态硬盘)，要么添加更多节点。

2）借助scroll的sliced提升写入效率

Reindex支持`Sliced Scroll`以并行化重建索引过程。 这种并行化可以提高效率，并提供一种方便的方法将请求分解为更小的部分。

sliced原理（from medcl）

1）用过Scroll接口吧，很慢？如果你数据量很大，用Scroll遍历数据那确实是接受不了，现在Scroll接口可以并发来进行数据遍历了。
2）每个Scroll请求，可以分成多个`Slice`请求，可以理解为切片，各Slice独立并行，利用Scroll重建或者遍历要快很多倍。

slicing使用举例

slicing的设定分为两种方式：手动设置分片、自动设置分片。
手动设置分片参见官网。
自动设置分片如下：


```bash
POST _reindex?slices=5&refresh
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

slices大小设置注意事项：

 - 1）slices大小的设置可以手动指定，或者设置slices设置为auto，auto的含义是：针对单索引，slices大小=分片数；针对多索引，slices=分片的最小值。
 - 2）当slices的数量等于索引中的分片数量时，查询性能最高效。slices大小大于分片数，非但不会提升效率，反而会增加开销。
 - 3）如果这个slices数字很大(例如500)，建议选择一个较低的数字，因为过大的slices 会影响性能。

效果

实践证明，比默认设置reindex速度能提升10倍+。


参考连接：
[https://www.cnblogs.com/Ace-suiyuan008/p/9985249.html](https://www.cnblogs.com/Ace-suiyuan008/p/9985249.html)
