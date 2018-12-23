```
#!/bin/bash

DAT=`date +%F`     # 所有的变量中必须放在第一位
PATH="/usr/local/crm.server"
BACKUP_PATH="/root/pkgs/bakcup"
REPLIC_PATH="/tmp/temp"
REMOTE_HOST="10.205.2.x"
TEMP_PATH="/root/tmp"

# 判断执行返回结果
Execution_result(){
    local method_name="$1"
    local right_message="$2"
    local error_message="$3"
    if [[ $? -eq 0 ]]; then
            echo "${method_name}" "${right_message}"
    else
            echo "${method_name}" "${error_message}"
            exit 1
    fi
}

# 备份项目
Backup_program(){
	local program_name="backup_program"
	local msg_or_rc=$1
	cd "${PATH}"/"${msg_or_rc}"
	/usr/bin/zip  -rq "${BACKUP_PATH}"/"${msg_or_rc}"/webapps_$DAT.zip    webapps/
	if [[ ！ -f  "${BACKUP_PATH}"/"${msg_or_rc}"/webapps_$DAT.zip ]]; then
		echo "$program_name $msg_or_rc  failed"
		exit 1
	else
		echo "$program_name $msg_or_rc success."
	fi
}  

# 从远端复制最新代码到临时目录
Replicate_program(){
	local program_name="Replicate_program"
	local msg_or_rc=$1
	/usr/bin/scp -rq "${REMOTE_HOST}":"${PATH}"/"${msg_or_rc}"/webapps  "${REPLIC_PATH}"/"${msg_or_rc}"
	Execution_result "${program_name}"  "scp program from $REMOTE_HOST success."  "scp program from $REMOTE_HOST failed."
}

# 停止Tomcat服务
Stop_tomcat(){
	local program_name="stop_tomcat"
	local msg_or_rc=$1
	is_actived=`ps -ef | grep "$PATH/$msg_or_rc" | grep -v grep`
	if [[ -n "${is_actived}" ]]; then
		ps -ef | grep "$PATH/$msg_or_rc" | grep -v grep | awk '{print $2}' | xargs kill -9
		sleep 3
		is_actived=`ps -ef | grep "$PATH/$msg_or_rc" | grep -v grep`
		if [[ -n  "${is_actived}" ]]; then
			echo "$program_name $msg_or_rc stop failed."
			exit 1
		else
			Execution_result "$program_name"  " stop $msg_or_rc success."  "stop $msg_or_rc failed"
		fi
	else
		echo "Service $msg_or_rc has been stopped before kill."
	fi
}

# 启动Tomcat服务
Start_tomcat(){
	local program_name="start_tomcat"
	local msg_or_rc=$1
	/bin/bash "${PATH}"/"${msg_or_rc}"/bin/startup.sh
	Execution_result "$program_name"  "start $msg_or_rc success."  "start $msg_or_rc failed."
}

# 替换项目文件
Replace_program(){
	local program_name="Replace_program"
	local msg_or_rc=$1
	if [[ -d "${TEMP_PATH}"/$DAT ]]; then
		mv "${PATH}"/"${msg_or_rc}"/webapps  "${TEMP_PATH}"/$DAT/"${msg_or_rc}"-webapps
		Execution_result "$program_name"  "remove old program packages to temp path success"
	else
		mkdir "${TEMP_PATH}"/$DAT
		mv "${PATH}"/"${msg_or_rc}"/webapps  "${TEMP_PATH}"/$DAT/"${msg_or_rc}"-webapps
		Execution_result "$program_name"  "remove old program packages to temp path success"
	fi

	mv "${REPLIC_PATH}"/"${msg_or_rc}"/webapps  "${PATH}"/"${msg_or_rc}"
	Execution_result "$program_name"  "remove new program packages to temp path success"
}

All(){
	msg_or_rc=$1
	Backup_program  "${msg_or_rc}"
	Replicate_program "${msg_or_rc}"
	Stop_tomcat  "${msg_or_rc}"
	Replace_program  "${msg_or_rc}" 
	Start_tomcat "${msg_or_rc}"
}

main(){
	case $1 in
		cca )
			All cca;
			;;
		crms )
			All crms;
			;;
		rqa )
			All rqa;
			;;
		spm )
			All spm;
			;;
		task )
			All task;
			;;
		web )
			All web;
			;;
		workflow )
			All workflow;
			;;
			*)
		echo $"Usage: $0 {cca | crms | rqa | spm | task | web | workflow}"
	esac
}
main $1
```
