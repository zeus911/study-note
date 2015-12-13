更新ES 从1.7到2.0
===

1. Elasticsearch Migration Plugin `./bin/plugin -i elastic/elasticsearch-migration`

> 检查远程集群有点麻烦

2. 浏览 http://localhost:9200/_plugin/migration

3. 防止node 重启时 shard 重分配

```bash
curl -XPUT "0.0.0.0:9200/_cluster/settings" -d ' 
{
  "persistent": {
    "cluster.routing.allocation.enable": "none"
  }
}'
```
4.  停止indices并且发送同步冲洗请求 `curl -XPOST "0.0.0.0:9200/_flush/synced"`

5. 停掉集群 更新(备份配置)

>  wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

> echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list

> sudo rm /etc/apt/sources.list.d/elasticsearch-1.*

> sudo apt-get update && sudo apt-get upgrade elasticsearch

`!!!!!!!! 删除plugin!!!!!!`

5. 启动集群(单播) !!!(https://www.elastic.co/blog/hot-warm-architecture?q=warm )

> 先启动master 然后启动datanode 等其 变黄

> GET _cat/health
  
> GET _cat/nodes

6. 开启分配

```bash
curl -XPUT "0.0.0.0:9200/_cluster/settings" -d ' 
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  }
}'
```

