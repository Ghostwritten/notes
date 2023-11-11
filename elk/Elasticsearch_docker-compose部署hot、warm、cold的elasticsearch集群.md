```bash
version: '2.2'
services:
  cerebro:
    image: lmenezes/cerebro:0.8.3
    container_name: hwc_cerebro
    ports:
      - "9000:9000"
    command:
      - -Dhosts.0.host=http://elasticsearch:9200
    networks:
      - hwc_es7net
  kibana:
    image: docker.elastic.co/kibana/kibana:7.1.0
    container_name: hwc_kibana7
    environment:
      #- I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - "5601:5601"
    networks:
      - hwc_es7net
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_hot
    environment:
      - cluster.name=geektime-hwc
      - node.name=es7_hot
      - node.attr.box_type=hot
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_hot,es7_warm,es7_cold
      - cluster.initial_master_nodes=es7_hot,es7_warm,es7_cold
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - hwc_es7data_hot:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - hwc_es7net
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_warm
    environment:
      - cluster.name=geektime-hwc
      - node.name=es7_warm
      - node.attr.box_type=warm
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_hot,es7_warm,es7_cold
      - cluster.initial_master_nodes=es7_hot,es7_warm,es7_cold
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - hwc_es7data_warm:/usr/share/elasticsearch/data
    networks:
      - hwc_es7net
  elasticsearch3:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_cold
    environment:
      - cluster.name=geektime-hwc
      - node.name=es7_cold
      - node.attr.box_type=cold
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_hot,es7_warm,es7_cold
      - cluster.initial_master_nodes=es7_hot,es7_warm,es7_cold
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - hwc_es7data_cold:/usr/share/elasticsearch/data
    networks:
      - hwc_es7net


volumes:
  hwc_es7data_hot:
    driver: local
  hwc_es7data_warm:
    driver: local
  hwc_es7data_cold:
    driver: local

networks:
  hwc_es7net:
    driver: bridge
```



```bash
[root@master ~]# docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                              NAMES
65b5ce81d7e4        lmenezes/cerebro:0.8.3                                "/opt/cerebro/bin/ce…"   18 minutes ago      Up 10 minutes       0.0.0.0:9000->9000/tcp             hwc_cerebro
71fb90d3cbca        docker.elastic.co/elasticsearch/elasticsearch:7.1.0   "/usr/local/bin/dock…"   18 minutes ago      Up 10 minutes       0.0.0.0:9200->9200/tcp, 9300/tcp   es7_hot
99756b48b874        docker.elastic.co/elasticsearch/elasticsearch:7.1.0   "/usr/local/bin/dock…"   18 minutes ago      Up 10 minutes       9200/tcp, 9300/tcp                 es7_warm
bd401acb16de        docker.elastic.co/kibana/kibana:7.1.0                 "/usr/local/bin/kiba…"   18 minutes ago      Up 10 minutes       0.0.0.0:5601->5601/tcp             hwc_kibana7
387d968672da        docker.elastic.co/elasticsearch/elasticsearch:7.1.0   "/usr/local/bin/dock…"   18 minutes ago      Up 10 minutes       9200/tcp, 9300/tcp                 es7_cold
```

