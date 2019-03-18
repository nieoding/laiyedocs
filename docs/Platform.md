知识库管理平台运行于aws ecs服务器,同时使用了阿里云的云数据库和开放搜索 (OPENSEARCH)

aws服务器所在云服务器公网地址为52.80.194.120,操作系统为CentOS7。

数据库为阿里云RDS mysql版本v5.6

```
appid:    rm-uf62bx6y83t6yws19
内网地址:  rm-uf62bx6y83t6yws19.mysql.rds.aliyuncs.com
外网地址:  rm-uf62bx6y83t6yws19mo.mysql.rds.aliyuncs.com
用户:     gemii_root
密码:     gemii!@123
```

知识库web访问地址为 http://pubcontent.gemii.cc/，web服务使用node js v6.9.5搭建，运行于端口8332、8333，使用nginx做80端口反代，使用pm2 v2.5.0控制和监控web服务，使用服务器本地redis v3.2.3作为临时key-value存储工具，使用端口6379、db3作为使用实例。