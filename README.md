#Mongo-connector集成MongoDB到Solr实现增量索引

# 1.配置MongoDB复制集
    1.1为每个节点建立必要的数据文件夹

    mkdir -p /srv/mongodb/rs0-0 
    mkdir -p /srv/mongodb/rs0-1 
    mkdir -p /srv/mongodb/rs0-2

    1.2 通过下述命令来启动 mongod 实例
    mongod --port 27017 --dbpath /srv/mongodb/rs0-0 --replSet rs0 --smallfiles --oplogSize 128 --logappend --logpath /srv/mongodb/rs0-0/mongod.log &
    mongod --port 27018 --dbpath /srv/mongodb/rs0-1 --replSet rs0 --smallfiles --oplogSize 128 --logappend --logpath /srv/mongodb/rs0-1/mongod.log &
    mongod --port 27019 --dbpath /srv/mongodb/rs0-2 --replSet rs0 --smallfiles --oplogSize 128 --logappend --logpath /srv/mongodb/rs0-2/mongod.log &


    1.3我们可以通过下列命令来连接到第一个实例：  

    mongo --port 27017


    1.4 在 mongo 中使用 rs.initiate() 来初始化复制集。我们可以通过下列方式来设定复制集配置对象：  

    rsconf = {    
                  _id: "rs0",    
                  members: [    
                      {    
                       _id: 0,    
                       host: "主机名或者IP:27017"    
                      }  ,
					  {    
                       _id: 1,    
                       host: "主机名或者IP:27018"    
                      } ,
					  {    
                       _id: 2,    
                       host: "主机名或者IP:27019"    
                      }   
                    ]    
             }

    rs.initiate( rsconf )


    1.5查看复制集配置
    rs.conf()
    1.6使用 mongo 连接到 primary，并用过 rs.add() 命令来添加第二个和第三个 mongod 实例到复制集中。

	rs.add("主机名或者IP:27018")   
    rs.add("主机名或者IP:27019")
    1.7通过 rs.status() 命令来检查复制集的状态。

#   2. 在CentOS下安装Solr5.3 


    [SOLR]http://www.apache.org/dyn/closer.lua/lucene/solr/5.3.0下载Solr安装文件solr-5.3.0.tgz。

    2.1 将solr-5.3.0.tgz文件放到/tmp目录下，执行如下脚本：    
  	
    # cd /tmp    
    # tar -zxvf solr-5.3.0.tgz // 解压压缩包

    创建应用程序和数据目录	
    # mkdir -p /data/solr /usr/local/solr

    创建运行solr的用户并赋权   	
    # groupadd solr    
    # useradd -g solr solr    
    # chown -R solr.solr /data/solr /usr/local/solr

    安装solr服务	
    # solr-5.3.0/bin/install_solr_service.sh solr-5.3.0.tgz -d /data/solr -i /usr/local/solr
    检查服务状态  
    # service solr status

    将会看到如下输出：  
    Solr process 29692 running on port 8983    
    {    
      "solr_home":"/data/solr/data/",    
      "version":"5.3.0 1696229 - noble - 2015-08-17 17:10:43",    
      "startTime":"2015-09-16T01:32:03.919Z",    
      "uptime":"0 days, 0 hours, 3 minutes, 6 seconds",    
      "memory":"89.8 MB (%18.3) of 490.7 MB"
    }
    solr命令用法
    定位到solr应用程序目录 
    
    # cd /usr/local/solr/solr
    
    查看solr命令选项   	
    # ./bin/solr
    # ./bin/solr start -help
    # ./bin/solr create -help

    安装solr服务脚本用法

      运行安装脚本
   
    	
      # /tmp/solr-5.3.0/bin/install_solr_service.sh

      创建集合
        在这个部分，我们创建一个简单的Solr集合。
        Solr可以有多个集合，但在这个示例，我们只使用一个。使用如下命令，创建一个新的集合。我们以solr用户运行以避免任何权限错误。
        # su - solr -c "/usr/local/solr/solr/bin/solr create -c gettingstarted -n data_driven_schema_configs"
        在这个命令中，gettingstarted是集合的名字，-n指定配置集合。Solr默认提供了3个配置集合。这里我们使用的是schemaless，意思是可以提供任意名字的任意列，类型将会被猜测。




# 3. 在CentOS下安装Python2.7

    操作步骤如下：

      1）安装devtoolset
       1 yum groupinstall "Development tools"

 

    2）安装编译Python需要的包包
        yum install zlib-devel   
        yum install bzip2-devel    
        yum install openssl-devel    
        yum install ncurses-devel    
        yum install sqlite-devel

 

    3）下载并解压Python 2.7.9的源代码
        cd /opt   
        wget --no-check-certificate https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tar.xz    
        tar xf Python-2.7.9.tar.xz    
        cd Python-2.7.9

 

    4）编译与安装Python 2.7.9
       ./configure --prefix=/usr/local   
        make && make altinstall

 

    5）将python命令指向Python 2.7.9
       ln -s /usr/local/bin/python2.7 /usr/local/bin/python
    6）检查Python版本
       python -V
       
       
       
# 4. 在CentOS下安装pip


    Python和操作系统支持

    Pip工作于CPython版本2.6，2.7，3.2，3.3，3.4以及pypy。

    pip包含于Python

    Python 2.7.9及之后（在python2系列），和Python 3.4和之后默认包含pip，因此你可能已经有了pip。


    安装pip


    去https://bootstrap.pypa.io/get-pip.py下载get-pip.py。

    运行如下命令：
  	
    python get-pip.py


    如何setuptools还没有安装，get-pip.py将会安装setuptools。


    为了升级已经存在的setuptools，运行pip install -U setuptools。


    此外，get-pip.py支持使用pip安装选项和普通选项。以下是一些示例：


    从本地的pip和setuptools副本安装：
    python get-pip.py --no-index --find-links=/local/copies
    安装到用户目录：
    python get-pip.py --user


    从proxy后安装：
    python get-pip.py --proxy=”[user:passwd@]proxy.server:port”


    升级pip
    pip install -U pip
    
    
# 5.安装mongo-connector

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


    5. 卸载mongo-connector服务
       
      python setup.py uninstall_service

      它将移除/etc/init.d/mongo-connector和/etc/mongo-connector.json


    6.  查看服务状态
       
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
