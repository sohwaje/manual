# Install GitLab 12.2.5 Community Version
깃랩에 필요한 패키지 설치
```
sudo yum install -y curl policycoreutils-python openssh-server
```

설치에 필요한 리포지토리 추가
```
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

yum으로 설치
```
sudo yum install gitlab-ce-12.2.5-ce.0.el7.x86_64
```

external_url 설정
```
vim /etc/gitlab/gitlab.rb
## GitLab URL
##! URL on which GitLab will be reachable.
##! For more details on configuring external_url see:
##! https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab
external_url 'http://gitlab.example.com'
```

gitlab reconfigure
```
gitlab-ctl reconfigure
```

# Manual Backup
디비 관련 프로세스 OFF
```
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
```
수동백업
```
gitlab-rake gitlab:backup:create
```
백업 파일 위치 : /var/opt/gitlab/backups

# Manual Restore
복원
```
gitlab-rake gitlab:backup:restore BACKUP=1636609641_2021_11_11_12.2.5
```
gitlab 재시작
```
gitlab-ctl start
gitlab-rake gitlab:satellites:create
gitlab-rake gitlab:check SANITIZE=true
```

# SSL/TLS 설정