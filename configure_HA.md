# CentOS 7/8에서 HA 구성

### 구성
- NGINX 2EA, VIP 1EA
- web01(10.0.0.2), web02(10.0.0.3)
- Floating IP(10.0.0.4)

### /etc/hosts 수정(both)
- /etc/hosts 파일에 노드의 별칭과 IP를 입력하여 클러스터가 노드의 별칭으로 통신할 수 있도록 설정한다.
```
vim /etc/hosts
10.0.0.2 web01
10.0.0.3 web02
```

### ping test(both)
- 클러스터에서 네트워크 이슈를 감지하는 방법은 ping, ethmonitor이다.
- ping을 사용하기 위해서는 노드 간 ICMP 패킷이 허용되어야 한다.
```
ping -c 3 web01
ping -c 3 web02
```

### 패키지 설치(both)
- CentOS7
```
yum -y install epel-release
yum -y install corosync pacemaker pcs
systemctl enable pcsd
systemctl enable corosync
systemctl enable pacemaker
systemctl start pcsd
yum -y install nginx
```
- CentOS8
```
dnf install pcs pacemaker fence-agents-all
systemctl enable pcsd
systemctl enable corosync
systemctl enable pacemaker
systemctl start pcsd
yum -y install nginx
```

### ***hacluster*** 계정 패스워드 구성(both)
- 클러스터 노드 간 통신하기 위해 서로를 식별할 수 있는 ID/PW
```
passwd hacluster
Enter new password:
```

### 클러스터 구성원을 등록한다.
- 클러스터 이름과 노드 이름은 대문자가 아니라 소문자로 할 것을 권고한다.
- CentOS7
```
pcs cluster auth web01 web02
Username: hacluster
Password: hacluster
pcs cluster setup --name test_cluster web01 web02
pcs cluster start --all
pcs cluster enable --all
```
- CentOS8
```
pcs host auth web01 web02
Username: hacluster
Password: hacluster
pcs cluster setup test_cluster --start web01 web02
pcs cluster enable --all
```

### 클러스터 상태 체크
```
pcs status cluster
```

### Property 설정
```
pcs property set symmetric-cluster=true
pcs property set no-quorum-policy=stop
pcs property set stonith-enabled=false
pcs property set stop-orphan-resources=true
pcs property set stop-orphan-actions=true
pcs property list
```

### NGINX 테스트 페이지 생성(both)
```
#Run Command on 'web01'
echo '<h1>web01</h1>' > /usr/share/nginx/html/index.html

#Run Command on 'web02'
echo '<h1>web02</h1>' > /usr/share/nginx/html/index.html
```

### 리소스 생성
#### [1]일반적인 VIP와 APP의 리소스 등록
- eth0에 VIP를 할당한다. 모든 노드에 동일한 이더넷 포트 이름이 있어야 한다.
```
# VIP 리소스
pcs resource create test_vip ocf:heartbeat:IPaddr2 ip=10.0.0.4 cidr_netmask=32 nic=eth0 op monitor interval=30s
# webserver 리소스
pcs resource create webserver ocf:heartbeat:nginx configfile=/etc/nginx/nginx.conf op monitor timeout="5s" interval="5s"
# 리소스 상태 확인
pcs status resources

# NGINX에 마이그레이션 임계값을 설정하여 자동으로 새 노드로 마이그레이션 되도록 설정(실패 회수 4)
pcs resource show webserver
pcs resource update webserver meta migration-threshold="4"
```
#### [2]systemd app과 VIP를 그룹으로 등록
- 그룹으로 등록할 경우 [VIP 리소스와 systemd app을 묶는 설정은 하지 않아도 된다.](https://github.com/sohwaje/manual/blob/main/configure_HA.md#vip-%EB%A6%AC%EC%86%8C%EC%8A%A4%EC%99%80-webserver-%EB%A6%AC%EC%86%8C%EC%8A%A4%EB%A5%BC-%EB%B3%B8%EB%94%A9%EC%8B%9C%ED%82%B4vip%EC%99%80-webserver%EB%A5%BC-%EB%8B%A8%EC%9D%BC-%EB%85%B8%EB%93%9C%EB%A1%9C-%EB%AC%B6%EB%8A%94%EB%8B%A4)
```
# VIP 리소스를 p_cluster 그룹으로 등록
pcs resource create test_vip ocf:heartbeat:IPaddr2 ip=10.0.0.4 cidr_netmask=32 nic=eth0 op monitor interval=30s --group p_cluster
# systemd app 등록
pcs resource create app_name systemd:systemd-app op monitor timeout=3s interval=5s --group p_cluster
```

### 고착성 수치 설정(서버 장애 복구 후 자동 복구 방지)
- 고착성 수치를 1000으로 설정해서 다른 노드로 리소스가 넘어가지 않도록 설정한다.
#### webserver
- CentOS7
```
pcs property set default-resource-stickiness=1000
pcs resource defaults
```
- CentOS8
```
pcs resource defaults update resource-stikiness=1000
pcs resource defaults
```
#### VIP
- VIP 모니터링 리소스를 생성하고 VIP가 다른 노드로 넘어가지 않도록 고착성을 설정한다.
```
# vip 모니터 리소스 생성
pcs resource create VIP-monitor ethmonitor interface=eth0 clone
pcs resource
# 서비스의 다운타임 최소화를 위해 Auto Failback을 방지
pcs constraint location vip rule score=-INFINITY ethmonitor-eth0 ne 1 
```
### VIP 리소스와 webserver 리소스를 본딩시킴(VIP와 webserver를 단일 노드로 묶는다.)
```
> pcs constraint colocation add <resource id> with <resource id> score
pcs constraint colocation add test_vip with webserver INFINITY
```

### VIP up -> webserver up
```
pcs constraint order test_vip then the webserver
```

### 클러스터 시작
```
pcs cluster stop --all
pcs cluster start --all
```

### 설정 점검
```
pcs status nodes
corosync-cmapctl | grep members
pcs status corosync
```

### Fail Over 테스트
```
pcs cluster stop web01
pcs status
```

#### 실패 회수 확인
```
pcs resource failcount show webserver
```
#### 실패 회수 재설정: [장애 복구(fail back) 방지 설정을 하지 않으면 복구된 Active 서버로 원복된다.](https://github.com/sohwaje/manual/blob/main/configure_HA.md#%EA%B3%A0%EC%B0%A9%EC%84%B1-%EC%88%98%EC%B9%98-%EC%84%A4%EC%A0%95%EC%84%9C%EB%B2%84-%EC%9E%A5%EC%95%A0-%EB%B3%B5%EA%B5%AC-%ED%9B%84-%EC%9E%90%EB%8F%99-%EB%B3%B5%EA%B5%AC-%EB%B0%A9%EC%A7%80)
```
pcs resource cleanup webserver
```
