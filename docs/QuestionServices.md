智能问答系统采用检索+排序架构实现问题和候选知识点的语义匹配，其架构示意图及各个模块功能描述如下。
<图>

各服务必须使用名为works的用户

# 索引服务

`/home/works/tools/elasticsearch-5.4.0`

该模块利用经审核的知识库数据建立倒排索引，用于召回问题的候选回复集合。
该模块只负责检索召回，不涉及业务逻辑，选用开源搜索引擎ElasticSearch搭建，安装jieba插件作为项目使用的中文分词器。通过supervisor进行管理。

- JDK & supervisor

``` bash
sudo yum install java-1.8.0-openjdk-devel
java -version
sudo pip install supervisor
```

- 启动es

``` bash
cd ~/tools/elasticsearch-5.4.0
supervisord -c config/sup.kb_es_node.conf
supervisorctl -c config/sup.kb_es_node.conf        # 查看状态
```

# 检索/排序模块

`/home/works/online-services/qa-search`

项目的检索+排序过程都封装在这个模块中。
检索:该模块的输入是问题文本，它调用检索服务后，根据业务需求做必要的处理，返回候选回复集合。
具体功能包括：对输入的问题做分词、词性过滤、同义词变换并构造ES检索表达式调用ES检索，同时对召回结果进行去重等操作。
该模块为问答系统内部服务，是Python开发的web service，供内部访问,不对外提供服务接口。
排序:该模块负责对由检索模块初步召回的候选回复集合做重排序，排序的主要依据是问题与候选知识点的语义相关性。

该模块由virtualenv提供隔离运行环境，且通过supervisor进行守护。所以，需要先激活virtualenv环境并安装好该模块依赖的第三方packages才能正常启动。

- 配置

``` python
# bin/conf.py
g_gemii_db_cfg = {
    'dbtype': 'mysql',
    'host': 'lizcloud4c.chnh6yhldzwc.rds.cn-north-1.amazonaws.com.cn',
    'port': 3306,
    'user': 'root',
    'password': 'Rg3UrelC',
    'dbname': 'jingli_knowledge',
}

g_qa_search_api_cfg = {
    'host': '127.0.0.1',    # 配置成qa-search前端的nginx服务地址
}
```

- 启动

``` bash
cd ~/online-services/qa-search
source venv/bin/activate
supervisord -c conf/sup.qsearch.conf
supervisorctl -c conf/sup.qsearch.conf        # 查看状态
```

端口占用 8131 - 8136

- nginx

``` bash
    ...
    upstream py_in_qasearch_jingli {
        server 127.0.0.1:8131;
        server 127.0.0.1:8132;
        server 127.0.0.1:8133;
        server 127.0.0.1:8134;
        server 127.0.0.1:8135;
        server 127.0.0.1:8136;
    }
    ...
    # @hailong: jingli qasearch api (internal api)
    location /in/nlp/tob/ {
        proxy_pass http://py_in_qasearch_jingli;
    }
    ...
```

# 问答接口模块

`/home/works/online-services/qa-coapi-jingli`
该模块提供对外接口服务，具体功能包括：用户鉴权，输入校验，调用检索/排序模块，生成topN条候选回复，按接口文档格式打包返回给调用方。该模块是Python开发的web service，调用内部服务，对外提供HTTP方式的访问接口。
该模块由virtualenv提供隔离运行环境，且通过supervisor进行守护。所以，需要先激活virtualenv环境并安装好该模块依赖的第三方packages才能正常启动。

- 配置

``` python
# bin/conf.py
g_gemii_db_cfg = {
    'dbtype': 'mysql',
    'host': 'lizcloud4c.chnh6yhldzwc.rds.cn-north-1.amazonaws.com.cn',
    'port': 3306,
    'user': 'root',
    'password': 'Rg3UrelC',
    'dbname': 'jingli_knowledge',
}
```

- 启动

``` bash
cd ~/online-services/qa-search
source venv/bin/activate
supervisord -c conf/sup.qa_coapi.conf
supervisorctl -c conf/sup.qa_coapi.conf        # 查看状态
```

端口占用 8250 - 8255

- nginx

``` conf
    ...
    upstream py_coapi_jingli {
        server 127.0.0.1:8250;
        server 127.0.0.1:8251;
        server 127.0.0.1:8252;
        server 127.0.0.1:8253;
        server 127.0.0.1:8254;
        server 127.0.0.1:8255;
    }
    ...
    # @zhenguo: jingli ai service
    location /v1/dialog/ {
        proxy_pass http://py_coapi_jingli;
    }
    location /v1/knowledge/ {
        proxy_pass http://py_coapi_jingli;
    }
    location /v1/analytics/ {
        proxy_pass http://py_coapi_jingli;
    }
    location /v1/issue/ {
        proxy_pass http://py_coapi_jingli;
    }
    ...
```

# 知识点推荐及接口统计模块

`/home/works/online-services/qa-gemmi-stats`

该模块负责统计挖掘高频但未成功召回的知识点，并为其推荐相关的知识点，推荐结果供管理人员审核。此外，问答系统接口相关的统计脚本（如接口调用次数、召回率）也由该模块实现。

该模块是Python开发的脚本，完成知识点推荐、接口数据统计等离线需求,模块由virtualenv提供隔离运行环境,通过crontab定时运行(运行频率：1次/天).在works帐号的crontab任务列表里添加如下配置(每天早上05:01执行1次):

- 配置

``` python
# bin/config.py
DB_HOST = 'lizcloud4c.chnh6yhldzwc.rds.cn-north-1.amazonaws.com.cn'
DB_PORT = '3306'
DB_USER = 'root'
DB_PASSWORD = 'Rg3UrelC'
DB_NAME = 'jingli_knowledge'
```

``` bash
# bin/configure.sh
host="lizcloud4c.chnh6yhldzwc.rds.cn-north-1.amazonaws.com.cn"
port=3306
user="root"
password="Rg3UrelC"
database="jingli_knowledge"
```

- 自启动

``` bash
crontab -e

01 05 * * * cd /home/works/online-services/qa-gemii-stats && (source venv/bin/activate && sh ./bin/run.sh 2>&1 1>cron.log)
```
