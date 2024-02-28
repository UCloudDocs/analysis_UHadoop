# UHadoop服务WebUI访问

## 1. 服务WebUI信息

| 服务名称            | 部署节点         | 默认端口 |
| ------------------- | ---------------- | -------- |
| HDFS WebUI          | master1, master2 | 50070    |
| Yarn WebUI          | master1, master2 | 23188    |
| Presto/Trino WebUI  | master1          | 28080    |
| Hue                 | master1          | 8888     |
| Tez WebUI           | master2          | 19999    |
| Spark HistoryServer | master2          | 18080    |
| Flink HistoryServer | master1          | 8082     |
| Airflow WebUI       | master1          | 8999     |

说明：

* 访问前需要确认实例是否部署对应组件；
* 以上展示位集群默认端口，如果用户有修改端口信息需要按照实际端口访问；

## 2. 服务WebUI访问方式

可通过以下几种方式访问集群服务WebUI。

### 2.1 Windows云主机访问

可通过Windows云主机对集群所在服务进行访问，具体步骤如下：

* 创建一台Windows云主机：可在与UHadoop实例同地域、同VPC下创建一台Windows云主机，具体见 [选购一台uhost主机](https://docs.ucloud.cn/uhost/newuser/briefguide)。

* Windows云主机安装浏览器：IE浏览器访问服务WebUI可能存在问题，可按需安装Chrome或者Edge浏览器。

* Windows云主机配置hosts：在主机c:\windows\system32\drivers\etc\hosts文件中追加集群节点信息，格式：“集群节点内网ip 主机名”。可从节点中任意节点/etc/hosts文件中拷贝：如下图标红区域，注意标红区域外的内容不要拷贝。

  ![image-20240228115846128](/Users/user/Library/Application Support/typora-user-images/image-20240228115846128.png)



### 2.1 节点EIP访问

实例master1与master2节点支持绑定EIP，可通过绑定EIP方式进行访问，步骤如下：

* 创建EIP：具体见[申请EIP](https://docs.ucloud.cn/unet/eip/guide?id=%e7%94%b3%e8%af%b7%e5%bc%b9%e6%80%a7ip)

* 创建或者更新防火墙策略：具体见 [防火墙操作指南](https://docs.ucloud.cn/unet/firewall/guide) ，其中配置防火墙时需要注意：

  * 端口按需开放：建议仅开放需要使用到端口，保证集群服务访问安全；
  * 添加原地址：达到将互联网中无关IP来源的请求过滤掉的目的，使此方式访问集群更安全。

* 集群节点绑定EIP与防火墙：可在集群管理->节点管理中对master节点绑定EIP与防火墙。

  ![webui_access_hosts](../images/webui_access_hosts.png)

该方式配置简单，适用于测试等场景，使用时一定注意集群外网安全。



其他方式需求可联系客户经理。