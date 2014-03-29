#!/bin/bash

CMDOUT_LOG="load.cmd.log"
STDOUT_LOG="load.std.log"
ERROR_LOG="load.err.log"
MAX_INITIAL=20.00
MAX=$MAX_INITIAL
RUNS=0
MAX_RUNS=12 # to get minutes: $MAX_RUNS / (60 / $SLEEP_TIME)
SLEEP_TIME=30
COMMAND="ps aux --sort=-pcpu"

# Background script if "--" isn't first argument
if [ "x$1" != "x--" ]; then
  $0 -- 1>> $STDOUT_LOG 2>> $ERROR_LOG &
  exit 0
fi

echo "[`date \"+%Y/%b/%d %H:%M:%S\"`] starting up..."

while true; do
  # Read in load average info
  read LOAD1 LOAD5 LOAD15 PROCS PROCID < /proc/loadavg
  
  if [ ${LOAD1/./} -gt ${MAX/./} ]; then
    TIMESTAMP=`date "+%Y/%b/%d %H:%M:%S"`
    echo "[$TIMESTAMP] load avg increased from $MAX to $LOAD1"
    echo -e "\n\n[$TIMESTAMP] Active Process Dump at $LOAD1 ==========================================" >> $CMDOUT_LOG
    $COMMAND >> $CMDOUT_LOG
    MAX=$LOAD1
  fi
  
  # Reset MAX to MAX_INITIAL after running MAX_RUNS times
  let RUNS+=1
  if [ $RUNS -gt $MAX_RUNS ]; then
    RUNS=1
    MAX=$MAX_INITIAL
  fi
  
  sleep $SLEEP_TIME
done
