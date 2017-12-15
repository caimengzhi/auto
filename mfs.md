### 1. 存储MFS部署搭建
1. 平台
    + ubunt14.04 server
    + server 64
2. 安装软件列表
    + moosefs-cgi_3.0.86-1_amd64.deb
        - wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-cgi_3.0.86-1_amd64.deb
    + moosefs-cgiserv_3.0.86-1_amd64.deb
        - wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-cgiserv_3.0.86-1_amd64.deb
    + moosefs-chunkserver_3.0.86-1_amd64.deb
        - wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-chunkserver_3.0.86-1_amd64.deb
    + moosefs-cli_3.0.86-1_amd64.deb
        - wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-cli_3.0.86-1_amd64.deb
    + moosefs-client_3.0.86-1_amd64.deb
        - wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-client_3.0.86-1_amd64.deb
    + moosefs-master_3.0.86-1_amd64.deb
        - wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-master_3.0.86-1_amd64.deb
    + moosefs-metalogger_3.0.86-1_amd64.deb
        - wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-metalogger_3.0.86-1_amd64.deb
    + moosefs-netdump_3.0.86-1_amd64.deb
        - wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-netdump_3.0.86-1_amd64.deb

3. 具体关联

| IP地址 |主机名|角色|安装软件|
|-|-|-|-|
|192.168.1.37|loocha37|mfs cgi,mfs backup|moosefs-client
|192.168.1.9|loocha9|mfs server, mfs backup|moosefs-client,moosefs-metalogger,moosefs-cgiserv,moosefs-master
|192.168.1.41|realcloud41|chunk server 01, mfs backup|moosefs-chunkserver
|192.168.1.42|loocha42|chunk server 02|moosefs-chunkserver
|192.168.1.44|realcloud44|chunk server 03|moosefs-chunkserver,moosefs-metalogger

4. 准备工作
    + 修改hosts
    ```
    root@loocha37:/home/admin_loocha83# grep mfs* /etc/hosts
    <!--192.168.1.9    mfsmaster-->
    <!--192.168.1.37   mfsbackup-->
    <!--192.168.1.41   mfsdata01-->
    <!--192.168.1.42   mfsdata02-->
    <!--192.168.1.44   mfsdata03-->
    <!--192.168.1.100  mfs.master.org-->
    192.168.1.42    realcloud42 192.168.1.42
    192.168.1.9     loocha9 192.168.1.9
    192.168.1.41    realcloud41 192.168.1.41
    192.168.1.44    realcloud44 192.168.1.44
    192.168.1.100  mfs.master.org

    ```
    > 所有机器的hosts文件都要做修改
    
    + 绑定VIP
    在master上手动绑定vip
    ```
    root@loocha9:/home/admin_loocha09# ifconfig bridge01:0 192.168.1.100
    root@loocha9:/home/admin_loocha09# ifconfig bridge01:0
    bridge01:0 Link encap:Ethernet  HWaddr 00:25:90:4a:0b:c8  
          inet addr:`192.168.1.100`  Bcast:192.168.1.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MASTER MULTICAST  MTU:1500  Metric:1
    ```
    + 测试
        ```
        root@loocha9:~# ping -c1 loocha9
        PING loocha9 (192.168.1.9) 56(84) bytes of data.
        64 bytes from loocha9 (192.168.1.9): icmp_seq=1 ttl=64 time=0.035 ms
        
        --- loocha9 ping statistics ---
        1 packets transmitted, 1 received, 0% packet loss, time 0ms
        rtt min/avg/max/mdev = 0.035/0.035/0.035/0.000 ms
        
        root@loocha9:~# ping -c1 realcloud41
        PING realcloud41 (192.168.1.41) 56(84) bytes of data.
        64 bytes from realcloud41 (192.168.1.41): icmp_seq=1 ttl=64 time=0.254 ms
        
        --- realcloud41 ping statistics ---
        1 packets transmitted, 1 received, 0% packet loss, time 0ms
        rtt min/avg/max/mdev = 0.254/0.254/0.254/0.000 ms
        
        root@loocha9:~# ping -c1 realcloud42
        PING realcloud42 (192.168.1.42) 56(84) bytes of data.
        64 bytes from realcloud42 (192.168.1.42): icmp_seq=1 ttl=64 time=0.219 ms
        
        --- realcloud42 ping statistics ---
        1 packets transmitted, 1 received, 0% packet loss, time 0ms
        rtt min/avg/max/mdev = 0.219/0.219/0.219/0.000 ms
        
        root@loocha9:~# ping -c1 realcloud44
        PING realcloud44 (192.168.1.44) 56(84) bytes of data.
        64 bytes from realcloud44 (192.168.1.44): icmp_seq=1 ttl=64 time=0.193 ms
        
        --- realcloud44 ping statistics ---
        1 packets transmitted, 1 received, 0% packet loss, time 0ms
        rtt min/avg/max/mdev = 0.193/0.193/0.193/0.000 ms
        
        root@loocha9:~# ping -c1 mfs.master.org
        PING mfs.master.org (192.168.1.100) 56(84) bytes of data.
        64 bytes from mfs.master.org (192.168.1.100): icmp_seq=1 ttl=64 time=0.035 ms
        
        --- mfs.master.org ping statistics ---
        1 packets transmitted, 1 received, 0% packet loss, time 0ms
        rtt min/avg/max/mdev = 0.035/0.035/0.035/0.000 ms

        ```
> 注意: 每个机器上都要测试一遍
### 2. 角色部署
1. mfs master部署
 - 详细步骤
 ```
 root@loocha9:~/mfs# wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-master_3.0.86-1_amd64.deb
--2017-11-24 10:47:16--  http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-master_3.0.86-1_amd64.deb
Connecting to 180.96.7.219:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 379238 (370K) [application/octet-stream]
Saving to: ‘moosefs-master_3.0.86-1_amd64.deb’

100%[=============================================================================================================================>] 379,238     --.-K/s   in 0.006s  

2017-11-24 10:47:16 (58.5 MB/s) - ‘moosefs-master_3.0.86-1_amd64.deb’ saved [379238/379238]

root@loocha9:~/mfs# ls
moosefs-master_3.0.86-1_amd64.deb
root@loocha9:~/mfs# dpkg -i moosefs-master_3.0.86-1_amd64.deb 
Selecting previously unselected package moosefs-master.
(Reading database ... 66886 files and directories currently installed.)
Preparing to unpack moosefs-master_3.0.86-1_amd64.deb ...
Unpacking moosefs-master (3.0.86-1) ...
Setting up moosefs-master (3.0.86-1) ...
 * moosefs-master not enabled in "/etc/default/moosefs-master", exiting...
Processing triggers for ureadahead (0.100.0-16) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
root@loocha9:~/mfs# vim /etc/mfs/mfs
mfsexports.cfg.sample   mfsmaster.cfg.sample    mfstopology.cfg.sample  
root@loocha9:~/mfs# cd /etc/mfs/
root@loocha9:/etc/mfs# ls
mfsexports.cfg.sample  mfsmaster.cfg.sample  mfstopology.cfg.sample
root@loocha9:/etc/mfs# cp mfsmaster.cfg.sample  mfsmaster.cfg
root@loocha9:/etc/mfs# cp mfsexports.cfg.sample mfsexports.cfg
root@loocha9:~# cp /var/lib/mfs/metadata.mfs /opt/mfsmaster/
root@loocha9:~# mfsmaster start
open files limit has been set to: 16384
working directory: /opt/mfsmaster
lockfile created and locked
initializing mfsmaster modules ...
exports file has been loaded
mfstopology configuration file (/etc/mfstopology.cfg) not found - using defaults
loading metadata ...
metadata file has been loaded
no charts data file - initializing empty charts
master <-> metaloggers module: listen on *:9419
master <-> chunkservers module: listen on *:9420
main master server module: listen on *:9421
mfsmaster daemon initialized properly


 ```
 - 配置文件解释
 ```
 root@loocha9:/etc/mfs# egrep -v '#|^$' mfsexports.cfg
*			/	rw,alldirs,admin,maproot=0:0
*			.	rw
 ```
 2. mfs cgiserver 部署
 - 详细部署
 ```
 root@loocha9:~/mfs# wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-cgiserv_3.0.86-1_amd64.deb
--2017-11-24 12:59:53--  http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-cgiserv_3.0.86-1_amd64.deb
Connecting to 180.96.7.219:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 30106 (29K) [application/octet-stream]
Saving to: ‘moosefs-cgiserv_3.0.86-1_amd64.deb’

100%[=============================================================================================================================>] 30,106      --.-K/s   in 0s      

2017-11-24 12:59:53 (59.3 MB/s) - ‘moosefs-cgiserv_3.0.86-1_amd64.deb’ saved [30106/30106]
root@loocha9:~/mfs# wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-cgi_3.0.86-1_amd64.deb
--2017-11-24 13:00:25--  http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-cgi_3.0.86-1_amd64.deb
Connecting to 180.96.7.219:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 81914 (80K) [application/x-debian-package]
Saving to: ‘moosefs-cgi_3.0.86-1_amd64.deb’

100%[=============================================================================================================================>] 81,914      --.-K/s   in 0.001s  

2017-11-24 13:00:25 (52.2 MB/s) - ‘moosefs-cgi_3.0.86-1_amd64.deb’ saved [81914/81914]

root@loocha9:~/mfs# ls
moosefs-cgi_3.0.86-1_amd64.deb  moosefs-cgiserv_3.0.86-1_amd64.deb  moosefs-master_3.0.86-1_amd64.deb
root@loocha9:~/mfs# dpkg -i moosefs-cgi
moosefs-cgi_3.0.86-1_amd64.deb      moosefs-cgiserv_3.0.86-1_amd64.deb  
root@loocha9:~/mfs# dpkg -i moosefs-cgi
moosefs-cgi_3.0.86-1_amd64.deb      moosefs-cgiserv_3.0.86-1_amd64.deb  
root@loocha9:~/mfs# dpkg -i moosefs-cgi_3.0.86-1_amd64.deb 
Selecting previously unselected package moosefs-cgi.
(Reading database ... 66917 files and directories currently installed.)
Preparing to unpack moosefs-cgi_3.0.86-1_amd64.deb ...
Unpacking moosefs-cgi (3.0.86-1) ...
Setting up moosefs-cgi (3.0.86-1) ...
root@loocha9:~/mfs# dpkg -i moosefs-cgiserv_3.0.86-1_amd64.deb 
(Reading database ... 66929 files and directories currently installed.)
Preparing to unpack moosefs-cgiserv_3.0.86-1_amd64.deb ...
Unpacking moosefs-cgiserv (3.0.86-1) over (3.0.86-1) ...
Setting up moosefs-cgiserv (3.0.86-1) ...
 * moosefs-cgiserv not enabled in "/etc/default/moosefs-cgiserv", exiting...
Processing triggers for ureadahead (0.100.0-16) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
root@loocha9:~/mfs# ls
moosefs-cgi_3.0.86-1_amd64.deb  moosefs-cgiserv_3.0.86-1_amd64.deb  moosefs-master_3.0.86-1_amd64.deb
root@loocha9:~/mfs# vim /etc/mfs/mfs
mfsexports.cfg          mfsexports.cfg.sample   mfsmaster.cfg           mfsmaster.cfg.sample    mfstopology.cfg.sample  
root@loocha9:~/mfs# cd /etc/mfs/
root@loocha9:/etc/mfs# ls
mfsexports.cfg  mfsexports.cfg.sample  mfsmaster.cfg  mfsmaster.cfg.sample  mfstopology.cfg.sample
root@loocha9:/etc/mfs# ll
total 44
drwxr-xr-x  2 root root 4096 Nov 24 11:23 ./
drwxr-xr-x 98 root root 4096 Nov 24 10:47 ../
-rw-r--r--  1 root root 4057 Nov 24 10:50 mfsexports.cfg
-rw-r--r--  1 root root 4057 Nov 30  2016 mfsexports.cfg.sample
-rw-r--r--  1 root root 8620 Nov 24 11:23 mfsmaster.cfg
-rw-r--r--  1 root root 8521 Nov 30  2016 mfsmaster.cfg.sample
-rw-r--r--  1 root root 1052 Nov 30  2016 mfstopology.cfg.sample
root@loocha9:/etc/mfs# mfs
mfscgiserv      mfsmaster       mfsmetadump     mfsmetarestore  mfsstatsdump    
root@loocha9:/etc/mfs# mfscgiserv start
lockfile created and locked
starting simple cgi server (host: any , port: 9425 , rootpath: /usr/share/mfscgi)
root@loocha9:/etc/mfs# netstat -lnp|grep 9425
tcp        0      0 0.0.0.0:9425            0.0.0.0:*               LISTEN      7147/python     
 ```
 - 配置文件解释 
 - web login
```
浏览器中输入`http://192.168.1.100:9425`即可。
此时在页面输入192.168.1.100（192.168.1.9）
```

 3. mfs metalogger 部署
 - 详细部署
 ```
 root@loocha9:~/mfs# wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-metalogger_3.0.86-1_amd64.deb
--2017-11-24 13:16:19--  http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-metalogger_3.0.86-1_amd64.deb
Connecting to 180.96.7.219:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 58012 (57K) [application/octet-stream]
Saving to: ‘moosefs-metalogger_3.0.86-1_amd64.deb’

100%[=============================================================================================================================>] 58,012      --.-K/s   in 0.003s  

2017-11-24 13:16:19 (21.7 MB/s) - ‘moosefs-metalogger_3.0.86-1_amd64.deb’ saved [58012/58012]

root@loocha9:~/mfs# dpkg -i moosefs-metalogger_3.0.86-1_amd64.deb 
Selecting previously unselected package moosefs-metalogger.
(Reading database ... 66929 files and directories currently installed.)
Preparing to unpack moosefs-metalogger_3.0.86-1_amd64.deb ...
Unpacking moosefs-metalogger (3.0.86-1) ...
Setting up moosefs-metalogger (3.0.86-1) ...
 * moosefs-metalogger not enabled in "/etc/default/moosefs-metalogger", exiting...
Processing triggers for ureadahead (0.100.0-16) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...

root@loocha9:~/mfs# cd /etc/mfs/
root@loocha9:/etc/mfs# ls
mfsexports.cfg  mfsexports.cfg.sample  mfsmaster.cfg  mfsmaster.cfg.sample  mfsmetalogger.cfg.sample  mfstopology.cfg.sample
root@loocha9:/etc/mfs# egrep -v '#|^$' mfsmetalogger.cfg.sample 
root@loocha9:/etc/mfs# cp mfsmetalogger.cfg.sample mfsmetalogger.cfg
root@loocha9:/etc/mfs# vim mfsmetalogger.cfg
root@loocha9:/etc/mfs# egrep -v '#|^$' mfsmetalogger.cfg
DATA_PATH = /opt/metalogger
MASTER_HOST = mfs.master.org
root@loocha9:/etc/mfs# mkdir /opt/metalogger
root@loocha9:/etc/mfs# chown -R mfs.mfs /opt/metalogger/
root@loocha9:/etc/mfs# mfsmetalogger start
open files limit has been set to: 4096
working directory: /opt/metalogger
lockfile created and locked
initializing mfsmetalogger modules ...
mfsmetalogger daemon initialized properly
root@loocha9:/etc/mfs# ps axf|grep mfsmetalogger
15748 pts/0    S+     0:00                          \_ grep --color=auto mfsmetalogger
14995 ?        S<     0:00 mfsmetalogger start
 ```
 > 注意： mfsmetalogger 没有启动端口只启动了监听进程
 
 - 配置文件解释 
 ```
 root@loocha9:/etc/mfs# egrep -v '#|^$' mfsmetalogger.cfg
 DATA_PATH = /opt/metalogger   # 手动配置metlogger的存储路径
 MASTER_HOST = mfs.master.org  # 配置master的ip或者域名（之前hosts配置好的）
 ```
 4. mfs chunkserver 部署
 - 详细部署
```
root@loocha9:~/mfs# wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-chunkserver_3.0.86-1_amd64.deb
--2017-11-24 13:37:37--  http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-chunkserver_3.0.86-1_amd64.deb
Connecting to 180.96.7.219:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 186346 (182K) [application/octet-stream]
Saving to: ‘moosefs-chunkserver_3.0.86-1_amd64.deb’

100%[=============================================================================================================================>] 186,346     --.-K/s   in 0.003s  

2017-11-24 13:37:37 (53.9 MB/s) - ‘moosefs-chunkserver_3.0.86-1_amd64.deb’ saved [186346/186346]

root@loocha9:~/mfs# dpkg -i moosefs-chunkserver_3.0.86-1_amd64.deb 
Selecting previously unselected package moosefs-chunkserver.
(Reading database ... 67635 files and directories currently installed.)
Preparing to unpack moosefs-chunkserver_3.0.86-1_amd64.deb ...
Unpacking moosefs-chunkserver (3.0.86-1) ...
Setting up moosefs-chunkserver (3.0.86-1) ...
 * moosefs-chunkserver not enabled in "/etc/default/moosefs-chunkserver", exiting...
Processing triggers for ureadahead (0.100.0-16) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
root@loocha9:/etc/mfs# cp mfschunkserver.cfg.sample mfschunkserver.cfg
root@loocha9:/etc/mfs# egrep -v '#|^$' mfschunkserver.cfg
root@loocha9:/etc/mfs# vim mfschunkserver.cfg
root@loocha9:/etc/mfs# egrep -v '#|^$' mfschunkserver.cfg
DATA_PATH = /opt/lchunk
MASTER_HOST = mfs.master.org 
root@loocha9:/etc/mfs# mkdir /opt/lchunk
root@loocha9:/etc/mfs# chown -R mfs.mfs /opt/lchunk
root@loocha9:/etc/mfs# egrep -v '#|^$' mfshdd.cfg.sample 
root@loocha9:/etc/mfs# cp mfshdd.cfg.sample mfshdd.cfg
root@loocha9:/etc/mfs# vim mfshdd.cfg
root@loocha9:/etc/mfs# egrep -v '#|^$' mfshdd.cfg
/opt/ldata
root@loocha9:/etc/mfs# mkdir /opt/ldata && chown -R mfs.mfs /opt/ldata

root@loocha9:/etc/mfs# mfschunkserver start
open files limit has been set to: 16384
working directory: /opt/lchunk
lockfile created and locked
setting glibc malloc arena max to 4
setting glibc malloc arena test to 4
initializing mfschunkserver modules ...
hdd space manager: path to scan: /opt/ldata/
hdd space manager: start background hdd scanning (searching for available chunks)
main server module: listen on *:9422
no charts data file - initializing empty charts
mfschunkserver daemon initialized properly
root@loocha9:/etc/mfs# netstat -lnp|grep 9422
tcp        0      0 0.0.0.0:9422            0.0.0.0:*               LISTEN      23514/mfschunkserve

```
- 配置文件解释
```
root@loocha9:/etc/mfs# egrep -v '#|^$' mfschunkserver.cfg
DATA_PATH = /opt/lchunk
MASTER_HOST = mfs.master.org  # master ip或者域名

root@loocha9:/etc/mfs# egrep -v '#|^$' mfshdd.cfg
/opt/ldata   # 配置挂载点,可以配置多个

```
4. 检查master主机上所有mfs信息
```
root@loocha9:/etc/mfs# netstat -lnp|grep 94
tcp        0      0 0.0.0.0:9425            0.0.0.0:*               LISTEN      7147/python     
tcp        0      0 0.0.0.0:9419            0.0.0.0:*               LISTEN      14653/mfsmaster 
tcp        0      0 0.0.0.0:9420            0.0.0.0:*               LISTEN      14653/mfsmaster 
tcp        0      0 0.0.0.0:9421            0.0.0.0:*               LISTEN      14653/mfsmaster 
tcp        0      0 0.0.0.0:9422            0.0.0.0:*               LISTEN      23514/mfschunkserve
root@loocha9:/etc/mfs# ps aux|grep -v grep|grep mfs
root      7147  0.0  0.0  41524  8960 ?        S    13:01   0:00 /usr/bin/python /usr/sbin/mfscgiserv start
root     10283  0.0  0.0      0     0 ?        Z    13:11   0:00 [mfs.cgi] <defunct>
mfs      14653  0.1  0.3 265592 239304 ?       S<   11:43   0:13 mfsmaster start
mfs      14995  0.1  0.0  17228  2380 ?        S<   13:25   0:02 mfsmetalogger start
mfs      23514  0.3  0.2 302672 138032 ?       S<l  13:47   0:00 mfschunkserver start

```
以上是mfs master端部署情况
--------------------------------------------------

# mfs chunserver部署
### 1. 快速部署
```
cd ~
mkdir mfs
cd mfs/
wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-chunkserver_3.0.86-1_amd64.deb
dpkg -i moosefs-chunkserver_3.0.86-1_amd64.deb
cd /etc/mfs/
cp mfschunkserver.cfg.sample mfschunkserver.cfg
sed -i 's@# DATA_PATH = /var/lib/mfs@DATA_PATH = /opt/lchunk@g' mfschunkserver.cfg
sed -i 's@# MASTER_HOST = mfsmaster@MASTER_HOST = mfs.master.org@g' mfschunkserver.cfg
mkdir /opt/lchunk
chown -R mfs.mfs /opt/lchunk
cp mfshdd.cfg.sample mfshdd.cfg
echo '/opt/ldata' >> mfshdd.cfg
mkdir /opt/ldata && chown -R mfs.mfs /opt/ldata
mfschunkserver start
netstat -lnp|grep 9422
```
##
### 2. 详细部署
```
root@realcloud41:~# mkdir mfs
root@realcloud41:~# cd mfs/
root@realcloud41:~/mfs# wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-chunkserver_3.0.86-1_amd64.deb
--2017-11-24 14:22:06--  http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-chunkserver_3.0.86-1_amd64.deb
Connecting to 180.96.7.219:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 186346 (182K) [application/x-debian-package]
Saving to: ‘moosefs-chunkserver_3.0.86-1_amd64.deb’

100%[=============================================================================================================================>] 186,346     --.-K/s   in 0.002s  

2017-11-24 14:22:06 (110 MB/s) - ‘moosefs-chunkserver_3.0.86-1_amd64.deb’ saved [186346/186346]

root@realcloud41:~/mfs# dpkg -i moosefs-chunkserver_3.0.86-1_amd64.deb 
Selecting previously unselected package moosefs-chunkserver.
(Reading database ... 61222 files and directories currently installed.)
Preparing to unpack moosefs-chunkserver_3.0.86-1_amd64.deb ...
Unpacking moosefs-chunkserver (3.0.86-1) ...
Setting up moosefs-chunkserver (3.0.86-1) ...
 * moosefs-chunkserver not enabled in "/etc/default/moosefs-chunkserver", exiting...
Processing triggers for ureadahead (0.100.0-16) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
root@realcloud41:~/mfs# cd /etc/mfs/
root@realcloud41:/etc/mfs# ls
mfschunkserver.cfg.sample  mfshdd.cfg.sample
root@realcloud41:/etc/mfs# cp mfschunkserver.cfg.sample mfschunkserver.cfg
root@realcloud41:/etc/mfs# egrep -v '#|^$' mfschunkserver.cfg
root@realcloud41:/etc/mfs# vim mfschunkserver.cfg
root@realcloud41:/etc/mfs# egrep -v '#|^$' mfschunkserver.cfg
DATA_PATH = /opt/lchunk
MASTER_HOST = mfs.master.org
root@realcloud41:/etc/mfs# mkdir /opt/lchunk
root@realcloud41:/etc/mfs# chown -R mfs.mfs /opt/lchunk
root@realcloud41:/etc/mfs# egrep -v '#|^$' mfshdd.cfg.sample
root@realcloud41:/etc/mfs# cp mfshdd.cfg.sample mfshdd.cfg
root@realcloud41:/etc/mfs# vim mfshdd.cfg
root@realcloud41:/etc/mfs# mkdir /opt/ldata && chown -R mfs.mfs /opt/ldata
root@realcloud41:/etc/mfs# mfschunkserver start
open files limit has been set to: 16384
working directory: /opt/lchunk
lockfile created and locked
setting glibc malloc arena max to 4
setting glibc malloc arena test to 4
initializing mfschunkserver modules ...
hdd space manager: path to scan: /opt/ldata/
hdd space manager: start background hdd scanning (searching for available chunks)
main server module: listen on *:9422
no charts data file - initializing empty charts
mfschunkserver daemon initialized properly
root@realcloud41:/etc/mfs# netstat -lnp|grep 9422
tcp        0      0 0.0.0.0:9422            0.0.0.0:*               LISTEN      3354/mfschunkserver
```


### mfs client 

### mfs client 快速部署
```
wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-client_3.0.86-1_amd64.deb
dpkg -i moosefs-client_3.0.86-1_amd64.deb 
mfsmount -H  mfs.master.org /opt/mfs/
```

详细操作步骤
```
root@loocha37:~# mkdir mfs
root@loocha37:~# cd mfs/
root@loocha37:~/mfs# wget http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-client_3.0.86-1_amd64.deb
--2017-11-27 09:29:25--  http://180.96.7.219:8080/software/cmz/moosefs/package/moosefs-client_3.0.86-1_amd64.deb
Connecting to 180.96.7.219:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 255862 (250K) [application/octet-stream]
Saving to: ‘moosefs-client_3.0.86-1_amd64.deb’

100%[===========================================================================================>] 255,862     --.-K/s   in 0.003s  

2017-11-27 09:29:25 (72.9 MB/s) - ‘moosefs-client_3.0.86-1_amd64.deb’ saved [255862/255862]
root@loocha37:~/mfs# dpkg -i moosefs-client_3.0.86-1_amd64.deb
root@loocha37:/home/admin_loocha83# df -h
Filesystem           Size  Used Avail Use% Mounted on
udev                  16G  4.0K   16G   1% /dev
tmpfs                3.2G  812K  3.2G   1% /run
/dev/sda1            884G   41G  798G   5% /
none                 4.0K     0  4.0K   0% /sys/fs/cgroup
none                 5.0M     0  5.0M   0% /run/lock
none                  16G  224K   16G   1% /run/shm
none                 100M     0  100M   0% /run/user
none                 884G   41G  798G   5% /var/lib/docker/aufs/mnt/c6fa896175ff2a64313f214688788ce03b950c8568070ac13317a67b82583666
shm                   64M     0   64M   0% /var/lib/docker/containers/72792fe6a203c760b84579f3b775e34f25aa36e7766dc7097931f0d21a733060/shm
none                 884G   41G  798G   5% /var/lib/docker/aufs/mnt/a8023b4420098e8fcd4ed1404b4a0b32df027a3c4c01b9eb5eda09437012c59e
shm                   64M     0   64M   0% /var/lib/docker/containers/beebddefe761057fceed9bc40e918b7da771951e1b1c394c3ee7e170cb02a528/shm

root@loocha37:/opt# ls /opt/mfs/
root@loocha37:/opt# mfsmount -H  mfs.master.org /opt/mfs/
mfsmaster accepted connection with parameters: read-write,restricted_ip,admin ; root mapped to root:root
root@loocha37:/opt# df -h
Filesystem           Size  Used Avail Use% Mounted on
udev                  16G  4.0K   16G   1% /dev
tmpfs                3.2G  812K  3.2G   1% /run
/dev/sda1            884G   41G  798G   5% /
none                 4.0K     0  4.0K   0% /sys/fs/cgroup
none                 5.0M     0  5.0M   0% /run/lock
none                  16G  224K   16G   1% /run/shm
none                 100M     0  100M   0% /run/user
none                 884G   41G  798G   5% /var/lib/docker/aufs/mnt/c6fa896175ff2a64313f214688788ce03b950c8568070ac13317a67b82583666
shm                   64M     0   64M   0% /var/lib/docker/containers/72792fe6a203c760b84579f3b775e34f25aa36e7766dc7097931f0d21a733060/shm
none                 884G   41G  798G   5% /var/lib/docker/aufs/mnt/a8023b4420098e8fcd4ed1404b4a0b32df027a3c4c01b9eb5eda09437012c59e
shm                   64M     0   64M   0% /var/lib/docker/containers/beebddefe761057fceed9bc40e918b7da771951e1b1c394c3ee7e170cb02a528/shm
mfs.master.org:9421   55T  2.4T   53T   5% /opt/mfs
root@loocha37:/opt# ls /opt/mfs/
data  set_readme.txt
```
### mfs zabbix监控
1. zabbix key模板
    + 根据mfs系统中的role来进行选择
```
1. 配置iterm和key
root@loocha9:~# cd /etc/zabbix/zabbix_agentd.conf.d
root@loocha9:/etc/zabbix/zabbix_agentd.conf.d# ls
mfs.conf
root@loocha9:/etc/zabbix/zabbix_agentd.conf.d# cat mfs.conf 
#mfs monitor
UserParameter=mfs.mfsmaster,ps -ef |grep -v grep | grep -cw mfsmaster
UserParameter=mfs.mfschunkserver,ps -ef |grep -v grep | grep -cw mfschunkserver
UserParameter=mfs.mfsmetalogger,ps -ef |grep -v grep | grep -cw mfsmetalogger
UserParameter=mfs.mfscgiserv,ps -ef |grep -v grep | grep -cw mfscgiserv
UserParameter=mfs.mfsVip,ip addr|grep 192.168.1.100|wc -l
```
> 注意: 根据系统的的角色选择对应的key

```
1. 192.168.1.41 
root@realcloud41:~# cd /etc/zabbix/zabbix_agentd.conf.d/
root@realcloud41:/etc/zabbix/zabbix_agentd.conf.d# cat mfs.conf 
#mfs monitor
UserParameter=mfs.mfschunkserver,ps -ef |grep -v grep | grep -cw mfschunkserver

2. 192.168.1.42
root@realcloud42:/home/realcloud# cd /etc/zabbix/zabbix_agentd.conf.d/
root@realcloud42:/etc/zabbix/zabbix_agentd.conf.d# cat mfs.conf 
#mfs monitor
UserParameter=mfs.mfschunkserver,ps -ef |grep -v grep | grep -cw mfschunkserver

3. 192.168.1.44
root@realcloud44:~# cd /etc/zabbix/zabbix_agentd.conf.d/
root@realcloud44:/etc/zabbix/zabbix_agentd.conf.d# cat mfs.conf 
#mfs monitor
UserParameter=mfs.mfschunkserver,ps -ef |grep -v grep | grep -cw mfschunkserver
UserParameter=mfs.mfsmetalogger,ps -ef |grep -v grep | grep -cw mfsmetalogger


```

2. 重新zabbix-agent
    + 重启zabbix agent
    
```
root@loocha9:~# /etc/init.d/zabbix-agent restart
 * zabbix_agentd stopping...                     [ OK ] 
 * zabbix_agentd starting...                     [ OK ]
```

3.  zabbix server 添加iterm ,Triggers和screens
    + 添加iterm
    以下我以mfs master为例，其他类似
    Configuration --> Hosts(找到mfs机器 192.168.1.9)-->Iterms -->Create item 
    注意点：
    1. Name       ：可以根据role写,如mfs master
    2. key        ：位置手动填写之前zabbix-agent中配置的key 如mfs.master
    3. 数据类型   ：选Numeric (unsigned)
    配置完以后选择update,提交。
---------------------------------------------------------------------------------
    + 添加Triggers
    Configuration --> Hosts(找到mfs机器 192.168.1.9)-->Triggers -->Create trigger 
    注意点：
    1. Name       ： 监控的process进程 ，如mfs process num
    2. Serverity  ： Warning
    3. Expression :  
```
{192.168.1.9:mfs.mfscgiserv.last()}=0 or
{192.168.1.9:mfs.mfschunkserver.last()}=0 or 
{192.168.1.9:mfs.mfsmaster.last()}=0 or 
{192.168.1.9:mfs.mfsmetalogger.last()}=0 or 
{192.168.1.9:mfs.mfsVip.last()}=0
```
> 注意判断的标准是mfs master上的所有关于mfs的进程只要process num等于0就触发，然后告警
---------------------------------------------------------------------------------

    + 添加screen
    Monitoring --> Screens --> 新建screen（mfs）-->添加所有机器的mfs相关消息到该screen中即可
