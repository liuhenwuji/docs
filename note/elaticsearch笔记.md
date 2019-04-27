#elaticsearch 本地集群方式
1.bin/elasticsearch
2.bin/elasticsearch -Ecluster.name=my_cluster -Enode.name=monde1 -Ehttp.port=8200 -Epath.data=node2
2.bin/elasticsearch -Ecluster.name=my_cluster -Enode.name=monde2 -Ehttp.port=7200 -Epath.data=node3

#查看集群
localhost:8200/_cat/nodes
http://localhost:8200/_cluster/stats


#filebeat
head -n 2 ~/nginx_logs/nginx.log| ./filebeat -e -c nginx.yml
#packetbeat
sudo ./packetbeat -e -c es.yml -strict.perms=false

#monitor
elasticsearch -Ecluster.name=sniff_search -Ehttp.port=8200 -Epath.data-sniff
kibana -e http://127.0.0.0.1:8200 -p 8601  

search
q=username:alfred way		way是匹配所有字段
q=username:"alfred way"		alfred way前后顺利短语
q=username:(alfred way)		username=alfred or way

~1 近似度

分析查询过程
GET test_search_index/_search?q=alfred
{
  "profile":true
}

分析算分过程(是基于shard集合计算，实验时要把shard设置为1）
GET test_search_index/_search?q=alfred
{
  "explain":true
}

PUT test_search_index
{
	"settings":{
		"index":{
			"number_of_shards":"1"
		}
	}
}

BM25相比TF/IDF的一大优化是降低了tf在过大时的权重

elasticsearch-plugin install x-pack
kibana-plugin install x-pack