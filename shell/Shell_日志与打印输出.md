# Shell 日志与打印输出
良好的 shell 脚本从记录日志内容，规范日志格式开始。
## 1
```bash
write_log()
{
    LOG_FILE=$LOGDIR/cckiller_$(date +%Y-%m-%d).log
    
    logout=""
    for((i=2;i<=$#;i++)); do 
        j=${!i}
        logout="${logout} $j "
    done
    
    if [[ $LOG_LEVEL == "INFO" ]] && [[ "$1" == "INFO" ]];then
        echo "[`date "+%Y-%m-%d %H:%M:%S"`][$1]: ${logout}" | tee -ai $LOG_FILE 
    
    elif [[ $LOG_LEVEL == "DEBUG" ]];then
        echo "[`date "+%Y-%m-%d %H:%M:%S"`][$1]: ${logout}" | tee -ai $LOG_FILE 
    
    else
        echo "[`date "+%Y-%m-%d %H:%M:%S"`][$1]: ${logout}"
    
    fi
    
}
```
