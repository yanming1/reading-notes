# 环境说明

#### 1. 远程存储服务器，并已设置共享目录 

#### 2. centos7服务器并安装有oracle数据库

# 操作步骤

 ### 说明:所有操作都是在centos服务器上进行

#### 1. 安装smaba-client(smaba-client作用是为了挂载远程存储的共享目录)

```shell
yum -y install samba-client
```

#### 2. 创建挂载目录（目录名字自行定义就行）

```shell
mkdir /ostore
```

#### 3.  挂载远程存储目录

```shell
mount -t cifs -o username="${用户名}",password="${密码}" //${存储ip}/${共享目录} /ostore
```

#### 4.  编写数据库备份脚本

本例脚本暂命名/ostore/dbak.sh

```shell
#!/bin/bash
#以下环境变量建议拷贝oracle用户的环境变量
export ORACLE_BASE=/data/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
export ORACLE_HOME_LISTNER=$ORACLE_HOME
export ORACLE_SID=orcl
export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH
export TNS_ADMIN=$ORACLE_HOME/network/admin
export NLS_LANG=AMERICAN_AMERICA.UTF8


date=`date +%Y_%m_%d`   #获取系统当前日期时间
days=7  #设置删除7天之前的备份文件
orsid=`${ip}:${端口}/${sid}`  #Oracle数据库服务器 格式为  IP:端口/SID
orowner=${oracle用户}  #备份此用户下面的数据
bakuser=${执行备份的用户}  #用此用户来执行备份，必须要有备份操作的权限
bakpass="""${执行备份的用户密码}"""  #执行备注的用户密码
bakdir=/ostroe  #备份文件路径，需要提前创建好，此处直接备份到了共享存储目录
bakdata=$orowner"_"$date.dmp #备份数据库名称
baklog=$orowner"_"$date.log #备份执行时候生成的日志文件名称
ordatabak=$orowner"_"$date.tar.gz #最后保存的Oracle数据库备份文件
cd $bakdir #进入备份目录
exp $bakuser/$bakpass@$orsid grants=y owner=$orowner file=$bakdir/$bakdata log=$bakdir/$baklog #执行备份
tar -zcvf $ordatabak $bakdata $baklog  #压缩备份文件和日志文件
find $bakdir -type f -name "*.log" -exec rm {} \; #删除备份文件
find $bakdir -type f -name "*.dmp" -exec rm {} \; #删除日志文件
find $bakdir -type f -name "*.tar.gz" -mtime +$days -exec rm -rf {} \;  #删除7天前的备份（注意：{} \中间有空格）
```

#### 5.  为执行脚本添加执行权限

```shell
chmod +x /ostore/dbbak.sh
```

#### 6.  添加cron任务配置脚本定期执行

##### （1）执行以下命令添加定时任务（此例代表每天下午19点执行）：

##### 0 19 * * * /ostore/dbbak.sh

```shell
crontab -e
```

##### （2）重启定时任务服务使之生效

```shell
systemctl restart crond
```

#### 7.  查看定时任务日志

```shell
cat /var/log/cron
```

