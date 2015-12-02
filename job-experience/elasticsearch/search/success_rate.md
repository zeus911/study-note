利用`scripted_metric`进行成功（比率）率查询
===

### 核心

> init_scripr 生成初始变量
> map_script  设置过滤条件
> combine_script 分片聚合
> reduce_script shard聚合

```json
 {
   "size":0,
   "aggregations":{
        "diff_term":{
            "terms":{
                "field":"data.params.req.exercise_id",
                "size":10000000
            },
            "aggs":{
                  "f_rate": {
                    "scripted_metric": {
                       "init_script" : "_agg[\"err_ids\"]=[0,0.0];",
                       "map_script" : "id=\"\"+doc[\"data.params.req.exercise_id\"].value;score=0;if(id && doc[\"data.params.data.result.score\"].value){if(doc[\"data.params.data.result.score\"].value<60){score=1};_agg[\"err_ids\"][0]+=1;_agg[\"err_ids\"][1]+=score}",
                       "combine_script" : "return _agg[\"err_ids\"]",
                       "reduce_script" : "result_array=[0,0]; for(i in _aggs){result_array[0]=result_array.get(0)+i[0];result_array[1]=result_array.get(1)+i[1]};return result_array[1]/result_array[0] "
                                      }
                             }
                }
        }
   }
 }
```

### 实例：获取成功率存入新的index
```ruby
require 'elasticsearch'
require 'elasticsearch/transport'

class DailyFailRate
  
  def initialize
    @client = Elasticsearch::Client.new url:'localhost'
    @local_client = Elasticsearch::Client.new
    @time=Time.now-86400
    @yesterday =@time.strftime('%Y.%m.%d')
    @terms=['exercise_id','passport_id']
  end

  def search(term)
    result=@client.search(
        index:"als-552b873f6d6f6e176d040000-#{@yesterday}",
        body:{
            size:0,
            aggregations:{
                cs_data:{
                    filter:{
                        term:{
                            "data.params.data.target"=>"cs"
                        }
                    },
                    aggs:{
                        diff_term:{
                            terms:{
                                field:"data.params.req.#{term}",
                                size:10000000
                            },
                            aggs:{
                                f_rate: {
                                    scripted_metric: {
                                        init_script: "_agg[\"err_ids\"]=[0,0.0];",
                                        map_script: "id=\"\"+doc[\"data.params.req.#{term}\"].value;score=0;if(id && doc[\"data.params.data.result.score\"].value){if(doc[\"data.params.data.result.score\"].value<60){score=1};_agg[\"err_ids\"][0]+=1;_agg[\"err_ids\"][1]+=score}",
                                        combine_script: "return _agg[\"err_ids\"]",
                                        reduce_script: "result_array=[0,0]; for(i in _aggs){result_array[0]=result_array.get(0)+i[0];result_array[1]=result_array.get(1)+i[1]};return result_array[1]/result_array[0] "
                                    }
                                }
                            }
                        }
                    }
                }

            }
        }
    )
    return result
  end
  
  def main
    @terms.each do |term|
      @key="data.params.req.#{term}"
      result=search(term)
      save(term,result)
    end
  end
  
  def save(term,result)
    body=[]
    count=0
    result['aggregations']['cs_data']['diff_term']['buckets'].each do |value|
      count+=1
      body<<{ index:  { _index: 'als-dsr-fail-rate-static-'+@yesterday, _type: term, data: {'@timestamp':@time.strftime('%Y-%m-%dT11:11:11.111+08:00'),fail_rate:value['f_rate']['value'],doc_count:value['doc_count'],@key=>value['key']} }}
      bulk(body) && body.clear if(count%100==0)
    end
    bulk body unless body.empty?
    #puts count
  end
  
  def bulk(body)
    @client.bulk body:body
  end
end

DailyFailRate.new.main
```
`[Back](/)`