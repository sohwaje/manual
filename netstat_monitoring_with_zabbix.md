## [1] netstat를 이용해서 정보를 수집하는 스크립트 생성
```
vim /zabbix/plugins/tcp_status.sh
#################Script content#################
#!/bin/bash
if [ $# -ne 1 ];then
    echo "Follow the script name with an argument "
fi

case $1 in

    LISTEN)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/LISTEN/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    ESTAB)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/ESTAB/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;


    CLOSE-WAIT)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/CLOSE-WAIT/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    TIME-WAIT)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/TIME-WAIT/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    SYN-SENT)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/SYN-SENT/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    SYN-RECV)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/SYN-RECV/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    FIN-WAIT-1)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/FIN-WAIT-1/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    FIN-WAIT-2)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/FIN-WAIT-2/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    UNCONN)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/UNCONN/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    LAST-ACK)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/LAST-ACK/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;

    CLOSING)
        result=`ss -ant | awk 'NR>1 {a[$1]++} END {for (b in a) print b,a[b]}' | awk '/CLOSING/{print $2}'`
        if [ "$result" == "" ];then
               echo 0
        else
           echo $result
        fi
        ;;
 esac
```
## [2] 자빅스 에이전트 설정
```
vim /zabbix/etc/zabbix_agentd.conf.d/userparameter.conf
Unsafeuserparameters = 1
UserParameter=tcp.status[*],sh /zabbix/plguins/tcp_status.sh $1
```

```
systemctl restart zabbix_agentd
```
## [3] 자빅스 템플릿 만들기
- Configuration>Templates>Create template
![screenshot](https://imgs.developpaper.com/imgs/2021610142602749.png)

## [4] 그래프 구성
- Configuration>Template>템플릿이름>Graph>Create Graph
![screenshot](https://imgs.developpaper.com/imgs/2021610142709102.jpg)

## [5] 결과
![screenshot](https://imgs.developpaper.com/imgs/2021610142759622.jpg)

### ref
https://developpaper.com/linux-uses-ss-command-combined-with-zabbix-to-monitor-sockets/
