批量更新字段
===

## 需求
>#### 按照data.params.data.result.sentence.split(/\s+/).size 修改 data.params.data.result.word_count

#### 更新单个document[语法](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)
#### bulk[语法](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

## 解决方式

  1. 单个document更新的语句</br>
  
>   curl -XPOST 'localhost:9200/als-552b873f6d6f6e176d040000-2015.09.07/als_production/AU-oSp7bq-7mSo1II33a/_update' -d '{
>     "script":"ctx._source.data.params.data.result.word_count=ctx._source.data.params.data.result.sentence.value.toString().split(/\\s+/).length"
>   }'

  2. 聚合查询出错误数据_id(错误  应该使用filter script 见最下方)
    
```json
 curl -XGET 'localhost:9200/als-552b873f6d6f6e176d040000-2015.09.07/_search?pretty' -d '
 {
    "size":0,
    "aggs":{
      "err_ids": {
        "scripted_metric": {
           "init_script" : "_agg[\"err_ids\"]=[];",
           "map_script" : "word_count=doc[\"data.params.data.result.word_count\"].value;sentence=doc[\"data.params.data.result.sentence\"].value.toString().split(/\\s+/).length;if(word_count!=sentence){_agg[\"err_ids\"].add(doc[\"_id\"].value)}",
           "combine_script" : "return _agg[\"err_ids\"]",
           "reduce_script" : "return _aggs"
                          }
                 }
    }
 }'
```

note: field class org.elasticsearch.index.fielddata.ScriptDocValues$Strings!!!

最后出现问题 在agg里取不到_id

```ruby

require 'elasticsearch'
require 'elasticsearch/transport'

client = Elasticsearch::Client.new url:'localhost:9200'
a=client.search(index:'als-552b873f6d6f6e176d040000-2015.09.31',body:{fields:[],size:41421})

bulk_body=[]
puts "search over"
a['hits']['hits'].each do |value|
  bulk_body<<{ update: { _index: value["_index"], _type: value["_type"], _id: value["_id"], data: { script:"ctx._source.data.params.data.result.word_count=ctx._source.data.params.data.result.sentence.value.toString().split(/\\s+/).length" } } }

  if (bulk_body.length%100 ==0)
    puts bulk_body.length
    puts "kaishi  bluk"
    client.bulk body:bulk_body
    bulk_body.clear
  end
end
puts bulk_body.length
client.bulk body:bulk_body if bulk_body.length !=0
```

  2.使用filter获取文档id
    
```json
 curl -XGET 'localhost:9200/als-552b873f6d6f6e176d040000-2015.09.07/_search?pretty' -d '
 {
    "size":10,
    "fields":[],
    "filter":{
      "script": {
           "script" : "(doc[\"data.params.data.result.word_count\"].value != doc[\"data.params.data.result.sentence\"].value.toString().split(/\\s+/).length)"
                 }
    }
 }'
```