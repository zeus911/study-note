利用esdump 进行数据备份
===

```bash
#!/bin/bash

#yesterday=`date -d "-1 day" +%Y.%m.%d`
yesterday=`date  +%Y.%m.%d`
export PATH=$PATH:/usr/local/bin
log_file=/Users/calvin/Desktop/als-backup/als-backup-$yesterday.log
backup_dir=/Users/calvin/Desktop/als-backup/
INPUT_URL=http://0.0.0.0

# log 相关函数
GOOD='\033[32;01m'
WARN='\033[33;01m'
BAD='\033[31;01m'
HILITE='\033[36;01m'
BRACKET='\033[34;01m'
NORMAL='\033[0m'

error() {
  local msg=$*

  log "${BAD}Error: ${NORMAL} ${msg}"
}

warn() {
  local msg=$*

  log "${WARN}Warning: ${NORMAL} ${msg}"
}

info() {
  local msg=$*

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

cd $backup_dir
mkdir $yesterday
writeLog $? "create backup dir"
touch $log_file
writeLog $? "create log_file:$log_file"
log "begin to get $yesterday index"

indices=$(curl -XGET "0.0.0.0/_cat/indices/*-$yesterday*?h=index")
writeLog $? "get indices"

for index in $indices;do
	elasticdump     --input=$INPUT_URL/$index    --output=./$yesterday/${index}_mapping.json     --type=mapping
	writeLog $? "backup mapping:$index"
    elasticdump      --limit 1000      --input=$INPUT_URL/$index//      --output=./$yesterday/${index}.json      --type=data
	writeLog $? "backup data:$index"
done

tar -zcf als-backup_$yesterday.tar.gz ./$yesterday
writeLog $? "create tar file"

```
