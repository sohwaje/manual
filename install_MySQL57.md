    # Install MySQL5.7 Binary Version
```
    $ groupadd mysql
    $ useradd -r -g mysql -s /bin/false mysql


    $ tar xvfz mysql-advanced-5.7.19-linux-glibc2.12-x86_64.tar.gz


    $ ln -s /home/mysql-advanced-5.7.19-linux-glibc2.12-x86_64 /mysql


    mkdir -p /data/mysqldata/
    mkdir -p /data/mysqllog/error/
    mkdir -p /data/mysqllog/binary/
    mkdir -p /data/mysqllog/slow/
    mkdir -p /data/mysqllog/redo/
    mkdir -p /data/mysqllog/undo/
    chown -R mysql.mysql /data/
```

```
    # 엔진 새로 설치(임시 루트 패스워드 생성)
    $ ./bin/mysqld --basedir=/mysql --datadir=/mysql_data/data --initialize --user=mysql
    
    # 엔진 이관 설치, 변경 설치(임시 루트 패스워드 생성)
    $ ./bin/mysqld --defaults-file=/home/pdm/my.cnf --basedir=/home/pdm/mysql --datadir=/home/pdm/data/mysqldata --initialize --user=mysql


    $ bin/mysql_ssl_rsa_setup


    $ cp support-files/mysql.server /etc/init.d/mysql.server # 옵션


    # mysql.err 파일에 MySQL root 임시 패스워드가 출력된다. 이 패스워드로 접속하여 root 패스워드를 반드시 변경한다.
    $ bin/mysqld_safe --defaults-file=/etc/my.cnf --user=mysql &


    # MySQL 임시 루트 패스워드 변경
    mysql> alter user 'root'@'localhost' identified by 'tlrhdaleldj!@#';
    mysql> flush privileges;
```
----------

— 데이터베이스 생성 → 사용자 계정 생성 → 모든 권한 부여
```
    # 데이터베이스 생생
    mysql> CREATE DATABASE 'NEW_DATABASE' CHARACTER SET utf8 COLLATE utf8_bin;
    
    # 사용자 계정 생성
    mysql> CREATE USER 'NEW_DATABASE_USER'@'localhost' IDENTIFIED BY 'USER_PASSWORD' PASSWORD EXPIRE NEVER;
    
    # 권한 부여
    mysql> GRANT ALL PRIVILEGES ON 'NEW_DATABASE'.* TO 'NEW_DATABASE_USER'@'localhost';
```