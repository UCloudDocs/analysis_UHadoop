

# Hadoop开发指南

> 注解：本例中所运行脚本需在CentOS操作系统上，其他操作系统请修改脚本后再尝试执行。

## 1. 创建Hadoop客户端节点

UHadoop提供客户端节点与SSH两种访问模式，优先建议使用客户端访问模式，具体见[集群访问](/uhadoop/developer/access)。


## 2. HDFS

HDFS是一个高度容错性和高吞吐量的分布式文件系统。它被设计的易于扩展也易于使用，适合海量文件的存储。

#### 2.1 HDFS基础操作

- 查询文件
  ```
  Usage: hadoop fs [generic options] -ls [-d] [-h] [-R] [<path>]
  ```
- 上传文件
  ```
  Usage: hadoop fs [generic options] -put [-f] [-p] [-l]
  <localsrc> ... <dst>
  ```
- 下载文件
  ```
  Usage: hadoop fs [generic options] -get [-p] [-ignoreCrc] [-crc]
  <src> ... <localdst>
  ```
更多请参考： hadoop fs -help

#### 2.2 WebHDFS

WebHDFS提供HDFS的RESTful接口，可通过此接口进行HDFS文件操作。使用WebHDFS时，客户端是先通过Namenode节点获取文件所在的Datanode地址，再通过与Datanode节点进行数据交互。

###### 2.2.1 上传文件

UHadoop集群默认配置2个Master节点，同一时刻只有一个节点Namenode处于Active状态，另一个处于Standby状态。下面以uhadoop-\*\*\*\*\*\*-master1的Namenode为Active为例

- 数据准备

  ```
    touch uhadoop.txt
    echo "uhadoop" > uhadoop.txt
  ```

- 创建文件请求

  ```
    curl -i -X PUT "http://uhadoop-******-master1:50070/webhdfs/v1/tmp/uhadoop.txt?op=CREATE"
  ```

  > 注解：
  > 1. 需要在执行此命令机器加上集群所有节点host
  > 2. 若提示Operation category READ is not supported in state standby，请更换uhadoop-\*\*\*\*\*\*-master2尝试

  执行上述命令将获取到Location地址，即文件的Datanode地址

  ```
  HTTP/1.1 307 TEMPORARY_REDIRECT
  Location: http://<DATANODE>:<PORT>/webhdfs/v1/<PATH>?op=CREATE...
  Content-Length: 0
  ```

- 使用上述Location地址上传文件

  ```
    curl -i -X PUT -T uhadoop.txt "http://uhadoop-******-core*:50075/webhdfs/v1/tmp/uhadoop.txt?op=CREATE&namenoderpcaddress=Ucluster&overwrite=false"
  ```

###### 2.2.2 append文件

- 数据准备

  ```
    touch  append_uhadoop.txt
    echo "ucloud" > append_uhadoop.txt
  ```

- 获取被append文件地址

  ```
    curl -i -X POST "http://uhadoop-hfygbg-master1:50070/webhdfs/v1/tmp/uhadoop.txt?op=APPEND"
  ```
  执行上述命令将获取到Location地址，即文件的Datanode地址
  ```
  HTTP/1.1 307 TEMPORARY_REDIRECT
  Location: http://<DATANODE>:<PORT>/webhdfs/v1/<PATH>?op=CREATE...
  Content-Length: 0
  ```

- 追加文件

  ```
    curl -i -X POST -T append_uhadoop.txt "http://uhadoop-******-core*:50075/webhdfs/v1/tmp/uhadoop.txt?op=APPEND&namenoderpcaddress=Ucluster"
  ```

###### 2.2.3 打开读取文件

```
  curl -i -L "http://uhadoop-******-master1:50070/webhdfs/v1/tmp/uhadoop.txt?op=OPEN"
```

###### 2.2.4 删除文件
```
  curl -i -X DELETE "http://uhadoop-******-master1:50070/webhdfs/v1/tmp/uhadoop.txt?op=DELETE"
```

#### 2.3 HttpFS

Httpfs是cloudera提供的一个HDFS的http接口，可以通过WebHDFS REST
API对HDFS进行读写等访问。与WebHDFS的区别是，Httpfs不需要客户端访问集群的每一个节点，只需授权访问启动了Httpfs服务的单台机器即可（UHadoop默认在master1:14000开启Httpfs）。由于Httpfs是在内嵌的tomcat中一个Web应用，因此性能上会受到一些限制。

###### 2.3.1 上传文件

- 数据准备

  ```
    touch httpfs_uhadoop.txt
    echo "httpfs_uhadoop" > httpfs_uhadoop.txt
  ```

- 上传数据

  ```
    curl -i -X PUT -T httpfs_uhadoop.txt --header "Content-Type: application/octet-stream" "http://uhadoop-******-master1:14000/webhdfs/v1/tmp/httpfs_uhadoop.txt?op=CREATE&user.name=root&data=true"
  ```

  > 注解：
  > 1. 需要在执行此命令机器加上集群master1的host
  > 2. url中需添加user.name，否则会报"HTTP Status 401 - Authentication required"错误

###### 2.3.2 append文件

- 数据准备

  ```
    touch append_httpfs.txt
    echo "append_httpfs" > append_httpfs.txt
  ```

- 追加文件

  ```
    curl -i -X POST -T append_httpfs.txt --header "Content-Type: application/octet-stream" "http://uhadoop-******-master1:14000/webhdfs/v1/tmp/httpfs_uhadoop.txt?op=APPEND&user.name=root&data=true"
  ```

###### 2.3.3 打开并读取文件
  ```
    curl -i -L "http://uhadoop-******-master1:14000/webhdfs/v1/tmp/httpfs_uhadoop.txt?op=OPEN&user.name=root"
    curl -i -X DELETE "http://uhadoop-******-master1:14000/webhdfs/v1/tmp/uhadoop.txt?op=DELETE"
  ```

###### 2.3.4 删除文件

```
  curl -i -X DELETE "http://uhadoop-******-master1:14000/webhdfs/v1/tmp/httpfs_uhadoop.txt?op=DELETE&user.name=root"
```

#### 2.4 MapReduce Job

以terasort为例，说明如何提交一个MapReduce Job

- 生成官方terasort input数据集

  ```
    hadoop jar /home/hadoop/hadoop-examples.jar teragen 100 /tmp/terasort_input
  ```

- 提交任务

  ```
    hadoop jar /home/hadoop/hadoop-examples.jar  terasort /tmp/terasort_input /tmp/terasort_output
  ```

#### 2.5 HDFS日常运维

###### 2.5.1 重启服务

重启Namenode：service hadoop-hdfs-namenode restart

重启Datanode：service hadoop-hdfs-datanode restart

重启ResourceManager: service hadoop-yarn-resourcemanager restart

重启NodeManager：service hadoop-yarn-nodemanager restart

重启整个Hadoop服务：请通过UCloud控制台集群服务管理页面操作

###### 2.5.2 查看HDFS状态，节点信息

```
  hdfs dfsadmin -report
```

###### 2.5.3 修改HDFS文件副本数量

```
  hdfs dfs -setrep -R [replication-factor] [targetDir]
```

> 示例：修改HDFS 根目录下文件副本数量为2，hdfs dfs -setrep -R 2 /

###### 2.5.4 查看HDFS文件系统状态

```
  hadoop fsck /
```

返回结果示例如下：

```
 Total size:    455660769497 B (Total open files size: 44723814 B)
 Total dirs:    47975
 Total files:   70456
 Total symlinks:        0 (Files currently being written: 11)
 Total blocks (validated):  69916 (avg. block size 6517260 B) (Total open file blocks (not validated): 10)
 Minimally replicated blocks:   69916 (100.0 %)
 Over-replicated blocks:    0 (0.0 %)
 Under-replicated blocks:   87 (0.12443504 %)
 Mis-replicated blocks:     0 (0.0 %)
 Default replication factor:    3
 Average block replication: 3.0011585
 Corrupt blocks:        0
 Missing replicas:      522 (0.24815665 %)
 Number of data-nodes:      4
 Number of racks:       1
FSCK ended at Thu Nov 24 16:08:12 CST 2016 in 2044 milliseconds

The filesystem under path '/' is HEALTHY
```

上述HEALTHY表示当前HDFS文件系统正常，无坏块或者数据丢失
