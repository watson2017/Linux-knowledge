```
#!/bin/sh                                                                                                                                                   
                                                                                                                                                            
#--------------------------------------------                                                                                                               
# 描述：Bash脚本公用方法                                                                                                                                    
# 作者：Watson.wang                                                                                                                   
#--------------------------------------------                                                                                                               
                                                                                                                                                            
##### Bash Set机制 开始 #####                                                                                                                               
# 启用错误触发，退出Bash                                                                                                                                    
set -o errexit                                                                                                                                              
                                                                                                                                                            
# 启用使用未初始化变量时，退出Bash                                                                                                                          
set -o nounset                                                                                                                                              
##### Bash Set机制 结束 #####                                                                                                                               
                                                                                                                                                            
                                                                                                                                                            
##### 变量定义 开始 #####                                                                                                                                   
current_dir=`pwd`                                                                                                                                           
now_time=`date "+%Y%m%d"`

# 全局日志文件名
g_log_file=""

# 日志文件最大值100M
log_file_size_bytes=104857600

# 日志前缀
LOG_PREFIX=""

# 日志存放目录
LOG_FOLEDER="${current_dir}/logs"
##### 变量定义 结束  #####


##### 返回码定义 开始 #####
FILE_NAME_NULL=100
FILE_NOT_EXISTED=101
CREATE_LOG_FILE_FAILED=102
##### 返回码定义 结束  #####

log_info() {
    local run_time="`date '+%Y-%m-%d %H:%M:%S'`"
    local func_name=$1
    local msg_or_rc=$2
    printf "\e[32m%s - [INFO] - run_user[${USER}] - func_name[%s] - msg_or_rc[%s]\e[0m\n" "${run_time}" ${func_name} "${msg_or_rc}"
}

log_warn() {
    local run_time="`date '+%Y-%m-%d %H:%M:%S'`"
    local func_name=$1
    local msg_or_rc=$2
    printf "\e[33m%s - [WARN] - run_user[${USER}] - func_name[%s] - msg_or_rc[%s]\e[0m\n" "${run_time}" ${func_name} "${msg_or_rc}"
}


log_error() {
    local run_time="`date '+%Y-%m-%d %H:%M:%S'`"
    local func_name=$1
    local msg_or_rc=$2
    printf "\e[31m%s - [ERROR] - run_user[${USER}] - func_name[%s] - msg_or_rc[%s]\e[0m\n" "${run_time}" ${func_name} "${msg_or_rc}"
}

#--------------------------------------------
# 功能：检查日志文件是否需要切割
#--------------------------------------------
rorate_log_file() {
    local func_name="rorate_log_file"
    local last_log_file_name=$1
    cd ${LOG_FOLEDER}
    if [[ -f ${last_log_file_name} ]];then
        local real_time_file_size=`ls -l ${last_log_file_name} | awk '{print $5}'`

        if [ ${real_time_file_size} -gt ${log_file_size_bytes} ];then
            local file_name_prefix=${last_log_file_name%.*}
            g_log_file=${LOG_FOLEDER}/${file_name_prefix}`date "+%H%M%S"`.log
        else
            g_log_file=${LOG_FOLEDER}/${last_log_file_name}
        fi
    else
        log_error "${func_name}" "${FILE_NOT_EXISTED}"
        exit 1
    fi
}



#--------------------------------------------
# 功能：获取当前日期日志文件名称
# 说明：
#   msg_or_rc 表示消息或上条命令执行的返回值
#--------------------------------------------
get_last_log_file_name() {
    local func_name="get_last_log_file_name"
    cd ${LOG_FOLEDER}
    local last_log_file_name=`ls -l | grep $1 | grep ${now_time} | sort -k8rn | head -1 |awk '{print $9}'`
    if [ -z ${last_log_file_name} ];then
        # 创建日志文件
        local log_file_local=$1${now_time}.log
        local abs_log_file=${LOG_FOLEDER}/${log_file_local}
        touch ${abs_log_file}

        if [[ $? -eq 0 ]];then
            echo ${log_file_local}
        else
            return ${CREATE_LOG_FILE_FAILED}
        fi
    fi
    echo ${last_log_file_name}
}

#--------------------------------------------
# 功能：日志文件预检查
#--------------------------------------------
inspect_log_file() {
    local func_name="inspect_log_file"

    if [[ ! -d "${LOG_FOLEDER}" ]];then
        mkdir -p "${LOG_FOLEDER}"
    fi

    local last_log_file_name=`get_last_log_file_name $1`
    if [[ ${last_log_file_name} == 102 ]];then
        log_error "${func_name}" "${CREATE_LOG_FILE_FAILED}"
        exit 1
    fi
    rorate_log_file ${last_log_file_name}
}

#--------------------------------------------
# 功能：日志工具接口
# 参数：
#   arg1：执行的方法名
#   arg2：日志文件前缀，区分多种部署组件脚本
#   arg3：表示消息或上条命令执行的返回值
#--------------------------------------------
log_utils() {
    local main_func_name=$1
    local log_prefix=$2
    # 命令行输入日志级别自动转换为大写
    local LOG_LEVEL=`echo $3 | tr a-z A-Z`
    local MSG_OR_RC=$4

    # 日志文件环境检查
    inspect_log_file ${log_prefix}

    case ${LOG_LEVEL} in
        WARN)
            log_warn "${main_func_name}" "${MSG_OR_RC}" | tee -a ${g_log_file}
            ;;
        INFO)
            log_info "${main_func_name}" "${MSG_OR_RC}" | tee -a ${g_log_file}
            ;;
        ERROR)
            log_error "${main_func_name}" "${MSG_OR_RC}" | tee -a ${g_log_file}
            ;;
        *)
            log_error "${main_func_name}" "Not support log level."
            exit 1
            ;;
    esac
}

#--------------------------------------------
# 功能：记录错误级别日志
# 参数：
#   arg1：执行的方法名
#   arg2：表示消息或上条命令执行的返回值
#--------------------------------------------
LOG_ERR() {
    local method_name="$1"
    local message="$2"
    log_utils "${method_name}" "${LOG_PREFIX}" "ERROR" "${message}"
}

#--------------------------------------------
# 功能：记录信息级别日志
# 参数：
#   arg1：执行的方法名
#   arg2：表示消息或上条命令执行的返回值
#--------------------------------------------
LOG_INFO() {
    local method_name="$1"
    local message="$2"
    log_utils "${method_name}" "${LOG_PREFIX}" "INFO" "${message}"
}

#--------------------------------------------
# 功能：记录告警级别日志
# 参数：
#   arg1：执行的方法名
#   arg2：表示消息或上条命令执行的返回值
#--------------------------------------------
LOG_WARN() {
    local method_name="$1"
    local message="$2"
    log_utils "${method_name}" "${LOG_PREFIX}" "WARN" "${message}"
}

```
