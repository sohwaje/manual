- bash 스크립트로 만들어진 [데몬](https://github.com/sohwaje/shell_scripts/blob/master/process/panopticon.sh)을 systemd에 등록해보겠다.
```
vim /etc/systemd/system/panopticon.service
[Unit]
Description=Test Daemon

[Service]
User=root
Group=root
ExecStart=/root/panopticon.sh start
ExecStop=/root/panopticon.sh stop

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
systemctl enable panopticon.service
systemctl start panopticon.service
```
