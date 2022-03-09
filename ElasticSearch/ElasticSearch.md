<h1 align="center">ElasticSearch</h1>

## 简介

**倒排索引与正排索引的区别：**

正排索引（传统）

| id   | content              |
| ---- | -------------------- |
| 1001 | my name is zhang san |
| 1002 | my name is li si     |

倒排索引

| keyword | id         |
| ------- | ---------- |
| name    | 1001, 1002 |
| zhang   | 1001       |

**`Elasticsearch` 里存储文档数据和关系型数据库 `MySQL` 存储数据的概念进行一个类比：**

![](https://raw.githubusercontent.com/isIvanTsui/img/master/20220309145749.png)

**新增文档的`URL`请求区别：**

|          版本          | 请求类型 | 请求URL                           |
| :--------------------: | -------- | --------------------------------- |
| `ElasticSearch6.7`以前 | POST     | `http://127.0.0.1:9200/ivan/user` |
|  `ElasticSearch6.7+`   | POST     | `http://127.0.0.1:9200/ivan/_doc` |

![image-20220309151234024](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20220309151234024.png)

> 注意：`ivan`是索引名，`user`是`Type`；由于`ElasticSearch6.7+`已经舍弃`Type`概念，所以添加文档的`URL`已经不再写类型，而是用关键字`_doc`表示。
>
> `6.7`以前，一个索引`ivan`只能拥有一个`user`类型，`ivan`索引里添加了`user`类型： `http://127.0.0.1:9200/ivan/user`成功，再添加`animal`类型：`http://127.0.0.1:9200/ivan/animal`将会报错

## 入门操作

### 索引的创建&查询&删除

- **创建索引：**

  创建一个`ivan`索引：

  ```
  请求URL：http://127.0.0.1:9200/ivan
  请求类型：PUT
  Response：{
      "acknowledged": true,
      "shards_acknowledged": true,
      "index": "ivan"
  }
  ```

- **查询索引**

  ```
  请求URL：http://127.0.0.1:9200/_cat/indices?v
  请求类型：GET
  Response：{
  	health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
  	yellow open   ivan  rXOtJJCESCayfu15pzpOsQ   1   1          0            0       208b           208b
  }
  
  ```

- **查询单个索引**

  ```
  请求URL：http://127.0.0.1:9200/ivan
  请求类型：GET
  Response：{
      "ivan": {
          "aliases": {},
          "mappings": {},
          "settings": {
              "index": {
                  "creation_date": "1646810542266",
                  "number_of_shards": "1",
                  "number_of_replicas": "1",
                  "uuid": "rXOtJJCESCayfu15pzpOsQ",
                  "version": {
                      "created": "7080099"
                  },
                  "provided_name": "ivan"
              }
          }
      }
  }
  ```

- **删除索引**

  ```
  请求URL：http://127.0.0.1:9200/ivan
  请求类型：DELETE
  Response：{
      "acknowledged": true
  }
  ```
  

-----



### 文档的CRUD

- **新增文档：**

  `Id`采用`ES`默认生成的方式新增：

  ```
  请求URL：http://127.0.0.1:9200/ivan/_doc
  请求类型：POST
  请求Body（Json）：{
      "title":"小米手机",
      "category":"小米",
      "images":"http://www.gulixueyuan.com/xm.jpg",
      "price":3999.00
  }
  Response: {
      "_index": "ivan",
      "_type": "_doc",
      "_id": "ft-9bX8BP3AvVKfjb5dj",
      "_version": 1,
      "result": "created",
      "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
      },
      "_seq_no": 4,
      "_primary_term": 1
  }
  ```

  自定义`Id`方式新增：

  ```
  请求URL：http://127.0.0.1:9200/ivan/_doc/1001
  请求方式：POST
  请求Body(Json):{
      "title":"小米手机",
      "category":"小米",
      "images":"http://www.gulixueyuan.com/xm.jpg",
      "price":3999.00
  }
  Response:{
      "_index": "ivan",
      "_type": "_doc",
      "_id": "1001",
      "_version": 4,
      "result": "updated",
      "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
      },
      "_seq_no": 5,
      "_primary_term": 1
  }
  ```

- **查询文档：**

  根据`Id`查询文档：

  ```
  请求URL：http://127.0.0.1:9200/ivan/_doc/1001
  请求方式：GET
  Response:{
      "_index": "ivan",
      "_type": "_doc",
      "_id": "1001",
      "_version": 4,
      "_seq_no": 5,
      "_primary_term": 1,
      "found": true,
      "_source": {
          "title": "小米手机",
          "category": "小米",
          "images": "http://www.gulixueyuan.com/xm.jpg",
          "price": 3999.00
      }
  }
  ```

  查询该索引下所有的文档数据：

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search
  请求方式：GET
  请求Body(Json):
  Response:{
      "took": 101,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 3,
              "relation": "eq"
          },
          "max_score": 1.0,
          "hits": [
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "fd-YbX8BP3AvVKfj-5dU",
                  "_score": 1.0,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 3999.00
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "ft-9bX8BP3AvVKfjb5dj",
                  "_score": 1.0,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 3999.00
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1001",
                  "_score": 1.0,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 3999.00
                  }
              }
          ]
      }
  }
  ```

- **修改文档**

  全解修改：

  ```
  请求URL：http://127.0.0.1:9200/ivan/_doc/1001
  请求方式：POST
  请求Body(Json):{
      "title":"HUAWEI手机",
      "category":"HUAWEI",
      "images":"http://www.gulixueyuan.com/xm.jpg",
      "price":3999.00
  }
  Response:{
      "_index": "ivan",
      "_type": "_doc",
      "_id": "1001",
      "_version": 5,
      "result": "updated",
      "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
      },
      "_seq_no": 6,
      "_primary_term": 1
  }
  ```

  局部修改：

  ```
  请求URL：http://127.0.0.1:9200/ivan/_update/1001
  请求方式：POST
  请求Body(Json):{
  	"doc": {
  		"title":"小米手机",
  		"category":"小米"
  	}
  }
  Response:{
      "_index": "ivan",
      "_type": "_doc",
      "_id": "1001",
      "_version": 6,
      "result": "updated",
      "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
      },
      "_seq_no": 7,
      "_primary_term": 1
  }
  ```

- **删除文档**

  ```
  请求URL：http://127.0.0.1:9200/ivan/_doc/1001
  请求方式：DELETE
  Response:{
      "_index": "ivan",
      "_type": "_doc",
      "_id": "1001",
      "_version": 7,
      "result": "deleted",
      "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
      },
      "_seq_no": 8,
      "_primary_term": 1
  }
  ```

-----

### 高级查询

- **URL带参查询**

  查找`category`为小米的文档：

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search?q=category:小米
  请求方式：GET
  Response:{
      "took": 92,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 2,
              "relation": "eq"
          },
          "max_score": 1.4208536,
          "hits": [
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1001",
                  "_score": 1.4208536,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 3999.00
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1002",
                  "_score": 1.4208536,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 2999.00
                  }
              }
          ]
      }
  }
  ```

- **请求体带参查询**

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search
  请求方式：GET
  请求Body(Json):{
  	"query":{
  		"match":{
  			"category":"小米"
  		}
  	}
  }
  Response:{
      "took": 1,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 2,
              "relation": "eq"
          },
          "max_score": 1.4208536,
          "hits": [
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1001",
                  "_score": 1.4208536,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 3999.00
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1002",
                  "_score": 1.4208536,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 2999.00
                  }
              }
          ]
      }
  }
  ```

- **请求体方式的查找所有内容**

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search
  请求方式：GET
  请求Body(Json):{
  	"query":{
  		"match_all":{}
  	}
  }
  Response:
  ```

- **查询指定字段**

  如果你不想查询出所有的字段

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search
  请求方式：GET
  请求Body(Json):{
  	"query":{
  		"match_all":{}
  	},
  	"_source":["title"]
  }
  Response:{
      "took": 2,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 4,
              "relation": "eq"
          },
          "max_score": 1.0,
          "hits": [
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1001",
                  "_score": 1.0,
                  "_source": {
                      "title": "小米手机"
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1002",
                  "_score": 1.0,
                  "_source": {
                      "title": "小米手机"
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1003",
                  "_score": 1.0,
                  "_source": {
                      "title": "华为手机"
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1004",
                  "_score": 1.0,
                  "_source": {
                      "title": "华为手机"
                  }
              }
          ]
      }
  }
  ```

- **分页查询**

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search
  请求方式：GET
  请求Body(Json):{
  	"query":{
  		"match_all":{}
  	},
  	"from":1,
  	"size":2
  }
  Response:{
      "took": 9,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 4,
              "relation": "eq"
          },
          "max_score": 1.0,
          "hits": [
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1002",
                  "_score": 1.0,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 2999.00
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1003",
                  "_score": 1.0,
                  "_source": {
                      "title": "华为手机",
                      "category": "HUAWEI",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 5999.00
                  }
              }
          ]
      }
  }
  ```

- **查询排序**

  如果你想通过排序查出价格最高的手机：

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search
  请求方式：GET
  请求Body(Json):{
  	"query":{
  		"match_all":{}
  	},
  	"sort":{
  		"price":{
  			"order":"desc"
  		}
  	}
  }
  Response:{
      "took": 49,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 4,
              "relation": "eq"
          },
          "max_score": null,
          "hits": [
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1003",
                  "_score": null,
                  "_source": {
                      "title": "华为手机",
                      "category": "HUAWEI",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 5999.00
                  },
                  "sort": [
                      5999.0
                  ]
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1004",
                  "_score": null,
                  "_source": {
                      "title": "华为手机",
                      "category": "HUAWEI",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 4999.00
                  },
                  "sort": [
                      4999.0
                  ]
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1001",
                  "_score": null,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 3999.00
                  },
                  "sort": [
                      3999.0
                  ]
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1002",
                  "_score": null,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 2999.00
                  },
                  "sort": [
                      2999.0
                  ]
              }
          ]
      }
  }
  ```

- **多条件查询**

  假设想找出小米牌子，价格为3999元的

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search
  请求方式：GET
  请求Body(Json):{
  	"query":{
  		"bool":{
  			"must":[{
  				"match":{
  					"category":"小米"
  				}
  			},{
  				"match":{
  					"price":3999.00
  				}
  			}]
  		}
  	}
  }
  Response:{
      "took": 15,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 1,
              "relation": "eq"
          },
          "max_score": 2.4208536,
          "hits": [
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1001",
                  "_score": 2.4208536,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 3999.00
                  }
              }
          ]
      }
  }
  ```

- **范围查询**

  假设想找出小米和华为的牌子，价格大于3000元的手机。

  ```
  请求URL：http://127.0.0.1:9200/ivan/_search
  请求方式：GET
  请求Body(Json):{
  	"query":{
  		"bool":{
  			"should":[{
  				"match":{
  					"category":"小米"
  				}
  			},{
  				"match":{
  					"category":"华为"
  				}
  			}],
              "filter":{
              	"range":{
                  	"price":{
                      	"gt":3000
                  	}
  	            }
      	    }
  		}
  	}
  }
  Response:{
      "took": 33,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 3,
              "relation": "eq"
          },
          "max_score": 1.4208536,
          "hits": [
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1001",
                  "_score": 1.4208536,
                  "_source": {
                      "title": "小米手机",
                      "category": "小米",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 3999.00
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1003",
                  "_score": 0.0,
                  "_source": {
                      "title": "华为手机",
                      "category": "HUAWEI",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 5999.00
                  }
              },
              {
                  "_index": "ivan",
                  "_type": "_doc",
                  "_id": "1004",
                  "_score": 0.0,
                  "_source": {
                      "title": "华为手机",
                      "category": "HUAWEI",
                      "images": "http://www.gulixueyuan.com/xm.jpg",
                      "price": 4999.00
                  }
              }
          ]
      }
  }
  ```

------

### 全文检索

这功能像搜索引擎那样，如品牌输入“小华”，返回结果带回品牌有“小米”和华为的。

```
请求URL：http://127.0.0.1:9200/ivan/_search
请求方式：GET
请求Body(Json):{
	"query":{
		"match":{
			"category" : "小华"
		}
	}
}
Response:{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 4,
            "relation": "eq"
        },
        "max_score": 0.83740485,
        "hits": [
            {
                "_index": "ivan",
                "_type": "_doc",
                "_id": "1003",
                "_score": 0.83740485,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 5999.00
                }
            },
            {
                "_index": "ivan",
                "_type": "_doc",
                "_id": "1004",
                "_score": 0.83740485,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 4999.00
                }
            },
            {
                "_index": "ivan",
                "_type": "_doc",
                "_id": "1001",
                "_score": 0.83740485,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 3999.00
                }
            },
            {
                "_index": "ivan",
                "_type": "_doc",
                "_id": "1002",
                "_score": 0.83740485,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 2999.00
                }
            }
        ]
    }
}
```

### 完全匹配

```
请求URL：http://127.0.0.1:9200/ivan/_search
请求方式：GET
请求Body(Json):{
	"query":{
		"match_phrase":{
			"category" : "为"
		}
	}
}
Response:{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 0.83740485,
        "hits": [
            {
                "_index": "ivan",
                "_type": "_doc",
                "_id": "1003",
                "_score": 0.83740485,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 5999.00
                }
            },
            {
                "_index": "ivan",
                "_type": "_doc",
                "_id": "1004",
                "_score": 0.83740485,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 4999.00
                }
            }
        ]
    }
}
```

### 高亮查询

```json
请求URL：http://127.0.0.1:9200/ivan/_search
请求方式：GET
请求Body(Json):{
	"query":{
		"match_phrase":{
			"category" : "为"
		}
	},
    "highlight":{
        "fields":{
            "category":{}//<----高亮这字段
        }
    }
}
Response:{
    "took": 111,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 0.83740485,
        "hits": [
            {
                "_index": "ivan",
                "_type": "_doc",
                "_id": "1003",
                "_score": 0.83740485,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 5999.00
                },
                "highlight": {
                    "category": [
                        "华<em>为</em>"//<------高亮一个为字。
                    ]
                }
            },
            {
                "_index": "ivan",
                "_type": "_doc",
                "_id": "1004",
                "_score": 0.83740485,
                "_source": {
                    "title": "华为手机",
                    "category": "华为",
                    "images": "http://www.gulixueyuan.com/xm.jpg",
                    "price": 4999.00
                },
                "highlight": {
                    "category": [
                        "华<em>为</em>"
                    ]
                }
            }
        ]
    }
}
```

-----

### 映射关系

1. **先创建一个索引：** `PUT http://127.0.0.1:9200/user`

2. **再创建一个映射：**

   ```json
   请求URL：http://127.0.0.1:9200/user/_mapping
   请求方式：PUT
   请求Body(Json):{
       "properties": {
           "name": {
               "type": "text",
               "index": true
           },
           "sex": {
               "type": "keyword",
               "index": true
           },
           "tel": {
               "type": "keyword",
               "index": false
           }
       }
   }
   Response:{
       "acknowledged": true
   }
   ```

   查询一下映射：

   ```
   请求URL：http://127.0.0.1:9200/user/_mapping
   请求方式：GET
   Response:{
       "user": {
           "mappings": {
               "properties": {
                   "name": {
                       "type": "text"
                   },
                   "sex": {
                       "type": "keyword"
                   },
                   "tel": {
                       "type": "keyword",
                       "index": false
                   }
               }
           }
       }
   }
   ```

3. 添加数据

   ```
   请求URL：http://127.0.0.1:9200/user/_create/1001
   请求方式：PUT
   请求Body(Json):{
   	"name":"小米",
   	"sex":"男的",
   	"tel":"1111"
   }
   Response:{
       "_index": "user",
       "_type": "_doc",
       "_id": "1001",
       "_version": 1,
       "result": "created",
       "_shards": {
           "total": 2,
           "successful": 1,
           "failed": 0
       },
       "_seq_no": 0,
       "_primary_term": 1
   }
   ```

4. 查询

   查找`name`含有”小“数据：

   ```sjon
   请求URL：http://127.0.0.1:9200/user/_search
   请求方式：GET
   请求Body(Json):{
   	"query":{
   		"match":{
   			"name":"小"
   		}
   	}
   }
   Response:{
       "took": 311,
       "timed_out": false,
       "_shards": {
           "total": 1,
           "successful": 1,
           "skipped": 0,
           "failed": 0
       },
       "hits": {
           "total": {
               "value": 1,
               "relation": "eq"
           },
           "max_score": 0.2876821,
           "hits": [
               {
                   "_index": "user",
                   "_type": "_doc",
                   "_id": "1001",
                   "_score": 0.2876821,
                   "_source": {
                       "name": "小米",
                       "sex": "男的",
                       "tel": "1111"
                   }
               }
           ]
       }
   }
   ```

   查找`sex`含有”男“数据：

   ```json
   #GET http://127.0.0.1:9200/user/_search
   {
   	"query":{
   		"match":{
   			"sex":"男"
   		}
   	}
   }
   ```

   > 找不想要的结果，只因创建映射时`sex`的类型为`keyword`。
   >
   > `sex`只能完全为”男的“，才能得出原数据。

   ```json
   #GET http://127.0.0.1:9200/user/_search
   {
   	"query":{
   		"match":{
   			"sex":"男的"
   		}
   	}
   }
   ```

   

