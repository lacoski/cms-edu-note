# Hướng dẫn cài đặt CMS EDU trên Ubuntu 18.04

## Chuẩn bị

Lưu ý:
- Cài đặt Ubuntu 18.04 trắng

Cho phép tài khoản root được ssh
```
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/'g /etc/ssh/sshd_config
service sshd restart
```

Đặt mật khẩu cho tài khoản root
```
sudo su 

# Đặt passwd cho root user
passwd
Enter new UNIX password: <root_passwd>
Retype new UNIX password: <root_passwd>
```

Khởi động lại SSH
```
sudo service ssh restart
```

## Phần 1: Chuẩn bị môi trường cài đặt CMS

> Thực hiện với tài khoản `root`

### Bước 1: Cài đặt môi trường chạy CMS

```
sudo apt-get install -y build-essential openjdk-8-jdk-headless fp-compiler \
    postgresql postgresql-client python3.6 cppreference-doc-en-html \
    cgroup-lite libcap-dev zip
sudo apt-get install -y python3.6-dev libpq-dev libcups2-dev libyaml-dev \
    libffi-dev python3-pip
sudo apt-get install -y python3-pip
sudo apt-get install -y python-psycopg2
sudo apt-get install -y libpq-dev
sudo apt-get install -y libcups2-dev -y
sudo pip3 install gevent
sudo pip3 install py-bcrypt
```

### Bước 2: Tạo mới database PostgreSQL cho CMS Edu

Chuyển sang tài khoản Postgre
```
sudo su - postgres
```

Tạo mới User Postgre cho CMS
```
createuser --username=postgres --pwprompt cmsuser
```

Khởi tạo Database
```
createdb --username=postgres --owner=cmsuser cmsdb
psql --username=postgres --dbname=cmsdb --command='ALTER SCHEMA public OWNER TO cmsuser'
psql --username=postgres --dbname=cmsdb --command='GRANT SELECT ON pg_largeobject TO cmsuser'
exit
```


