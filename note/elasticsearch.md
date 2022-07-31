<!-- TOC -->

* [一、环境搭建](#)
    * [1.软件下载地址](#1)
    * [2.安装问题集锦](#2)
    * [3.学习文档](#3)
* [二、Elasticsearch](#elasticsearch)
    * [1. 基础语法](#1-)
        * [1.1数据删除](#11)
        * [1.2 搜索](#12-)

<!-- TOC -->

# 一、环境搭建

## 1.软件下载地址

https://www.elastic.co/cn/downloads/elasticsearch

## 2.安装问题集锦

- missing authentication credentials for REST request [/_cat/indices?v]

​ 关闭用户认证

```yaml
# Enable security features
xpack.security.enabled: false
```

## 3.学习文档

参考CSDN https://blog.csdn.net/u011863024/article/details/115721328#02_53

# 二、Elasticsearch

## 1. 基础语法

![img](https://img-blog.csdnimg.cn/img_convert/146a779da01f53e7f7a8d53132d3c7cf.png)

### 1.1数据删除

elasticsearch中的删除，并不会马上删除而只是标记删除。而删除的时间点是进行段合并的时候。可以顺着谈一下段合并，es是一个近实时的，因为在默认情况下，es
每秒进行一次刷新，刷新会产生一个段，而一次搜索是正是遍历所有的段，所以随着时间的推移，段的个数会越来越多。段合并不会把删除的数据写入新段，段合并之后，之前删除的数据才会真正的删除从磁盘上。

段合并这一块有一个问题，那就是需要段合并的时候，需要额外磁盘空间。当段足够大的以后，剩余的空间不够进行段合并，那么就不进行段合并了，所以删除的哪些数据，也就永远占用磁盘了。

要想释放磁盘的空间，要使用forcemerge命令合并段减少分片中段数量、删除冗余数据。

1、优化所有索引：

POST http://localhost:9200/_forcemerge?only_expunge_deletes=true

2、优化单个索引：

POST http://localhost:9200/索引名/_forcemerge?only_expunge_deletes=true

### 1.2 查询

<p>GET 请求  URL:http://127.0.0.1:9200/shopping/_search</p>

- 基础查询
```json5
{
    "query": {
        // "字段名":"字段值",
        "match": {
            "category": "小米"
        },
        // 查询所有doc
        "match_all": {
        },
        // 查询指定字段
        "query": {
            "match_all": {
            }
        },
        "_source": [
            "title"
        ]
    }
}
```
- 分页查询
```json5
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2
}
```
- 查询排序
```json5
{
	"query":{
		"match_all":{}
	},
	"sort":{
		"price":{
			"order":"desc"
		}
	}
}

```

- 多条件查询
> **match** 相当于数据库的 **&&** 
```json5
{
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
```
> **should** 相当于数据库的 **||**
```json5
{
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
      }]
    },
    "filter":{
      "range":{
        "price":{
          "gt":2000
        }
      }
    }
  }
}
```
---

> bool 查询
- must 文档必须匹配 must 选项下的查询条件，相当于逻辑运算的 AND，且参与文档相关度的评分。
- should 文档可以匹配 should 选项下的查询条件也可以不匹配，相当于逻辑运算的 OR，且参与文档相关度的评分。
- must_not 与 must 相反，匹配该选项下的查询条件的文档不会被返回；需要注意的是，must_not 语句不会影响评分，它的作用只是将不相关的文档排除。
- filter 和 must 一样，匹配 filter 选项下的查询条件的文档才会被返回，但是 filter 不评分，只起到过滤功能，与 must_not 相反。





