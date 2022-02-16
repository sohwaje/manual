# CentOS 7/8에서 HA 구성

### 구성
- NGINX web server 2EA, VIP 1EA
- web01 10.0.0.2, web02 10.0.0.3
- VIP(Floating IP) 10.0.0.4

### /etc/host 수정
```
vim /etc/hosts
10.0.0.2 web01
10.0.0.3 web02
```

### ping test
```
ping -c 3 web01
ping -c 3 web02
```

### 패키지 설치
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

### ***hacluster*** 계정 패스워드 구성
```
passwd hacluster
Enter new password:
```

### 클러스터 구성
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

### NGINX 테스트 페이지 생성
```
#Run Command on 'web01'
echo '<h1>web01</h1>' > /usr/share/nginx/html/index.html

#Run Command on 'web02'
echo '<h1>web02</h1>' > /usr/share/nginx/html/index.html
```

### 리소스 생성
```
pcs resource create test_vip ocf:heartbeat:IPaddr2 ip=10.0.0.4 cidr_netmask=32 op monitor interval=30s
pcs resource create webserver ocf:heartbeat:nginx configfile=/etc/nginx/nginx.conf op monitor timeout="5s" interval="5s"
pcs status resources

# NGINX에 마이그레이션 임계값을 설정하여 자동으로 새 노드로 마이그레이션 되도록 설정(실패 회수 4)
pcs resource show webserver
pcs resource update webserver meta migration-threshold="4"
```

### 고착성 수치 설정(서버 장애 복구 후 자동 복구 방지)
- CentOS7
```
pcs property set default-resource-stickiness=1000
```
- CentOS8
```
pcs resource defaults update resource-stikiness=1000
```

### VIP 리소스와 webserver 리소스를 본딩시킴
- CentOS7
```
pcs constraint colocation add test_vip webserver INFINITY
```
- CentOS8
```
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
