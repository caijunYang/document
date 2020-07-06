Elasticsearch 使用kibana常用语法
=====================================

# ElasticSearch常用查询语法

##  1、	简单查询(单字段单条件查询)
  * 直接访问_search的查询，应用于对于es里面所有的数据查询，此种查询不会区分es中存在的数据归属于哪一个索引；默认是分页查询，默认只查询10条数据，如：
    ```json
      GET _search
        {
          "query": {
            "match_all": {}
          }
        }
    ```
  * 访问/索引名称_search的查询，应用于对于es里面针对于某个索引下的数据进行查询，不会查询到其余的索引下的数据，默认是分页查询，默认只查询10条数据；
  同样的也可以访问/索引名称/类型_search查询，此针对于查询索引下数据的一个补充，查询指定索引下某种类型的数据,如：

    请求：
    ```json
      GET /log_auto_info_2020.06.29/_search
      {
      "query": {
        "match_all": {}
      }
      }
    ```
    响应说明：
    ```json
      {
        "took":,                            请求花了多少时间
        "time_out":,				  	 有没有超时
        "_shards":{    						 执行请求时查询的分片信息
            "total":,     					 查询的分片数量
            "successful":,  				 成功返回结果的分片数量
            "failed":  					 失败的分片数量
        },
        "hits":{
            "total":,       				 查询返回的文档总数
            "max_score":,  			 计算所得的最高分
            "hits":[						 返回文档的hits数组
                {
                    "_index":,		 索引
                    "_type":,     		 属性(来源于mapping 的定义)
                    "_id":"1",      		 标志符
                    "_score":,      	 得分
                    "_source":{			  	 发送到索引的JSON对象

                    }
                }//可能会出现多个
            ]
        }
      }
    ```
##  2、简单多字段多条件and查询
      term 精确匹配查询，一般用于值完全一样的查询；match 用于模糊匹配（match在匹配时会对所查找的关键词进行分词，然后按分词匹配查找）
      注意：默认情况下term查询精确匹配时，如果不是关键字字段且含有大写字母，中文等非小写字母或数字的均完全匹配搜索不到, 大写字母可以用小写字母搜索到，最好不要使用 term 对类型是text的字段进行查询，是关键字字段时值完全匹配才能搜索到；match查询会根据分词匹配查找，默认分词是一字一分，词组是2字一分，也就是说用match 查询的时候只要能匹配上一个分词就会被查出来

      如：

      ```json
      GET /log_auto_error_2020.06.29/doc/_search
      {
        "query": {
          "bool": {
            "must": [
              {
                "term": {
                  "loglevel": "error"
                }
              },
              {
                "term": {
                  "account": "u8757"
                }
              }
            ]
          }
        }
      }

      ```
##  3、	match查询
    match_all (查询全部)简如第一点中所述
    match_phrase(短语查询)，match_phrase_prefix(最左前缀查询)这两种都可以根据数据中含有某一部分查找，match_phrase类似于%xxx%, match_phrase_prefix类似于 xx%
    multi_match(多字段查询)这是对match的补充，match也可完成多字段查询，但需要写多个match查询条件，而multi_match则只需一个查询指定多个字段，查询多个字段包含同一个关键字

    如：

    ```json
    {
      "query": {
        "multi_match": {
          "query": "关键字",
          "fields": ["字段1", "字段2"]
        }
      }
    }

    ```
    同样multi_match 甚至可以当作 match_phrase  和 match_phrase_prefix 使用，只是需要指定type类型，如当 match_phrase_prefix 使用则加一个 “type”: “phrase_prefix”

##  4、	简单聚合分组查询
  * 查询loglevel 字段分组对应的数量:

  ```json
  GET /log_auto_warn_2020.06.30/_search
    {
      "size": 0,
      "aggs": {
        "group_loglevel": {
          "terms": {
            "field": "loglevel"
          }
        }
      }
    }
  ```  

  * 查询 timeUsed(数字类型) 总和

    ```json
    GET _search
    {
      "size": 0,
      "aggs": {
        "timeUsed": {
          "sum": {
            "field": "timeUsed"
          }
        }
      }
    }
    ```
  * 查询loglevel字段分组对应的timeUsed(数字类型) 总和

  ```json
  GET _search
    {
      "size": 0,	// size为0标识只返回聚合结果数据，非0的时候会返回这个数值条数的原记录数据
      "aggs": {
        "group_loglevel": {
          "terms": {
            "field": "loglevel"
          },
          "aggs": {
            "timeUsed": {
              "sum": {
                "field": "timeUsed"
              }
            }
          }
        }
      }
    }
  ```
    分组字段不能是分词字段，如果是分词字段则分组报异常，查询无效；计算类型的计算也只能是数字类型的字段，如果是非数字类型的字段则报异常，计算无效

##  5、使用range 范围值查询
  匹配某一范围内的数据型、日期类型或者字符串型字段的文档，注意只能查询一个字段，不能作用在多个字段上。

  ```json
    GET _search
      {
        "query": {
          "range": {
            "timeUsed": {
              "gte": 1,
              "lte": 20
            }
          }
        }
      }
  ```
##  6、mapping模板
  在es或kibana的提供的查询界面可以用一下命令设置mapping存储模板，模板主要对index中的数据类型进行限制，如text类型不支持聚合。keyword、date、number等支持聚合，如果使用到类似的场景，需要强制设定mapping进行类型限制

  6.x版本:

  ```json
  PUT _template/logstash_self -- logstash_self 自定义的名称
    {
      "template": "log_*",   -- 匹配索引名称(可用* 模糊匹配)
      "order": 2147483647,		-- 匹配模板的权重, 当索引同时匹配上了多个 template ，那么就会先应用 order 数值小的 template 设置，然后再应用一遍 order 数值高的作为覆盖，最终达到一个 merge 的效果
      "version": 60002,		-- 版本号 随便取
      "index_patterns": [
        "log_*"
      ],					-- 匹配索引的表达式,与 template 模糊匹配是一样的效果
      "settings": {
        "index": {
          "refresh_interval": "5s"	-- 刷新一次数据间隔时间
        }
      },
      "mappings": {
        "_default_": {		-- es索引的type, 笔者当前6.8.10,_default_对应的type默认值是doc
          "dynamic_templates": [	-- 动态模板
            {
              "message_field": {
                "path_match": "message",	-- 匹配带message 的数据类型为text
                "mapping": {
                  "norms": false,
                  "type": "text"
                },
                "match_mapping_type": " string"
              }
            },
            {
              "string_fields": {
                "mapping": {
                  "norms": false,
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "ignore_above": 256,
                      "type": "keyword"
                    }
                  }
                },
                "match_mapping_type": " string",
                "match": "*"	-- 匹配所有的非设定的字段均为text类型
              }
            }
          ],
          "properties": {
            "@timestamp": {	-- 默认字段 @timestamp
              "type": "date"
            },
            "account": {	-- 配置设定的字段，
              "type": "keyword"
            },
            "loglevel": {
              "type": "keyword"
            },
            "responseCode": {
              "type": "keyword"
            },
            "server": {
              "type": "keyword"
            },
            "service_id": {
              "type": "keyword"
            },
            "sys": {
              "type": "keyword"
            },
            "timeUsed": {
              "type": "long"
            },
            "geoip": {
              "dynamic": true,
              "properties": {
                "ip": {
                  "type": "ip"
                },
                "latitude": {
                  "type": "half_float"
                },
                "location": {
                  "type": "geo_point"
                },
                "longitude": {
                  "type": "half_float"
                }
              }
            },
            "@version": {
              "type": "keyword"
            }
          }
        }
      },
      "aliases": {}
    }

  ```

  7.x版本:

  ```json
  PUT _template/logstash_self
    {
      "template": "*",
      "order": 2147483647,
      "version": 60002,
      "index_patterns": [
        "*"
      ],
      "settings": {
        "index": {
          "refresh_interval": "5s"
        }
      },
      "mappings": {
          "dynamic": true,
          "properties": {
            "@timestamp": {
              "type": "date"
            },
            "account": {
              "type": "keyword"
            },
            "loglevel": {
              "type": "keyword"
            },
            "responseCode": {
              "type": "keyword"
            },
            "server": {
              "type": "keyword"
            },
            "service_id": {
              "type": "keyword"
            },
            "sys": {
              "type": "keyword"
            },
            "timeUsed": {
              "type": "long"
            },
            "@version": {
              "type": "keyword"
            }
          }
      },
      "aliases": {}
    }

  ```

##  7、java使用聚合查询

* 6.x版本：

    代码实现如4.3的聚合查询(查询loglevel为INFO并且server为auto，以loglevel值分组聚合查询出各类型各有多少数据条数和各类型timeUsed之和总值，实现类似于sql 的 select loglevel, count(0), sum(timeUsed) from table where loglevel=’INFO’ and server=’auto’ group by loglevel)

```java
      // 创建一个查询条件对象
  BoolQueryBuilder queryBuilder = QueryBuilders.boolQuery();
  // 拼接查询条件
  queryBuilder.must(QueryBuilders.termQuery("loglevel", "INFO"));
  queryBuilder.must(QueryBuilders.termQuery("server", "auto"));
  SumAggregationBuilder aggTimeUsed =
          AggregationBuilders.sum("group_timeUsed").field("timeUsed");
  // 创建聚合查询条件
  TermsAggregationBuilder agg =
          AggregationBuilders.terms("group_loglevel").field("loglevel");
  //组合 分组之后 再分组
  agg.subAggregation(aggTimeUsed);
  // 创建查询对象
  SearchQuery build = new NativeSearchQueryBuilder()
          //添加查询条件
          .withQuery(queryBuilder)
          // 添加聚合条件
          .addAggregation(agg)
          .build();

  Aggregations testEntities = elasticsearchTemplate.query(build, new ResultsExtractor<Aggregations>() {
      @Override
      public Aggregations extract(SearchResponse response) {
          return response.getAggregations();
      }
  });

```
    testEntities里面包含的就是聚合查询返回的结果（其内可能是多个Aggregation），其中结果中第一层可以转化为Terms然后获取其中的Bucket 再从Bucket中获取第一层的分组值和数量，如果Bucket中还有内层 Aggregations 那么 再循环 这个Aggregations以便于获取内层的聚合结果名称和数据，但这一层的 Aggregation 在笔者用的 spring-boot-starter-data-elasticsearch 2.0.6 版本上 不能再转换成 Terms，只能转换为InternalSum来获取数据如下：

```java
    Map<String, Aggregation> dataMap = testEntities.asMap();
    for (Map.Entry<String, Aggregation> entry : dataMap.entrySet()) {
        Terms term = (Terms) entry.getValue();
        if (term.getBuckets().size() > 0) {
            for (Terms.Bucket bk : term.getBuckets()) {
                String key = bk.getKeyAsString();
                log.info("keyAsString: {}", key);
                long docCount = bk.getDocCount();
                log.info("num: {}", docCount);
                Aggregations aggregations = bk.getAggregations();
                if(null != aggregations){
                    Map<String, Aggregation> map = aggregations.asMap();
                    for(Map.Entry<String, Aggregation> entry1 : map.entrySet()){
                        double value = ((InternalSum)entry1.getValue()).getValue();
                        int num = Double.valueOf(value).intValue();
                        log.info("key: {}", entry1.getKey());
                        log.info("value:{}", num);
                    }
                }
            }
        }
    }
```
    经笔者简单验证，查询语句中用了aggs 之后在内层最多只能有一个aggs，且内层的aggs只能是计算或数据去重 筛选等，如果想再要加第3层aggs是不行的，语句执行就会抛异常【也有可能是笔者没找到相应的写法，望知之者指教

  * 7.x版本：

```java
    // 创建一个查询条件对象
      BoolQueryBuilder queryBuilder = QueryBuilders.boolQuery();
      // 拼接查询条件
      queryBuilder.must(QueryBuilders.termQuery("loglevel", "ERROR"));
      queryBuilder.must(QueryBuilders.termQuery("server", "auto"));
      SumAggregationBuilder aggTimeUsed =
              AggregationBuilders.sum("group_timeUsed").field("timeUsed");
      // 创建聚合查询条件
      TermsAggregationBuilder agg =
              AggregationBuilders.terms("group_loglevel").field("loglevel");
      //组合 分组之后 再分组
      agg.subAggregation(aggTimeUsed);
      // 创建查询对象
      NativeSearchQuery build = new NativeSearchQueryBuilder()
              //添加查询条件
              .withQuery(queryBuilder)
              // 添加聚合条件
              .addAggregation(agg)
              .build();


      IndexCoordinates of = IndexCoordinates.of(index);
      of = of.withTypes("doc");
      System.out.println(of.toString());
//        elasticsearchTemplate.
      SearchHits<Map> search = elasticsearchTemplate.search(build, Map.class, of);
      Aggregations aggregations = search.getAggregations();
```
