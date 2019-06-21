---
title: x-pack之graph研究
date: 2018-07-22 18:44:57
tags: research
categories:
- elasticsearch
---
# x-pack之graph研究
## 简介
Graph 功能是一种基于 API 和基于 UI 的工具，能够让您数据中存在的相关关系浮现出来，同时能够在任何规模下利用各项 Elasticsearch 功能，例如分布式查询执行、实时数据可用性和索引等。

- **反欺诈**：挖掘海量购物行为数据和用户画像，探究由哪位店主对一组信用卡被盗事件负责。
- **个性化推荐**：根据听众偏好为喜欢莫扎特的听众推荐下一首最佳歌曲， 从而吸引和取悦听众。
- **安全分析**：发现潜在的破坏分子和其他意想不到的相关事件， 如挖掘网络中的主机与之进行通信的外部 IP的关联关系。

## 场景实战

### 业务场景

针对反欺诈场景，对商家打分作弊，马甲风险评估

### 准备分析数据

运行python IndexReviews.py 将review 数据写入到es 中


```
{
      "date": "2006-04-22 13:48",
      "reviewer": "15300",
      "rating": 5,
      "hour": "2006-04-22 13",
      "seller": "74"
}
```

IndexReviews.py 代码如下：
```
import csv
from elasticsearch import helpers
from elasticsearch.client import Elasticsearch
import sys
reload(sys)
sys.setdefaultencoding('utf8')

es = Elasticsearch(
[
        'http://10.16.238.101:9200/'
]
)
indexName = "reviews"
es.indices.delete(index=indexName, ignore=[400, 404])
indexSettings = {
    "settings": {
        "index.number_of_replicas": 0,
        "index.number_of_shards": 1
    },
    "mappings": {

        "review": {
            "properties": {
                "reviewer": {
                    "type": "keyword"
                },
                "seller": {
                    "type": "keyword"
                },
                "rating": {
                    "type": "integer"
                },
                "date": {
                    "type": "date",
                    "format": "yyyy-MM-dd HH:mm"
                },
                "hour": {
                    "type": "keyword"
                }
            }
        }

    }
}
es.indices.create(index=indexName, body=indexSettings)

actions = []
rowNum = 0
csvFilename = "reviews.csv"
with open(csvFilename, 'rb') as csvfile:
    csvreader = csv.reader(csvfile)
    for row in csvreader:
        rowNum += 1
        if rowNum == 1:
            continue

        action = {
            "_index": indexName,
            '_op_type': 'index',
            "_type": "review",
            "_source": {
                "reviewer": row[0],
                "seller": row[1],
                "rating": int(row[2]),
                "date": row[3],
                "hour": row[3][:-3],
            }
        }
        actions.append(action)
        # Flush bulk indexing action if necessary
        if len(actions) >= 5000:
            print rowNum
            helpers.bulk(es, actions)
            del actions[0:len(actions)]

if len(actions) >= 0:
    helpers.bulk(es, actions)
    del actions[0:len(actions)]
```
### 使用xpack graph 分析

1. 创建index
创建用于输出结果的index,
2. 使用es api 获取前50000个seller 
3. 调用graph 针对每个seller进行关系分析
例如
```
POST review/_xpack/graph/_explore
{
        "query": {
            "bool": {
                "must": [
                    {
                        "term": {"seller": "A9JN"}
                    },
                    {
                        "match": {"rating": 5}
                    }
                ]
            }
        },
        "controls": {
            "sample_size": 20,
            "use_significance": true
        },
        "vertices": [
            {
                "field": "customer",
                "size": 50,
                "min_doc_count": 1
            }
        ],
        "connections": {
            "query": {
                "term": {
                    "seller": "A9JN"
                }
            },
            "vertices": [
                {
                    "field": "hour",
                    "size": 500,
                    "min_doc_count": 1
                }
            ]
        }
    }
```

4. 根据查询的图关系进行客户端图计算，查找出reviewer ---->hour---->reviewer出现一次以上的路径相框，将该结果汇总。
5. 将第四步的结果组织存储到es 中
```
{
            "url": workspaceUrl,
            "seller": sellerId,
            "riskType": "SockPuppetry",
            "riskRating": numCoincidences,
            "numDocs": sellerDetails["numReviews"]
}
```


完整代码参考一下：
```
import networkx as nx
from elasticsearch import helpers
from elasticsearch.client import Elasticsearch
import sys
import urllib
import json

es = Elasticsearch(
[
        'http://10.16.238.101:9200/'
]
)

alertsIndexName = "alerts"

def createAlertsIndex():
    alertsIndexSettings = {
        "settings": {
            "number_of_replicas": "0",
            "number_of_shards": "1"
        },
        "mappings": {
            "task": {
                "properties": {
                    "riskType": {
                        "type": "keyword"
                    },
                    "url": {
                        "type": "text",
                        "index": "false"
                    }
                }
            }
        }
    }
    if not es.indices.exists(index=alertsIndexName):
        es.indices.create(index=alertsIndexName, body=alertsIndexSettings)

def getSellersList():
    sellers=[]
    sellersQuery = {
        "size": 0,
        "aggs": {
            "topTerms": {
                "terms": {
                    "field": "seller",
                    "size": 50000
                }
            }
        }
    }
    results = es.search(index="reviews", body=sellersQuery)["aggregations"]["topTerms"]["buckets"]
    for bucket in results:
        sellers.append({
            "seller" : bucket["key"],
            "numReviews" : bucket["doc_count"]
        })
    return sellers

def nodeId(node):
    return node["field"] + ":" + node["term"]
    
def erasePrint(msg):
    sys.stdout.write('\r')
    sys.stdout.write(msg)
    sys.stdout.flush()

createAlertsIndex()
sellers=getSellersList()
rowNum = 0;
totalNumReviews=0
for sellerDetails in sellers:
    rowNum +=1
    totalNumReviews+=sellerDetails["numReviews"]
    sellerId=sellerDetails["seller"]
    q = {
        "query": {
            "bool": {
                "must": [
                    {
                        "term": {"seller": sellerId}
                    },
                    {
                        "match": {"rating": 5}
                    }
                ]
            }
        },
        "controls": {
            "sample_size": 2000,
            "use_significance": True
        },
        "vertices": [
            {
                # Find most loyal reviewers - significantly connected to the current seller
                "field": "reviewer",
                "size": 50,
                "min_doc_count": 1
            }
        ],
        "connections": {
            # Find date/times of loyal reviewers' activity - guide the exploration so only current-seller-related reviews
            "query": {
                "term": {
                    "seller": sellerId
                }
            },
            # This will naturally favour date/times that are common to multiple reviewers
            "vertices": [
                {
                    "field": "hour",
                    "size": 500,
                    "min_doc_count": 1
                }
            ]
        }
    }

    erasePrint("Examining seller " + str(rowNum) + " of " + str(len(sellers)))

    results = es.transport.perform_request('POST', "/reviews/_xpack/graph/_explore", body=q)

    # Use NetworkX to create a client-side graph of reviewers and date/times we can analyze
    G = nx.Graph()

    for node in results["vertices"]:
        G.add_node(nodeId(node), type=node["field"])
    for edge in results["connections"]:
        n1 = results["vertices"][int(edge["source"])]
        n2 = results["vertices"][int(edge["target"])]
        G.add_edge(nodeId(n1), nodeId(n2))

    # Examine all "islands" of reviewers connected by same date/time
    subgraphs = nx.connected_component_subgraphs(G)

    numCoincidences = 0
    for subgraph in subgraphs:
        numHours = 0
        reviewers = []
        for n, d in subgraph.nodes(data=True):
            if d["type"] == "hour":
                numHours += 1
            if d["type"] == "reviewer":
                reviewers.append(n)
        if numHours > 1:
            if len(reviewers) > 1:
                for srcI in range(0, len(reviewers)):
                    reviewer1 = reviewers[srcI]
                    for targetI in range(srcI + 1, len(reviewers)):
                        reviewer2 = reviewers[targetI]
                        paths = nx.all_simple_paths(subgraph, source=reviewer1, target=reviewer2, cutoff=2)
                        #  Each path is a reviewer <-> date/time <-> reviewer triple
                        sameTimeReviews = 0
                        for path in paths:
                            sameTimeReviews += 1
                        # Two reviewers reviewing at same date/time is not a coincidence - however
                        # repeated synchronized reviews *are* a coincidence
                        if sameTimeReviews > 1:
                            numCoincidences += sameTimeReviews - 1

    if numCoincidences > 0:
        erasePrint("")
        print "seller:" + str(sellerId) + " has ", numCoincidences, " reviewer coincidences in", sellerDetails["numReviews"], "reviews"
        gq = json.dumps(q)
        workspaceUrl = "graph#/workspace/32935810-5e4d-11e8-811e-7b68199c5a31?query=" + urllib.quote_plus(gq)
        doc = {
            "url": workspaceUrl,
            "seller": sellerId,
            "riskType": "SockPuppetry",
            "riskRating": numCoincidences,
            "numDocs": sellerDetails["numReviews"]
        }
        res = es.index(index=alertsIndexName, doc_type='task', id=sellerId, body=doc)
erasePrint("")
print "Completed analysis of", len(sellers), "sellers and",totalNumReviews,"reviews"

```

## 注意
本文代码适用于python2.7,其他版本，请自行参照修改
## 参考
1. [Fraud detection using the elastic stack](https://www.youtube.com/watch?v=liMhiiyQ9co)