# 产品架构

![](/images/jiagou.png)

## 1. HDFS

HDFS默认采用HA方式部署，2个NameNode分别部署于master1与master2，DataNode则分配在所有Core节点上，Task不部署DataNode。

![](/images/developer/hdfs.jpg)

## 2. Yarn

Yarn同样默认采用HA方式部署，2个ResourceManager分别部署于master1与master2，NodeManager则分配在所有Core、Task节点上。

![](/images/developer/yarn.jpg)

## 3. Hive

Hive目前只支持On yarn模式，2个Hive-MetaStore分别部署于master1与master2，元数据库支持云数据库或者本地MySQL，避免了单个master节点宕机引起的Hive服务故障，可以通过HiveCli或者Beeline连接Hive服务。

![](/images/developer/hive.jpg)

## 4. HBase

HBase默认采用HA方式部署，2个HMaster分别部署于master1与master2，HRegionServer则分配在所有Core节点上。

![](/images/developer/hbase.jpg)

## 5. Spark

Spark采用On Yarn模式，具体可参考[Spark开发指南](https://docs.ucloud.cn/uhadoop/developer/sparkdev)。