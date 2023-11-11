
## 第一个
```bash
processName="test4.sh"
processNum=`ps -aef |grep "${processName}" | grep -v grep | wc -l`
 
if [ "${processNum}" -gt "2" ]; then
  echo "已经有脚本在运行，本脚本不支持多实例运行${processNum}"
  exit 1
fi
```

## 第二个

```bash
#!/bin/ksh
 
RUNDIR=`dirname $0`
PIDFILE="${RUNDIR}/$0.pid"
 
if [ -s ${PIDFILE} ]; then
  echo "脚本已经在运行，不重复运行，退出."
  exit 1
fi
echo $$ > ${PIDFILE}
 
<各种业务处理逻辑>
 
cat /dev/null > ${PIDFILE}
```

## 第三个

```bash
#!/bin/ksh
 
RUNDIR=`dirname $0`
PIDFILE="${RUNDIR}/$0.pid"
 
if [ -s ${PIDFILE} ]; then
   SPID=`cat ${PIDFILE}`
   if [ -e /proc/${SPID}/status ]; then
      echo "脚本已经在运行，不重复运行，退出."
      exit 1
  fi
  cat /dev/null > ${PIDFILE}
fi
echo $$ > ${PIDFILE}
 
#各种业务逻辑
 
cat /dev/null > ${PIDFILE}
```

