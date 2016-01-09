方法一：使用pip安装
       
pip install mongo-connector

安装到了ython的默认包目录下：

/usr/local/lib/python2.7/site-packages


方法二：安装为服务

1. 去https://github.com/mongodb-labs/mongo-connector/archive/master.zip下载mongo-connector-master.zip。

2. 解压缩进入目录。
       
unzip mongo-connector-master.zip
cd mongo-connector-master

3. 编辑配置文件。
       
vi config.json

4. 安装为服务。
       
python setup.py install_service

在/etc/init.d下创建了mongo-connector服务，并拷贝config.json文件到/etc/mongo-connector.json。


卸载mongo-connector服务
       
python setup.py uninstall_service

它将移除/etc/init.d/mongo-connector和/etc/mongo-connector.json


查看服务状态
       
service mongo-connector status


配置Solr


在Solr数据目录/data/solr/data/下有Solr配置文件solr.xml


创建core
       
su - solr -c "/usr/local/solr/solr/bin/solr create -c card -n data_driven_schema_configs"

生成了文件夹/data/solr/data/card，在/data/solr/data/card/conf目录下是card的配置目录，可以配置同义词、停止词。


配置solrconfig.xml

1. 确保启用了LukeRequestHandler

以下行应用出现在solrconfig.xml文件中。
       
<requestHandler name="/admin/luke" class="org.apache.solr.handler.admin.LukeRequestHandler" />

2. 配置硬提交（刷新到硬盘上的频率）和软提交（提交到内存中的频率）

在solrconfig.xml文件中配置<autoCommit>和<autoSoftCommit>
       
<autoCommit>
<maxTime>300000</maxTime>
<maxDocs>10000</maxDocs>
<openSearcher>true</openSearcher>
</autoCommit>
<!-- softAutoCommit is like autoCommit except it causes a
'soft' commit which only ensures that changes are visible
but does not ensure that data is synced to disk. This is
faster and more near-realtime friendly than a hard commit.
-->
<autoSoftCommit>
<maxDocs>1000</maxDocs>
<maxTime>60000</maxTime>
</autoSoftCommit>


配置schema.xml

1. Mongo Connector存储元数据在每个文档中帮助处理回滚。为了支持这些数据，你需要添加如下信息到你的schema.xml中：
       
<field name="_ts" type="long" indexed="true" stored="true" />
<field name="ns" type="string" indexed="true" stored="true"/>

2. 在schema.xml中配置配置<uniqueKey>、<field>


启动mongo-connector


方法一：以命令行启动
       
nohup sudo mongo-connector -m localhost:27019 -t http://localhost:8983/solr/card -o oplog_progress.txt -n example.card -u _id -d solr_doc_manager > mongo-connector.out 2>&1


方法二：以服务启动
       
service mongo-connector start


Solr删除全部索引


http://192.168.11.52:8983/solr/card/update/?stream.body=%3Cdelete%3E%3Cquery%3E*:*%3C/query%3E%3C/delete%3E&stream.contentType=text/xml;charset=utf-8&commit=true
