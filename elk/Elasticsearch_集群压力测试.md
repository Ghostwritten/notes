

---
## 1. 压力测试
压力测试的目的

 - 容量规划 / 性能优化 / 版本间性能比较 / 性能问题诊断
 - 确定系统稳定性，考察系统功能极限和隐患

压力测试的方法与步骤

 - 测试计划（确定测试场景和测试数据集）
 - 脚本开发
 - 测试环境搭建（不同的软硬件配置） & 运行测试
 - 分析比较结果

## 2. 测试目标 & 测试数据
测试目标

 - 测试集群的读写性能 / 做集群容量规划
 - 对 ES 配置参数进行修改，评估优化效果
 - 修改 Mapping 和 Setting，对数据建模进行优化，并测试评估性能改进
 - 测试 ES 新版本，结合实际场景和老版本进行比较，评估是否进行升级

测试数据

 - 数据量 / 数据分布

## 3. 测试脚本
ES 本身提供了 `REST API`，所以，可以通过很多传统的性能测试工具

 - `Load Runner` （商业软件，支持录制 + 重放 + DSL ）
 - `JMeter` （ Apache 开源 ，Record & Play）
 - `Gatling` （开源，支持写 Scala 代码 + DSL）

专门为 Elasticsearch 设计的工具

 - ES Pref & Elasticsearch-stress-test
 - Elastic Rally

## 4. ES Rally 简介
Elastic 官方开源，基于 Python 3 的压力测试工具

 - [https://github.com/elastic/rally](https://github.com/elastic/rally)

性能测试结果比较： [https://elasticsearch-benchmarks.elastic.c...](https://elasticsearch-benchmarks.elastic.co/)
功能介绍

 - 自动创建，配置，运行测试，并且销毁 ES 集群
 - 支持不同的测试数据的比较，也支持将数据导入 ES 集群，进行二次分析
 - 支持测试时指标数据的搜集，方便对测试结果进行深度的分析

## 5. Rally 的安装以及入门
安装运行

 - Python 3.4+ 和 pip3 / JDK 8 /git 1.9+
 - 运行 pip3 install esrally
 - 运行 esrally configure

运行

 - 运行 esrally –distribution-version=7.1.0
 - 运行 1000 条测试数据： esrally –distribution-version=7.1.0 –test-mode

## 6. Rally 基本概念讲解

 - Tournament – 定义测试目标，由多个 race 组成
Esrally list races
 - Track – 赛道：测试数据和测试场景与策 略
[https://github.com/elastic/rallytracks](https://github.com/elastic/rallytracks)
esrally list tracks
 - Car– 执行测试方案
不同配置的 es 实例
 - Award – 测试结果和报告

## 7. Benchmark Reports

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210315142821741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 8. 运行一个测试
[https://esrally.readthedocs.io/en/stable/t...](https://esrally.readthedocs.io/en/stable/tournament.html)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210315142940913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 9. 什么是压测的流程
`Pipeline` 指的是压测的一个流程

 - `Esrally list pipelines`

默认的流程

 - From-source-complete
 - From-source-skip-build
 - From-distribution

## 10. 自定义 & 分布式测试
Car

 - [https://esrally.readthedocs.io/en/latest/c...](https://esrally.readthedocs.io/en/latest/car.html)
 - 使用自建的集群

Track

 - 自带的测试数据集：Nyc_taxis 4.5 G /logging 1.2G
 - 更多的测试数据集： https://github.com/elastic/rally-tracks

分布式测试

 - https://esrally.readthedocs.io/en/latest/r...
 - Benchmark-only （对已有的集群进行测试）

## 11. 实例：比较不同的版本的性能
测试

```bash
esrally race –distribution-version=6.0.0 –track=nyc_taxis  –challenge=append-noconflicts –user-tag=”version:6.0.0”
esrally race –distribution-version=7.1.0 –track=nyc_taxis –challenge=append-noconflicts –user-tag=”version:7.1.0”
```

比较结果

```bash
esrally list races
esrally compare –baseline=[6.0.0 race] –contender=[7.1.0 race]
```

## 12. 实例：比较不同 Mapping 的性能
测试

```bash
esrally race –distribution-version=7.1.0 –track=nyc_taxis –challenge=append-noconflicts –user-tag=”enableSource:true” –include-tasks=”type:index”
```

修改：benchmarks/tracks/default/nyc_taxis/mappings.json，修改 _source.enabled 为 false

```bash
esrally race –distribution-version=7.1.0 –track=nyc_taxis –challenge=append-noconflicts –user-tag=”enableSource:false” –include-tasks=”type:index
```

比较

```bash
esrally compare –baseline=[enableAll race] –contender=[disableAll race]
```

## 13. 实例：测试现有集群的性能
测试

```bash
esrally race –pipeline=benchmark-only –target-hosts=127.0.0.1:9200 –track=geonames -challenge=append-no-conflicts
```


