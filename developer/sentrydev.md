


# Sentry使用指南

介绍：

Apache Sentry是Cloudera公司发布的一个Hadoop开源组件，它提供了细粒度级、基于角色的授权以及多租户的管理模式，Sentry当前可以和Hive/Hcatalog、Apache Solr 和Cloudera Impala集成， 为这些组件提供权限管理服务。
基于角色的管理（role-based acess control）通过创建角色，将每个组件的权限授予给此角色，然后在用户(组)中添加此角色，用户便具备此角色访问组件的权限，使用sentry对hive进行权限管理时，这里的组件可以是整个server，也可以是单个db,或者单张table.




## 1.使用hive用户登陆集群

注:（测试时需要将uhadoop-sul4mhf4替换为自己集群的id)
### 1.1 查看全部数据库

```
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "show databases;"

```

### 1.2 查看全部角色

```
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "show roles;"

```

### 1.3 查看当前角色

```
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "show current roles;"

```

### 1.4 查看当前用户

```
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "select current_user();"

```

## 2. hive用户授予管理员权限

### 2.1 创建管理员角色admin

```
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "CREATE ROLE admin;"

```
### 2.2 为admin角色授予全部server权限

```
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive 
> grant all on server `uhadoop-thp4pjtb-master1` to role admin;

```

### 2.3 为hive用户赋予admin角色

```
//经过这一步，hive用户已经可以作为管理员用户执行全部数据和权限操作。
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "GRANT ROLE admin TO GROUP hive;"

```


## 3. 创建测试数据库（使用hive用户创建）

### 3.1 创建测试db1,db2

```
//使用管理员用户登陆，创建db1,db2两个数据库。
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "create database db1;create database db2;"

// 创建测试表，并插入数据

beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "create table db1.t1(id string);"

beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "insert into  db1.t1  values ('t1_001'),('t1_002');"

beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "create table db2.t2(id string);"

beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "insert into  db2.t2  values ('t2_001'),('t2_002');"

```


### 3.2 master1,master2节点上创建linux测试用户user1, user2

```
useradd -M -s /sbin/nologin user1

useradd -M -s /sbin/nologin user2

```

### 3.3 hive中创建两个角色，分别授予不同的角色权限

```
//创建角色role1, 授予其对db1的管理权限
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "CREATE ROLE role1;"
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "grant all on database db1 to role role1 with grant option;"


//创建角色role2, 授予其对db2的管理权限
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "CREATE ROLE role2;"
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "grant all on database db2 to role role2 with grant option;"

// show grant role role1; (查看role1角色的权限列表)
// show grant role role2; (查看role2角色的权限列表)

```



### 3.4 管理员用户登陆hive,为两个用户赋予不同的角色

```
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "GRANT ROLE role1 TO GROUP user1;"


beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n hive -e "GRANT ROLE role2 TO GROUP user2;"

//show role grant group user1 (查看user1的角色列表)
// show role grant group user2（查看user2的角色列表）

```

## 4 使用user1, user2用户登陆，验证权限隔离

```
//user1登陆，只能看到db1数据库
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n user1 -e "show databases;"

// user2用户登陆，只能看到db2数据库
beeline -u "jdbc:hive2://uhadoop-sul4mhf4-master2:10000"  -n user2 -e "show databases;"

```

## 5. 其他使用测试

### 5.1 将role从角色中剔除
    REVOKE ROLE role1 FROM GROUP user1;

### 删除role
    //先查看角色列表
    show roles

    // 删除角色
    drop role role2;


### 角色权限撤销

    // 先查看角色当前授权信息
    show grant role role1;

    // 将db1的操作权限从role1撤销
    revoke all on database db1 from role role1;



授权语句说明：

角色授权和撤销
```
GRANT ROLE role_name [, role_name] TO GROUP <groupName> [,GROUP <groupName>]
REVOKE ROLE role_name [, role_name] FROM GROUP <groupName> [,GROUP <groupName>]


```

权限的授予和撤销

 ```
 GRANT <PRIVILEGE> [, <PRIVILEGE> ] ON <OBJECT> <object_name> TO ROLE <roleName> [,ROLE <roleName>]
REVOKE <PRIVILEGE> [, <PRIVILEGE> ] ON <OBJECT> <object_name> FROM ROLE <roleName> [,ROLE <roleName>]


 ```

 查看角色/组权限

```
SHOW ROLES;
SHOW CURRENT ROLES;
SHOW ROLE GRANT GROUP <groupName>;
SHOW GRANT ROLE <roleName>;
SHOW GRANT ROLE <roleName> on OBJECT <objectName>;


```


