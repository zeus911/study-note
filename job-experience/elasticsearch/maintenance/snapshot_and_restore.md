### Snapshot And Restore

##### Snapshot 

```bash
 curl -XPUT 'http://als.shuobaotang.com/elasticsearch/_snapshot/day-backup?verify=false' -d '{
    "type": "s3",
    "settings": {
        "bucket": "als",
        "region": "cn-north-1",
        "base_path": "test-day"
    }
}'


curl -XPOST 'http://als.shuobaotang.com/elasticsearch/_snapshot/day-backup/_verify'

curl -XGET 'http://als.shuobaotang.com/elasticsearch/_snapshot/'

curl -XPUT 'http://als.shuobaotang.com/elasticsearch/_snapshot/day-backup/day-20151020' -d '
{
  "indices": "*2015.10.20",
  "ignore_unavailable": "true",
  "include_global_state": false
}'

curl -XGET 'http://als.shuobaotang.com/elasticsearch/_snapshot/es_repository/all-20151020?pretty'
```

#### Snapshot

```bash
curl -XPUT 'http://als.shuobaotang.com/elasticsearch/_snapshot/es_repository/day-20151020' -d '
{
  "indices": "2015.10.20",
  "ignore_unavailable": "true",
  "include_global_state": false
}'


GET /_snapshot/_status
GET /_snapshot/my_backup/_status
GET /_snapshot/my_backup/snapshot_1/_status
```

#### restore

```bash
curl -XPOST 'localhost:9200/_snapshot/day-backup/all-20151020/_restore' -d '{
 "indices": "*2015.01*",
 "index_settings": {
    "index.number_of_replicas": 0
  },
  "ignore_index_settings": [
    "index.refresh_interval"
  ],
  "ignore_unavailable": "true",
  "include_global_state": false
}'
```

#### 实例

```bash
#!/bin/bash

#log 相关函数

today=`date +%Y.%m.%d`
yesterday=`date -d "-1 day" +%Y.%m.%d`
log_file=/mnt/log-analysis/als-backup/log/auto-backup-${yesterday}.log
touch $log_file
GOOD='\033[32;01m'
WARN='\033[33;01m'
BAD='\033[31;01m'
HILITE='\033[36;01m'
BRACKET='\033[34;01m'
NORMAL='\033[0m'

error() {
  local msg="$*"

  log "${BAD}Error: ${NORMAL} ${msg}"
}

warn() {
  local msg="$*"

  log "${WARN}Warning: ${NORMAL} ${msg}"
}

info() {
  local msg="$*"

  log "Info: ${msg}"
}

log() {
  local msg="$*"
  if [ -n "${log_file}" -a -f "${log_file}" ]; then
    echo "$(date): ${msg}" >> ${log_file} 2>/dev/null
  fi
}

writeLog() {
	local error="$1"
	local msg="$2"

	if [ $error -gt 0 ]; then
		warn $msg
		exit -1
	fi
	info $msg

}

befor0day=`date   +%Y%m%d`
befor1day=`date -d "-1 day" +%Y%m%d`
befor2day=`date -d "-2 day" +%Y%m%d`
befor3day=`date -d "-3 day" +%Y%m%d`
befor4day=`date -d "-4 day" +%Y%m%d`
befor5day=`date -d "-5 day" +%Y%m%d`
befor6day=`date -d "-6 day" +%Y%m%d`
befor7day=`date -d "-7 day" +%Y%m%d`

snapshot() {
 local is_all=$1  # 1 all backup；2 day backup
 local indices=''
 local snapshot_name=''
 if [[ is_all -eq 1  ]]
 then
   indices="*,-$today"
   snapshot_name="all-$befor0day"
 else
   indices="$yesterday"
   snapshot_name="day-$befor1day"
 fi
 cmd="curl -XPUT '0.0.0.0/_snapshot/es_repository/$snapshot_name' -d ' \
{ \
  \"indices\": \"$indices\", \
  \"ignore_unavailable\": \"true\", \
  \"include_global_state\": false \
}'" 
 info $cmd
 result=`eval $cmd`
 if [[ $result =~ true ]]
 then
   info "backup accsep!"
 else
   warn "backup failed! info:[$result]" 
 fi
}

backup_finash() {
  local i=0
  local result=''
  while true
  do 
    sleep 60s
    i=`expr $i + 1`
    result=`curl 0.0.0.0/_snapshot/es_repository/all-$befor0day`
    if [[ $result =~ SUCCESS ]]
    then
      break
    fi
    if [[ $i -eq 10 ]] 
    then
      break
    fi
  done 
}

delete_snapshot() {
 local cmd=''
 info "begin delete snapshot"
 for snapshot_date in $befor2day $befor3day $befor4day $befor5day $befor6day 
 do 
   cmd="curl -XDELETE 0.0.0.0/_snapshot/es_repository/day-$snapshot_date/'"
   result=`eval $cmd`
   info "$cmd\t$result"
 done
 cmd="curl -XDELETE '0.0.0.0/_snapshot/es_repository/all-$befor7day/'"
 result=`eval $cmd`
 info "$cmd\t$result"
 info "end delete snapshot"
}

main() {
 dayinweek=`date '+%u'`

 if [[ $dayinweek -eq 2 ]] 
 then
  writeLog $? "Begin all backup! $yesterday"
  snapshot 1
  writeLog $? "end day backup! $yesterday"
   backup_finash
  delete_snapshot
 else
  writeLog $? "Begin day backup! $yesterday"
  snapshot 0
  writeLog $? "end day backup! $yesterday"
 fi
}

main

```