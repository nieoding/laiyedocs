以root账号登陆 安装基础服务

``` bash
yum install -y epel-release
yum install –y wget lrzsz nginx redis
yum install –y mysql/mariadb          # 安装数据库客户端
(yum –y install mysql-server)         # 使用本地数据库另外安装mysql服务端
```

关闭防火墙

``` bash
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
systemctl stop firewalld
```

# redis配置

``` conf
# /etc/redis.conf
daemonize yes
save ""
bind 127.0.0.1
```

开启自启动

``` bash
systemctl start redis
systemctl enable redis
```

# nginx

等后续各服务配置好再配置

# 创建works用户

``` bash
useradd -m works
cd /home/works && su works
```

之后的应用服务必须放在/home/works下 否则需要从代码中修改路径

# nvm

``` bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.4/install.sh | bash
```

# 拷贝线上服务的所有文件到当前部署的主机

(/home/works目录)

可以清除旧的log

``` bash
find . -regex ".*\.log\.[0-9]*" -print0 | xargs -0 -I {} rm '{}'
```

# nodejs & yarn

``` bash
nvm install v6.9.5
node -v
```

多个node环境 需要手动切换v6.9.5

``` bash
nvm alias default v6.9.5
nvm use v6.9.5
npm install -g cnpm yarn
```

# 数据库迁移

``` bash
# 拷贝原始数据到当前部署的RDS
mysqldump ... > jingli_knowledge_20190213.sql
mysql -h lizcloud4c.chnh6yhldzwc.rds.cn-north-1.amazonaws.com.cn -u root -p
create database jingli_knowledge;
mysql -h lizcloud4c.chnh6yhldzwc.rds.cn-north-1.amazonaws.com.cn -u root jingli_knowledge -p < jingli_knowledge_20190213.sql
```