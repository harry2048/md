## sh调用接口

```sh
usage() {
	echo "全局流水号"
	echo "==="
	echo "Usage:"
	echo " [-h ip:port] [-g isLogDB] [-w isWriteDbSync] [-t waittime]"
	echo "Parameter:"
	echo " -h 服务的ip和端口(ip:port)(必输)"
	echo " -g 记录日志 (默认:true)"
	echo " -w 记录数据库日志是否同步 默认异步 (默认:false)"
	echo " -t 等待时间 (默认:10000)"
	echo "Example:"
	echo " $thisname psjzd.unl \"select fieldname,mycomment from psjzd\""
	echo " $thisname -g false -w true"
	echo
	exit 1
} 
SERVERIP=""
isLogDB=""
isWriteDbSync=""
waittime=""

while getopts :h:g:w:t: OPTION
do
	case $OPTION in
		h) SERVERIP=$OPTARG ;;
		g) isLogDB=$OPTARG ;;
		w) isWriteDbSync=$OPTARG ;;
		t) waittime=$OPTARG ;;
		?) usage ;;
	esac
done

URL="http://${SERVERIP}/webapi/globalserial/changeParameter/"
echo "SERVERIP:"$SERVERIP
echo "URL:"$URL
echo "isLogDB:"$isLogDB
echo "isWriteDbSync:"$isWriteDbSync
echo "waittime:"$waittime

HEADER="Content-type:application/json"
BODY={'"isLogDB"':'"'$isLogDB'"','"waittime"':'"'$waittime'"','"isWriteDbSync"':'"'$isWriteDbSync'"'}
echo "BODY:"$BODY

http_statecode=`curl -i -X POST -H $HEADER -d $BODY -s -o /dev/null -w %{http_code} $URL`
echo "http_statecode:"$http_statecode
if [[ "$http_statecode" =~ ^20[0-9]+ ]]; then
	echo "ChangeGlobalserial Success!"
	echo "0"
else
	echo "ChangeGlobalserial Failed!"
fi

```

## 精简版

```sh
aa=$1
result=$(curl -H "Content-type: application/json" -X POST -o /dev/null -s -w %{http_code} -d '[{"aa":"'${aa}'","bb":"0","cc":"0"}]' URL)
echo $result
```

> -o /dev/null 屏蔽原有输出信息
>
> -s silent 静音模式。不输出任何东西
>
> -w %{http_code} 控制额外输出 ${http_code} 返回码



















