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
sudo apt-get install -y python3-pip byobu
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

Kết quả
```
postgres@cmsedu:~$ createuser --username=postgres --pwprompt cmsuser
Enter password for new role: cloud3652020
Enter it again: cloud3652020
```

Khởi tạo Database
```
createdb --username=postgres --owner=cmsuser cmsdb
psql --username=postgres --dbname=cmsdb --command='ALTER SCHEMA public OWNER TO cmsuser'
psql --username=postgres --dbname=cmsdb --command='GRANT SELECT ON pg_largeobject TO cmsuser'
exit
```

Kết quả
```
postgres@cmsedu:~$ createuser --username=postgres --pwprompt cmsuser
Enter password for new role: 
Enter it again: 
postgres@cmsedu:~$ createdb --username=postgres --owner=cmsuser cmsdb
postgres@cmsedu:~$ psql --username=postgres --dbname=cmsdb --command='ALTER SCHEMA public OWNER TO cmsuser'
ALTER SCHEMA
postgres@cmsedu:~$ psql --username=postgres --dbname=cmsdb --command='GRANT SELECT ON pg_largeobject TO cmsuser'
GRANT
postgres@cmsedu:~$ exit
logout
root@cmsedu:~# 
```

### Bước 3: Tạo mới User dành cho cms

Lưu ý:
- User `cms` sẽ là User sử dụng để cài đặt và chạy source code cms

Thực hiện
```
adduser cms
```

Kết quả
```
root@cmsedu:~# adduser cms
Adding user `cms' ...
Adding new group `cms' (1001) ...
Adding new user `cms' (1001) with group `cms' ...
Creating home directory `/home/cms' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: cloud3652020
Retype new UNIX password: cloud3652020
passwd: password updated successfully
Changing the user information for cms
Enter the new value, or press ENTER for the default
	Full Name []: ENTER
	Room Number []: ENTER
	Work Phone []: ENTER
	Home Phone []: ENTER
	Other []: ENTER
Is the information correct? [Y/n] y
```

Cấp quyền sudo cho User `cms`
```
usermod -aG sudo cms
```

## Phần 2: Cài đặt CMS

> Thực hiện với quyền user `cms`

### Bước 1: Chuyển từ user `root` sang user `cms`
```
su - cms
```

Kết quả
```
root@cmsedu:~# su - cms
cms@cmsedu:~$
```

### Bước 2: Tải source code cms và giải nén

```
wget https://github.com/cms-dev/cms/releases/download/v1.4.rc1/v1.4.rc1.tar.gz
tar -xvf v1.4.rc1.tar.gz
```

Kết quả
```
cms@cmsedu:~$ ls -lh
total 1.1M
drwxr-xr-x 14 cms cms 4.0K Nov  1  2018 cms
-rw-rw-r--  1 cms cms 1.1M Nov  1  2018 v1.4.rc1.tar.gz
```

### Bước 3: Cài đặt project cms

Truy cập đường dân `/home/cms/cms/config`
```
cd /home/cms/cms/config
ls -lh
```

Kết quả:
```
cms@cmsedu:~/cms/config$ ls -lh
total 20K
-rw-r--r-- 1 cms cms 6.4K Nov  1  2018 cms.conf.sample
-rw-r--r-- 1 cms cms  467 Nov  1  2018 cms.ranking.conf.sample
-rw-r--r-- 1 cms cms 5.4K Nov  1  2018 nginx.conf.sample
```

Tạo file config
```
cp cms.ranking.conf.sample cms.ranking.conf
cp cms.conf.sample cms.conf
```

Chỉnh sửa kết nối database tại file `cms.conf` (Dòng 59) thành user và mật khẩu PostgreSQL đã tạo ở phần 1
```
cms@cmsedu:~/cms/config$ cat -n cms.conf | grep database
    58	    "_help": "Connection string for the database.",
    59	    "database": "postgresql+psycopg2://cmsuser:cloud3652020@localhost:5432/cmsdb",
    62	    "database_debug": false,
```

### Bước 4: Khởi tạo settings mặc định

```
cd /home/cms/cms
sudo python3 prerequisites.py install
```

Kết quả
```
cms@cmsedu:~/cms$ sudo python3 prerequisites.py install
[sudo] password for cms: 
===== Creating group cmsuser
===== Creating user cmsuser
===== Compiling isolate
gcc -std=gnu99 -Wall -Wextra -Wno-parentheses -Wno-unused-result -Wno-missing-field-initializers -Wstrict-prototypes -Wmissing-prototypes -D_GNU_SOURCE -DVERSION='"1.6"' -DYEAR='"2018"' -DBUILD_DATE='"2020-10-29"' -DBUILD_COMMIT='"<unknown>"'   -c -o isolate.o isolate.c
gcc -std=gnu99 -Wall -Wextra -Wno-parentheses -Wno-unused-result -Wno-missing-field-initializers -Wstrict-prototypes -Wmissing-prototypes -D_GNU_SOURCE   -c -o util.o util.c
gcc -std=gnu99 -Wall -Wextra -Wno-parentheses -Wno-unused-result -Wno-missing-field-initializers -Wstrict-prototypes -Wmissing-prototypes -D_GNU_SOURCE   -c -o rules.o rules.c
gcc -std=gnu99 -Wall -Wextra -Wno-parentheses -Wno-unused-result -Wno-missing-field-initializers -Wstrict-prototypes -Wmissing-prototypes -D_GNU_SOURCE   -c -o cg.o cg.c
gcc -std=gnu99 -Wall -Wextra -Wno-parentheses -Wno-unused-result -Wno-missing-field-initializers -Wstrict-prototypes -Wmissing-prototypes -D_GNU_SOURCE -DCONFIG_FILE='"/usr/local/etc/isolate"'   -c -o config.o config.c
gcc  -o isolate isolate.o util.o rules.o cg.o config.o -lcap
===== Copying isolate to /usr/local/bin/
===== Copying isolate config to /usr/local/etc/
===== Copying configuration to /usr/local/etc/
===== Creating directories
===== Copying Polygon testlib
===== Adding yourself to the cmsuser group
Type Y if you want me to automatically add "cms" to the cmsuser group: y

###########################################################################
###                                                                     ###
###    Remember that you must now logout in order to make the change    ###
###    effective ("the change" is: being in the cmsuser group).         ###
###                                                                     ###
###########################################################################
```

### Bước 5: Cài đặt Python Requirement của CMS

```
sudo pip3 install -r requirements.txt
```

### Bước 6: Cài đặt CMS
```
sudo python3 setup.py install
```

Kết quả
```
...
Installing cmsRemoveContest script to /usr/local/bin
Installing cmsRemoveParticipation script to /usr/local/bin
Installing cmsRemoveSubmissions script to /usr/local/bin
Installing cmsRemoveTask script to /usr/local/bin
Installing cmsRemoveUser script to /usr/local/bin
Installing cmsRunTests script to /usr/local/bin
Installing cmsSpoolExporter script to /usr/local/bin

Installed /usr/local/lib/python3.6/dist-packages/cms-1.4rc1-py3.6.egg
Processing dependencies for cms==1.4rc1
Finished processing dependencies for cms==1.4rc1
cms@cmsedu:~/cms$ 
```

### Bước 7: Migrate Database
```
sudo cmsInitDB
```

Kết quả
```
cms@cmsedu:~/cms$ sudo cmsInitDB
2020-10-29 07:41:21,108 - INFO [<unknown>] Using configuration file /usr/local/etc/cms.conf.
```

### Bước 8: Bổ sung tài khoản Admin CMS
```
sudo pip3 install bcrypt==3.1.1
sudo cmsAddAdmin -p cloud3652020 admin
```

### Bước 9: Kiểm tra dịch vụ
```
cms@cmsedu:~/cms$ sudo cmsAddAdmin -p cloud3652020 admin
2020-10-29 07:49:06,151 - INFO [<unknown>] Using configuration file /usr/local/etc/cms.conf.
2020-10-29 07:49:06,799 - INFO [<unknown>] Creating the admin on the database.
2020-10-29 07:49:07,282 - INFO [<unknown>] Admin with complete access added. Login with username admin and password cloud3652020
cms@cmsedu:~/cms$ sudo cmsAdminWebServer
2020-10-29 07:49:40,969 - INFO [<unknown>] Using configuration file /usr/local/etc/cms.conf.
2020-10-29 07:49:42,067 - INFO [Admin,0] AdminWebServer 0 up and running!
2020-10-29 07:49:42,070 - INFO [Admin,0] Established connection with localhost:21100 (AdminWebServer,0) (local address: 127.0.0.1:34248).
2020-10-29 07:49:42,073 - INFO [Admin,0] Established connection with 127.0.0.1:34248 (local address: 127.0.0.1:21100).
```

Lưu ý:
- Nếu chạy được câu lệnh tức đã khởi tạo được được server CMS.
- Giữ màn hình CLI này để cấu hình Contest đầu tiến

## Phần 3: Cấu hình Contest CMS

Lưu ý:
- Để chạy được CMS Server đầu đủ tính năng, thì CMS phải tồn tại 1 Contest.
- Phải giữ màn hình chạy CLI `sudo cmsAdminWebServer` để cấu hình contest đầu tiên

### Bước 1: Truy cập giao diện

Lưu ý:
- Địa chỉ IP CMS trong docs là: 10.10.14.225
- Truy cập thông qua IP, port 8889

Truy cập địa chỉ: http://10.10.14.225:8889. Tài khoản admin / cloud3652020

![](/images/1-install-cms/pic1.png)

Kết quả

![](/images/1-install-cms/pic2.png)

### Bước 2: Tạo mới Contest

Chọn `Create new contest`
![](/images/1-install-cms/pic3.png)

Nhập tên `contest` và chọn `submit`
![](/images/1-install-cms/pic4.png)

Kết quả
![](/images/1-install-cms/pic5.png)

## Phần 4: Khởi tạo Server CMS

Lưu ý:
- CMS chạy server đầy đủ tiến trình bằng CLI
- Vì vậy ta sẽ phải chạy tiến trình CMS Server này trong byobu

### Bước 1: Trở lại CLI đang chạy `sudo cmsAdminWebServer`, chạy `CTRL + C`
```
^C2020-10-29 08:02:20,064 - WARNING [Admin,0] AdminWebServer,0 received request to shut down.
2020-10-29 08:02:20,069 - INFO [Admin,0] AdminWebServer 0 is shutting down
2020-10-29 08:02:20,070 - INFO [Admin,0] Terminated connection with localhost:21100 (AdminWebServer,0) (local address: 127.0.0.1:34248): Disconnection requested.
```

### Bước 2: Khởi động byobu
```
byobu
```

Kết quả
```
New release '20.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.



Welcome to the light, powerful, text window manager, Byobu.
You can toggle the launch of Byobu at login with:
  'byobu-disable' and 'byobu-enable'

For tips, tricks, and more information, see:
 * http://bit.ly/byobu-help
```

### Bước 3: Khởi động Server CMS
```
sudo cmsResourceService -a
```

Kết quả
```
cms@cmsedu:~/cms$ sudo cmsResourceService -a
[sudo] password for cms:
2020-10-29 08:04:20,714 - INFO [<unknown>] Using configuration file /usr/local/etc/cms.conf.
Contests available:
  1  -  ID: 1  -  Name: contest1demo  -  Description: contest1demo (default)
Insert the row number next to the contest you want to load (not the id): 1
2020-10-29 08:04:25,513 - INFO [Resource,0] ResourceService 0 up and running!
2020-10-29 08:04:25,545 - INFO [Resource,0] Restarting (AdminWebServer, 0)...
2020-10-29 08:04:25,555 - INFO [Resource,0] Restarting (Checker, 0)...
2020-10-29 08:04:25,569 - INFO [Resource,0] Restarting (ContestWebServer, 0)...
2020-10-29 08:04:25,587 - INFO [Resource,0] Restarting (EvaluationService, 0)...
2020-10-29 08:04:25,611 - INFO [Resource,0] Restarting (PrintingService, 0)...
2020-10-29 08:04:25,635 - INFO [Resource,0] Restarting (ProxyService, 0)...
2020-10-29 08:04:25,650 - INFO [Resource,0] Restarting (ScoringService, 0)...
2020-10-29 08:04:25,679 - INFO [Resource,0] Restarting (Worker, 0)...
2020-10-29 08:04:25,710 - INFO [Resource,0] Restarting (Worker, 1)...
2020-10-29 08:04:25,750 - INFO [Resource,0] Restarting (Worker, 2)...
2020-10-29 08:04:25,790 - INFO [Resource,0] Restarting (Worker, 3)...
2020-10-29 08:04:25,826 - INFO [Resource,0] Restarting (Worker, 4)...
2020-10-29 08:04:25,862 - INFO [Resource,0] Restarting (Worker, 5)...
2020-10-29 08:04:25,910 - INFO [Resource,0] Restarting (Worker, 6)...
2020-10-29 08:04:25,958 - INFO [Resource,0] Restarting (Worker, 7)...
2020-10-29 08:04:26,010 - INFO [Resource,0] Restarting (Worker, 8)...
2020-10-29 08:04:26,057 - INFO [Resource,0] Restarting (Worker, 9)...
2020-10-29 08:04:26,128 - INFO [Resource,0] Restarting (Worker, 10)...
2020-10-29 08:04:26,197 - INFO [Resource,0] Restarting (Worker, 11)...
2020-10-29 08:04:26,258 - INFO [Resource,0] Restarting (Worker, 12)...
2020-10-29 08:04:26,333 - INFO [Resource,0] Restarting (Worker, 13)...
2020-10-29 08:04:26,397 - INFO [Resource,0] Restarting (Worker, 14)...
2020-10-29 08:04:26,473 - INFO [Resource,0] Restarting (Worker, 15)...
2020-10-29 08:04:37,186 - INFO [Resource,0] Established connection with 127.0.0.1:42766 (local address: 127.0.0.1:28000).
2020-10-29 08:04:37,793 - INFO [Resource,0] Established connection with 127.0.0.1:42892 (local address: 127.0.0.1:28000).
```

Tới đây đã cấu hình Server CMS thành công, để tắt server CMS thực hiện `CTRL + C`

Trở lại giao diện

![](/images/1-install-cms/pic6.png)

