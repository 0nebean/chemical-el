# 1. Magnesium 是什么




* ### Magnesium 是一个以管理API 为核心功能的数据中台

Magnesium 通过两个核心的概念管理API，分别是应用（consumer）和服务（provider），API由服务提供，一个服务即对应了一个web service，我们把它其中的API定义到平台上进行可视化的管理，平台有了这些API加持，应用更像是一个应用市场的账号在海量的API市场里选择自己需要的API进行绑定获得授权，应用通过设定的授权方式即可访问这些API，那么我们实际意义上的开发的应用，即对应了平台上一个应用的概念，只要接入了平台就等于获得了海量的数据API的门票，并不关心提供方来自哪一个模块，哪一个数据库，用何种方式调用API，在此种模式下开发，API的复用率在得到最大化，开发成本将大幅度下降。




* ### Magnesium 是一个基于k8s的完整的devops工具链

Magnesium 基于k8s的`service`和`deployment`开发了一套流量路由组件，当项目通过gitlab发布新版本后，其将会自动部署在k8s上，k8s并会将新版本的节点访问地址推送回 Magnesium ，实现物理节点的版本切换和一键销毁过期版本等功能，路由的流量通过uag网关以合法的鉴权模式暴露出去，甚至前端项目可以在页面上配置域名及ssl证书，直接发布到nginx，不需要离开管控台页面。




* ### Magnesium 拓展了多种API及用户鉴权模式以满足多种业务场景接入数据中台的需求
```
oauth授权码(可用于开放平台，对外暴露API)

IP白名单+通行令牌(适用于管理后台接入中台，访问API)

私有令牌（可用于移动端单设备登录）

私有免登陆令牌（适合一些有自己登录体系的移动端应用，一机一令牌，直接携带上用户信息无需登录）
```






* ### Magnesium 为用户鉴权提供了独立的用户体系，但并不臃肿
在用户信息鉴权这块，Magnesium 维护了一套用户体系，按照应用分表，数据层面完全隔离，但并没有冗余过多的用户业务信息，这将会导致在网关和业务系统之间来回同步用户信息造成完全没必要的性能损失和用户数据延迟，在用户登录成功后，Magnesium 会将用户身份信息加工在请求头内再让其进行穿透转发到服务上，而服务不关心用户以何种业务逻辑登录，这取决于应用授权模式的配置。





* ### Magnesium 为各种语言开发的restful API提供了无差别的接入和消费方式，解耦业务代码
Magnesium 在请求穿透合法网关后，将用户和API鉴权信息无差别的转发到业务层服务上，并不要求服务用同一种语言开发，业务层的服务，都可以在请求头中获取风格统一的应用信息，和当前登录用户信息。




* ### Magnesium 租户中心 模块 为 Silicon 提供了一站式的账号下发服务
[`Silicon`](https://0nebean.github.io/Silicon/)是SaaS结构的管理后台，所有租户的用户权限机构等沙箱完全隔离，互不影响，而Magnesium 的`租户中心` 为其提供了一站式的租户账号激活下发，管控功能。



---

# 2. 结构和介绍

## 2.1 组件结构

Magnesium 由以下组件构成，其功能含义如下：

### java模块：
```
mngt-portal ===> 云控制台框架 （web模块 UI）
server-mngt ===> 云控制台-服务应用管理中心 （web模块 UI）
tenant-mngt ===> 云控制台-租户管理中心 （web模块 UI）（选装功能）
user-mngt ===> 用户管理中心 （web模块 UI）
api-adapter ===> api 适配器 （web模块）
message-center ===> 消息中心 （web模块）
```

### lua模块：
```
uag ===> user api gateway API网关业务lua实现  （openresty模块）
```

### jar包API：
```
uag-log ===> 云控台操作日志api
uag-sso ===> 单点登录api
```

<br/>
<br/>


Magnesium 由4个带有UI的模块，其功能分别是： 
`server-mngt`对服务应用及devops功能，提供了一个管理界面的功能, 
`user-mngt`则是对auth部分用户信息提供了一个管理界面的功能，
`mngt-portal`是整个平台的UI框架并集成了单点登录的API，
`tenant-mngt`则是对[`Sodium`](https://0nebean.github.io/Sodium/)SaaS后台管理系统，SaaS租户账号管理下发， 
文档的后面部分会对这些UI管理的功能做具体的介绍。

<br/>
<br/>

架构图解：

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/58.png)

---

# 3.  运行和访问

## 3.1 中台启动访问指南



### 3.1.1. 环境准备：
Magnesium 运行环境及依赖的中间件如下，请按照清单准备好：
* JDK-8
* maven-3
* mysql
* 携程apollo
* redis
* rabbit-mq
* eureka
* Kubernetes || minikube
* gitlab-ce || gitlab-ee
* nexus repository manager-3 
================================  
以及以下依赖需要部署到你的nexus私服中： 
[onebean.boot.stater](https://github.com/0nebean/onebean.boot.stater) 
[net.onebean.core](https://github.com/0nebean/net.onebean.core) 
[net.onebean.data.redis](https://github.com/0nebean/net.onebean.data.redis) 
[net.onebean.data.druid](https://github.com/0nebean/net.onebean.data.druid) 
[net.onebean.data.jsch](https://github.com/0nebean/net.onebean.data.jsch) 
以及项目中本身的模块： 
[api.adapter.api](https://github.com/0nebean/Magnesium/tree/master/api-adapter/net.onebean.api.adapter.api) 
[server.mngt.api](https://github.com/0nebean/Magnesium/tree/master/server-mngt/net.onebean.server.mngt.api) 
[user.mngt.api](https://github.com/0nebean/Magnesium/tree/master/user-mngt/net.onebean.user.mngt.api) 


---

### 3.1.2. 初始化数据库脚本 

执行 [初始化数据库脚本](https://github.com/0nebean/Magnesium/wiki/%E5%88%9D%E5%A7%8B%E5%8C%96%E6%95%B0%E6%8D%AE%E8%84%9A%E6%9C%AC)，初始化数据库 `mgnesium_devops` `mgnesium_message_center` `mgnesium_uag_account`


---


### 3.1.3. 安装启动Openresty

1. 在`uag`模块配置文件`platform.conf`中，设置你自己的uag域名:

```
server {

	listen 80;
	server_name ${uag域名};

	access_log /usr/local/openresty/nginx/uag/logs/platform/access.log main;
	error_log  /usr/local/openresty/nginx/uag/logs/platform/error.log debug;
.......
```
2. 安装启动 [安装openresty并启动uag](https://github.com/0nebean/Magnesium/wiki/openresty%E6%A8%A1%E5%9D%97%E7%9A%84%E4%BE%9D%E8%B5%96%E5%AE%89%E8%A3%85%E4%B8%8E%E5%90%AF%E5%8A%A8)  


---
### 3.1.4. 修改代码仓库地址
1. 修改[onebean.boot.stater](https://github.com/0nebean/onebean.boot.stater) 中的nexus maven仓库地址到你的私服上，这样全局的仓库地址都被修改：
```

......
    <name>onebean-boot-stater</name>
    <description>框架公共依赖</description>
    <properties>
        <java.version>1.8</java.version>
        <child.model.version>1.0.0-SNAPSHOT</child.model.version>
        <api.rest.version>1.0.0-SNAPSHOT</api.rest.version>
        <core.version>1.0.0-SNAPSHOT</core.version>
        <thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
        <repo.snapshots.url>${快照仓库地址}</repo.snapshots.url>
        <repo.releases.url>${正式仓库地址}</repo.releases.url>
    </properties>
......
```

2. 修改所有运行在k8s中模块的 `.gitlab-ci.yml` , `deployment.yml` 文件中的docker私服地址为你的nexus地址：
```
    .gitlab-ci.yml:

    - docker login ${仓库地址} -u=${仓库用户} -p='{仓库密码}'
    - docker build -t ${仓库地址}/server-mngt:${CI_COMMIT_TAG} .
    - docker push ${仓库地址}/server-mngt:${CI_COMMIT_TAG}
    - docker rmi ${仓库地址}/server-mngt:${CI_COMMIT_TAG}
    - rm -rf ./app

    deployment.yml:

        - name: server-mngt-pod-kubernetes_version
          image: ${仓库地址}/server-mngt:latest
          imagePullPolicy: Always
```


---

### 3.1.5. 部署项目到GitLab
安装gitlab-ee || gitlab-ce，和gitlab runner，并注册shell-runner到gitlab，在gitlab上，为这些模块创建项目，并提交代码

---

### 3.1.6. gitlab 设置CI参数：
CI参数可以设置到每一个项目中，也可以设置group，在组下创建项目，这样参数只需要设置到group中即可，参数有四：

```shell
 key:DEFAULT_VM_ARGS
 value:
 -server -Xms128m -Xmx128m -Xmn256m -XX:MetaspaceSize=128M -XX:MaxMetaspaceSize=128M -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:-OmitStackTraceInFastThrow
```

```yaml
 key:KUBE_CONFIG
 value:${你的k8s的config文件内容}
```

需要注意的是，这两个参数设置的时候，value必须base64

```yaml
 key:UAG_HOST
 value:${步骤三里设置的uag的域名}
```


```yaml
 key:UAG_PORT
 value:${步骤三里设置的uag的端口}
```


---

### 3.1.7. 更新服务器节点配置

```sql
USE `mgnesium_devops`
UPDATE  `t_server_machine_node` t SET t.ip_address = '${openresty.ip}',t.access_user = ${openresty.user} ,t.access_password = ${openresty.pass}, t.access_port = ${openresty.ssh.port}  WHERE t.server_machine_type = '0';
UPDATE  `t_server_machine_node` t SET t.ip_address = '${kubernetes-master.ip}',t.access_user = ${kubernetes-master.user} ,t.access_password = ${kubernetes-master.pass}, t.access_port = ${kubernetes-master.ssh.port}  WHERE t.server_machine_type = '1';

```

### 3.1.8. 按照以下顺序提交git tag 
---
提交git tag 'dev_v.x.x.x'，这里gitlab runner 会把程序发布到k8s上 

1. api-adapter （启动成功后再部署下一个）
2. server-mngt （启动成功后再部署下一个）

这里中断一下先不要急着部署后面的模块，从k8s控制台上获取一下这两个模块的service的nodePort和masterIP，来更新一下数据库：
```sql
USE `mgnesium_devops`;
UPDATE `t_upsteam_node` t SET t.physical_path = '${masterIp:nodePort}' WHERE node_name = 'api-adapter';
UPDATE `t_upsteam_node` t SET t.physical_path = '${masterIp:nodePort}' WHERE node_name = 'server-mngt';
```
这里一定要确定两个项目已经正常启，然后执行一下 `步骤9`，确认mq执行正常，没有阻塞，再继续

3. message-center mngt-portal tenant-mngt user-mngt  
后面这些模块的访问地址k8s会自动通知到Magnesium，不需要再操作数据库  
在k8s的控制台日志上确定各模块都启动成功，开始下一步

---

### 3.1.9. 设置控制台访问的域名 
执行以下脚本,这里的域名全部解析指向步骤3里的uag域名所在ip即可:
```sql
USE `mgnesium_devops`
UPDATE  `t_server_info` t SET t.server_host = 'user-mngt.onebean.net' WHERE t.upsteam_node_name = 'user-mngt';
UPDATE  `t_server_info` t SET t.server_host = 'server-mngt.onebean.net' WHERE t.upsteam_node_name = 'server-mngt';
UPDATE  `t_server_info` t SET t.server_host = 'mngt-portal.onebean.net' WHERE t.upsteam_node_name = 'mngt-portal';
UPDATE  `t_server_info` t SET t.pc_page_url = 'http://server-mngt.onebean.net' WHERE t.app_name = '服务中心';
UPDATE  `t_server_info` t SET t.pc_page_url = 'http://user-mngt.onebean.net' WHERE t.app_name = '用户中心';
UPDATE  `t_server_info` t SET t.oauth_base_url = 'http://mngt-portal.onebean.net' WHERE t.app_name = '云管控台';
```

---
### 3.1.10. MQ控制台发送同步消息
在rabbitMQ控制台发送消息 `sync` 给一下两个队列
```properties
devops.update.openresty.upsteam.node
devops.update.server.or.api
```
![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/53.png)



## 3.2 配置文件

Magnesium框架使用[`Apollo`](https://github.com/ctripcorp/apollo)作为配置中心 ，以下是需要的配置namespace。  

当然你也可以不使用配置中心,直接将配置文件放置到项目类`\src\main\resources`目录下也是可以的。  



* #### **[api-adapter](#api-adapter-1)**
* #### **[mngt-portal](#mngt-portal-1)**
* #### **[server-mngt](#server-mngt-1)**
* #### **[user-mngt](#user-mngt-1)**
* #### **[message-center](#message-center-1)**



<br/>

---

<br/>

* #### **api-adapter**

`application.properties`
```properties
server.port = 8080
server.context-path = 
bootstrap.without.jdbc.datasource = true
#spring
spring.http.converters.preferred-json-mapper = fastjson
spring.http.encoding.charset = UTF-8
spring.http.encoding.enabled = true
spring.http.encoding.force = true
spring.http.multipart.maxFileSize = 100Mb
spring.http.multipart.maxRequestSize = 1000Mb
simple.client.http.request.factory.read.timeout = 5000
simple.client.http.request.factory.connect.timeout = 15000
#accessToken
access.token.expire = 7200
#sms
uag.login.sms.time.out = 70
#logging
logging.level.root = info
```
[public-conf.spring-cloud](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.spring-cloud.properties)  
[public-conf.rabbitmq](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.rabbitmq.properties)


<br/>

---

<br/>


* #### **mngt-portal**

`application.properties`
```properties
server.port = 8080
server.context-path = 
bootstrap.without.jdbc.datasource = true
#spring
spring.http.converters.preferred-json-mapper = fastjson
spring.http.encoding.charset = UTF-8
spring.http.encoding.enabled = true
spring.http.encoding.force = true
spring.http.multipart.maxFileSize = 100Mb
spring.http.multipart.maxRequestSize = 1000Mb
simple.client.http.request.factory.read.timeout = 5000
simple.client.http.request.factory.connect.timeout = 15000
#accessToken
access.token.expire = 7200
#sms
uag.login.sms.time.out = 70
#logging
logging.level.root = info
```
[public-conf.spring-cloud](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.spring-cloud.properties)  
[public-conf.sso](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.sso.properties)  

<br/>

---

<br/>


* #### **server-mngt**

`application.properties`
```properties
server.port = 8080
server.context-path = 
bootstrap.without.jdbc.datasource = false
#spring
spring.http.converters.preferred-json-mapper = fastjson
spring.http.encoding.charset = UTF-8
spring.http.encoding.enabled = true
spring.http.encoding.force = true
spring.http.multipart.maxFileSize = 100Mb
spring.http.multipart.maxRequestSize = 1000Mb
simple.client.http.request.factory.read.timeout = 5000
simple.client.http.request.factory.connect.timeout = 15000

#logging
logging.level.root = info

#uag
uag.local.relative.path = /usr/local/openresty/nginx/uag/conf
uag.remote.backup.path = /tmp/uag/backup
uag.remote.delete.path = /tmp/uag/delete

```
(mysql schema:mgnesium_devops)
[public-conf.jdbc](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.jdbc.properties)  
[public-conf.spring-cloud](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.spring-cloud.properties)  
[public-conf.rabbitmq](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.rabbitmq.properties)  
[public-conf.redis](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.redis.properties)必须和user-mngt同db

<br/>

---

<br/>



* #### **user-mngt**

`application.properties`
```properties
server.port = 8080
server.context-path = 
bootstrap.without.jdbc.datasource = false
#spring
spring.http.converters.preferred-json-mapper = fastjson
spring.http.encoding.charset = UTF-8
spring.http.encoding.enabled = true
spring.http.encoding.force = true
spring.http.multipart.maxFileSize = 100Mb
spring.http.multipart.maxRequestSize = 1000Mb
simple.client.http.request.factory.read.timeout = 5000
simple.client.http.request.factory.connect.timeout = 15000

#logging
logging.level.root = info

```
(mysql schema:mgnesium_uag_account)
[public-conf.jdbc](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.jdbc.properties)  
[public-conf.spring-cloud](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.spring-cloud.properties)  
[public-conf.rabbitmq](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.rabbitmq.properties)  
[public-conf.redis](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.redis.properties)必须和uag-conf同db


<br/>

---

<br/>


* #### **message-center**

`application.properties`
```properties
server.port = 8080
server.context-path = 
bootstrap.without.jdbc.datasource = false
#spring
spring.http.converters.preferred-json-mapper = fastjson
spring.http.encoding.charset = UTF-8
spring.http.encoding.enabled = true
spring.http.encoding.force = true
spring.http.multipart.maxFileSize = 100Mb
spring.http.multipart.maxRequestSize = 1000Mb
salesIdToMony = 200

#logging
logging.level.root = info

#rest
simple.client.http.request.factory.read.timeout = 50000
simple.client.http.request.factory.connect.timeout = 15000

#send channel
msg.send.channel = 1


```
(mysql schema:mgnesium_message_center)
[public-conf.jdbc](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.jdbc.properties)  
[public-conf.spring-cloud](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.spring-cloud.properties)  
[public-conf.rabbitmq](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.rabbitmq.properties)  
[public-conf.redis](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.redis.properties)建议与server-mngt，user-mngt的db分离



## 3.3 初始化数据脚本

```sql
/*
SQLyog Ultimate v12.09 (64 bit)
MySQL - 5.7.28 : Database - mgnesium_devops
*********************************************************************
*/


/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`mgnesium_devops` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */;

USE `mgnesium_devops`;

/*Table structure for table `t_api_info` */

DROP TABLE IF EXISTS `t_api_info`;

CREATE TABLE `t_api_info` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `api_name` varchar(64) NOT NULL DEFAULT '' COMMENT '接口名称',
  `proxy_path` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '代理地址',
  `api_uri` varchar(225) NOT NULL DEFAULT '' COMMENT '接口地址',
  `api_status` char(1) NOT NULL DEFAULT '0' COMMENT '接口状态 0:未启用 1启用',
  `server_id` int(11) NOT NULL DEFAULT '0' COMMENT '服务ID',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=175 DEFAULT CHARSET=utf8 COMMENT='接口信息';

/*Data for the table `t_api_info` */

insert  into `t_api_info`(`id`,`api_name`,`proxy_path`,`api_uri`,`api_status`,`server_id`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (105,'发送登录短信验证码','/auth/sendLoginSms','/auth/sendLoginSms','1',18,'2019-04-28 17:51:49','2019-11-09 15:37:11',2,'0neBean','0'),(166,'短信验证码登录注册二合一','/auth/smsCodeLoginRegister','/userAuth/smsCodeLoginRegister','1',15,'2019-06-17 16:45:42','2019-11-09 15:37:11',2,'0neBean','0'),(167,'账号密码登录','/auth/passwordLogin','/userAuth/passwordLogin','1',15,'2019-06-17 16:46:17','2019-11-09 15:37:11',2,'0neBean','0'),(168,'短信验证码登录','/auth/smsCodeLogin','/userAuth/smsCodeLogin','1',15,'2019-06-18 14:20:50','2019-11-09 15:37:11',2,'0neBean','0'),(169,'短信验证码注册','/auth/smsCodeRegister','/userAuth/smsCodeRegister','1',15,'2019-06-18 14:21:15','2019-11-09 15:37:11',2,'0neBean','0'),(170,'账号密码注册','/auth/passwordRegister','/userAuth/passwordRegister','1',15,'2019-06-18 14:21:44','2019-11-09 15:37:11',2,'0neBean','0'),(174,'登出接口','/auth/logOut','/userAuth/logOut','1',15,'2019-08-10 10:20:32','2019-11-09 15:37:11',2,'0neBean','0');

/*Table structure for table `t_app_api` */

DROP TABLE IF EXISTS `t_app_api`;

CREATE TABLE `t_app_api` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `app_id` int(11) NOT NULL DEFAULT '0' COMMENT '应用ID',
  `api_id` int(11) NOT NULL DEFAULT '0' COMMENT '接口ID',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8 COMMENT='应用接口关联关系';

/*Data for the table `t_app_api` */

insert  into `t_app_api`(`id`,`app_id`,`api_id`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (1,27,105,'2019-10-22 23:16:13','2019-11-09 15:37:04',2,'0neBean','0'),(2,27,166,'2019-10-22 23:16:14','2019-11-09 15:37:04',2,'0neBean','0'),(3,27,167,'2019-10-22 23:16:15','2019-11-09 15:37:04',2,'0neBean','0'),(4,27,168,'2019-10-22 23:16:16','2019-11-09 15:37:04',2,'0neBean','0'),(5,27,169,'2019-10-22 23:16:19','2019-11-09 15:37:04',2,'0neBean','0'),(6,27,170,'2019-10-22 23:16:34','2019-11-09 15:37:04',2,'0neBean','0'),(7,27,174,'2019-10-22 23:16:36','2019-11-09 15:37:04',2,'0neBean','0');

/*Table structure for table `t_app_info` */

DROP TABLE IF EXISTS `t_app_info`;

CREATE TABLE `t_app_info` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `app_version` varchar(64) NOT NULL DEFAULT '' COMMENT 'app版本',
  `login_type` char(1) NOT NULL DEFAULT '' COMMENT '登录模式 0:通用验证码登录 1:账号密码登录',
  `auth_type` char(1) NOT NULL DEFAULT '0' COMMENT '授权模式 0:oauth授权码,1:oauth私有令牌,2:ip白名单+通信令牌,3:oauth私有免登陆令牌',
  `inner_api_token` varchar(64) NOT NULL DEFAULT '' COMMENT 'inner api 访问令牌',
  `open_id` varchar(64) NOT NULL DEFAULT '' COMMENT '对外暴露应用ID',
  `secret` varchar(255) NOT NULL DEFAULT '' COMMENT '秘钥',
  `app_name` varchar(64) NOT NULL DEFAULT '' COMMENT 'app名称',
  `app_category` char(1) NOT NULL DEFAULT '0' COMMENT 'app分类 0:基础应用,1:普通应用,2云应用',
  `app_status` char(1) NOT NULL DEFAULT '0' COMMENT 'app状态 0:连接 1:断开',
  `menu_sort` tinyint(4) NOT NULL DEFAULT '0' COMMENT '菜单排序',
  `oauth_base_url` varchar(255) NOT NULL DEFAULT '' COMMENT 'oauth 授权url',
  `mobile_page_url` varchar(255) NOT NULL DEFAULT '' COMMENT '移动端地址链接',
  `mobile_menu_icon` varchar(64) NOT NULL DEFAULT '' COMMENT '移动端图标icon',
  `pc_page_url` varchar(255) NOT NULL DEFAULT '' COMMENT 'PC端地址链接',
  `pc_menu_icon` varchar(64) NOT NULL DEFAULT '' COMMENT 'PC端图标icon',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=29 DEFAULT CHARSET=utf8 COMMENT='应用信息';

/*Data for the table `t_app_info` */

insert  into `t_app_info`(`id`,`app_version`,`login_type`,`auth_type`,`inner_api_token`,`open_id`,`secret`,`app_name`,`app_category`,`app_status`,`menu_sort`,`oauth_base_url`,`mobile_page_url`,`mobile_menu_icon`,`pc_page_url`,`pc_menu_icon`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (15,'','','','','57977203211','1eb7d1dc6f51b7cb083884f0c5452883','租户中心','0','0',3,'','','','http://tenant-mngt.onebean.net','md-key','2019-05-16 11:26:40','2019-11-09 15:37:46',2,'0neBean','0'),(19,'','','','','58497473971','f9f7295972fbefcc64524f05fc62d114','服务中心','0','0',2,'','','','http://server-mngt.onebean.net','md-analytics','2019-05-22 11:57:53','2019-11-09 15:35:16',2,'0neBean','0'),(21,'','','','','60643759551','b5c4eb3f5279f5a0da80e8c8ef2f42dc','用户中心','0','0',4,'','','','http://user-mngt.onebean.net','md-contacts','2019-06-16 08:09:19','2019-11-09 15:37:46',2,'0neBean','0'),(27,'','2','1','','60917091710','8b197ecc0f8413e77eac5b144e3d2484','云管控台','1','0',0,'http://mngt-portal.onebean.net','','','','','2019-06-19 12:04:51','2019-11-09 15:35:17',2,'0neBean','0');

/*Table structure for table `t_ip_white_list` */

DROP TABLE IF EXISTS `t_ip_white_list`;

CREATE TABLE `t_ip_white_list` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `ip_address` varchar(64) NOT NULL DEFAULT '' COMMENT 'ip地址',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='ip白名单';

/*Data for the table `t_ip_white_list` */

/*Table structure for table `t_server_info` */

DROP TABLE IF EXISTS `t_server_info`;

CREATE TABLE `t_server_info` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `server_name` varchar(64) NOT NULL DEFAULT '' COMMENT '服务名称',
  `deploy_type` char(1) NOT NULL DEFAULT '0' COMMENT '部署类型 0:物理地址部署 1:marathon部署',
  `upsteam_node_name` varchar(64) NOT NULL DEFAULT '' COMMENT '节点名称',
  `selected_version` varchar(64) NOT NULL DEFAULT '' COMMENT '选中版本',
  `is_front` char(1) NOT NULL DEFAULT '0' COMMENT '是否是前端项目,0否1是',
  `is_ssl` char(1) NOT NULL DEFAULT '0' COMMENT '是否开启ssl',
  `listen_port` varchar(32) NOT NULL DEFAULT '' COMMENT '监听端口号',
  `ssl_listen_port` varchar(32) NOT NULL DEFAULT '' COMMENT 'ssl监听端口号',
  `server_host` varchar(64) NOT NULL DEFAULT '' COMMENT '域名',
  `ssl_crt_key_path` varchar(64) NOT NULL DEFAULT '' COMMENT '证书路径',
  `ssl_crt_path` varchar(64) NOT NULL DEFAULT '' COMMENT '证书key路径',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=26 DEFAULT CHARSET=utf8 COMMENT='服务信息';

/*Data for the table `t_server_info` */

insert  into `t_server_info`(`id`,`server_name`,`deploy_type`,`upsteam_node_name`,`selected_version`,`is_front`,`is_ssl`,`listen_port`,`ssl_listen_port`,`server_host`,`ssl_crt_key_path`,`ssl_crt_path`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (15,'USER-MNGT','1','user-mngt','dev_v.0.1.3','1','0','80','443','user-mngt.onebean.net','','','2019-06-16 08:06:41','2019-11-09 15:37:56',2,'0neBean','0'),(18,'API-ADAPTE','1','api-adapter','dev_v.0.9.6','0','0','','','','','','2019-11-04 15:44:25','2019-11-09 15:37:56',2,'0neBean','0'),(20,'SERVER-MNGT','1','server-mngt','dev_v.0.7.4','1','0','80','','server-mngt.onebean.net','','','2019-11-06 15:03:14','2019-11-09 15:37:56',2,'0neBean','0'),(21,'TENANT-MNGT','1','tenant-mngt','dev_v.0.1.1','1','0','80','','tenant-mngt.onebean.net','','','2019-11-06 15:04:10','2019-11-09 15:37:56',2,'0neBean','0'),(22,'MNGT-PORTAL','1','mngt-portal','dev_v.0.1.6','1','0','80','','mngt-portal.onebean.net','','','2019-11-06 15:04:39','2019-11-09 15:37:56',2,'0neBean','0'),(25,'MASSAGE-CENTER','1','message-center','dev_v.0.1.4','0','0','','','','','','2019-11-08 00:12:39','2019-11-09 15:37:56',2,'0neBean','0');

/*Table structure for table `t_server_machine_node` */

DROP TABLE IF EXISTS `t_server_machine_node`;

CREATE TABLE `t_server_machine_node` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `ip_address` varchar(64) NOT NULL DEFAULT '' COMMENT 'ip地址',
  `access_user` varchar(64) NOT NULL DEFAULT '' COMMENT '访问账户',
  `access_password` varchar(255) NOT NULL DEFAULT '' COMMENT '访问密码',
  `access_rsa_path` varchar(100) NOT NULL DEFAULT '' COMMENT '访问rsa_path',
  `access_port` varchar(32) NOT NULL DEFAULT '' COMMENT '访问端口号',
  `access_auth_type` char(1) NOT NULL DEFAULT '0' COMMENT '访问授权方式 (0: 密码模式,1:公私钥模式)',
  `server_machine_type` char(1) NOT NULL DEFAULT '0' COMMENT '服务节点类型,0：openresty，1：kubernetes-master',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;

/*Data for the table `t_server_machine_node` */

insert  into `t_server_machine_node`(`id`,`ip_address`,`access_user`,`access_password`,`access_rsa_path`,`access_port`,`access_auth_type`,`server_machine_type`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (2,'192.168.146.142','root','123456','','22','0','0','2019-10-22 23:48:46','2019-11-09 15:38:00',2,'0neBean','0'),(3,'192.168.146.143','root','123456','','22','0','1','2019-11-01 16:32:22','2019-11-09 15:38:00',2,'0neBean','0');

/*Table structure for table `t_un_login_access_api_white_list` */

DROP TABLE IF EXISTS `t_un_login_access_api_white_list`;

CREATE TABLE `t_un_login_access_api_white_list` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `api_id` int(11) NOT NULL DEFAULT '0' COMMENT '接口ID',
  `app_id` int(11) NOT NULL DEFAULT '0' COMMENT '应用ID',
  `api_name` varchar(64) NOT NULL DEFAULT '' COMMENT '应用名',
  `api_path` varchar(255) NOT NULL DEFAULT '' COMMENT 'API接口',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8 COMMENT='易开伙伴未登录访问接口白名单';

/*Data for the table `t_un_login_access_api_white_list` */

insert  into `t_un_login_access_api_white_list`(`id`,`api_id`,`app_id`,`api_name`,`api_path`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (8,168,27,'短信验证码登录','/auth/smsCodeLogin','2019-11-09 12:55:16','2019-11-09 15:38:03',2,'0neBean','0'),(9,167,27,'账号密码登录','/auth/passwordLogin','2019-11-09 12:55:21','2019-11-09 15:38:03',2,'0neBean','0'),(10,105,27,'发送登录短信验证码','/auth/sendLoginSms','2019-11-09 12:55:25','2019-11-09 15:38:03',2,'0neBean','0');

/*Table structure for table `t_upsteam_name` */

DROP TABLE IF EXISTS `t_upsteam_name`;

CREATE TABLE `t_upsteam_name` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `deploy_type` char(1) NOT NULL DEFAULT '0' COMMENT '部署类型 0:物理地址部署 1:kubernetes部署',
  `upsteam_name` varchar(64) NOT NULL DEFAULT '' COMMENT '节点名称',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=25 DEFAULT CHARSET=utf8;

/*Data for the table `t_upsteam_name` */

insert  into `t_upsteam_name`(`id`,`deploy_type`,`upsteam_name`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (17,'1','user-mngt','2019-06-16 08:05:48','2019-11-09 15:38:06',2,'0neBean','0'),(18,'1','server-mngt','2019-11-06 13:35:51','2019-11-09 15:38:06',2,'0neBean','0'),(20,'1','message-center','2019-11-06 13:42:11','2019-11-09 15:38:06',2,'0neBean','0'),(21,'1','mngt-portal','2019-11-06 13:42:37','2019-11-09 15:38:06',2,'0neBean','0'),(22,'1','tenant-mngt','2019-11-06 13:43:10','2019-11-09 15:38:06',2,'0neBean','0'),(24,'1','api-adapter','2019-11-06 14:57:47','2019-11-09 15:38:06',2,'0neBean','0');

/*Table structure for table `t_upsteam_node` */

DROP TABLE IF EXISTS `t_upsteam_node`;

CREATE TABLE `t_upsteam_node` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `node_name` varchar(64) NOT NULL DEFAULT '' COMMENT '节点名称',
  `node_namespace` varchar(64) NOT NULL DEFAULT '' COMMENT '节点命名空间',
  `selected_version` varchar(64) NOT NULL DEFAULT '' COMMENT '选中版本',
  `current_version` varchar(64) NOT NULL DEFAULT '' COMMENT '当前版本',
  `deploy_type` char(1) NOT NULL DEFAULT '0' COMMENT '部署类型 0:物理地址部署 1:kubernetes部署',
  `physical_path` varchar(255) NOT NULL DEFAULT '' COMMENT '物理地址',
  `running_status` char(1) NOT NULL DEFAULT '0' COMMENT '运行状态，0运行中，1已停止',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `deprecated_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '弃用时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=98 DEFAULT CHARSET=utf8;

/*Data for the table `t_upsteam_node` */

insert  into `t_upsteam_node`(`id`,`node_name`,`node_namespace`,`selected_version`,`current_version`,`deploy_type`,`physical_path`,`running_status`,`create_time`,`update_time`,`deprecated_time`,`operator_id`,`operator_name`,`is_deleted`) values (80,'api-adapter','onebean-func','dev_v.0.0.1','dev_v.0.0.1','1','192.168.146.143:31381','0','2019-11-07 18:41:05','2019-11-14 13:44:31','2019-11-07 18:41:05',2,'0neBean','0'),(94,'server-mngt','onebean-func','dev_v.0.0.1','dev_v.0.0.1','1','192.168.146.143:30032','0','2019-11-09 14:32:31','2019-11-14 13:44:33','2019-11-09 14:37:45',2,'0neBean','0');


/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;




/*
SQLyog Ultimate v12.09 (64 bit)
MySQL - 5.7.28 : Database - mgnesium_message_center
*********************************************************************
*/


/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`mgnesium_message_center` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */;

USE `mgnesium_message_center`;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;



/*
SQLyog Ultimate v12.09 (64 bit)
MySQL - 5.7.28 : Database - mgnesium_uag_account
*********************************************************************
*/


/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`mgnesium_uag_account` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */;

USE `mgnesium_uag_account`;

/*Table structure for table `uag_operator_log` */

DROP TABLE IF EXISTS `uag_operator_log`;

CREATE TABLE `uag_operator_log` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `app_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '项目名',
  `operator_description` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '用户名',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=429 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='操作日志';

/*Data for the table `uag_operator_log` */

insert  into `uag_operator_log`(`id`,`app_name`,`operator_description`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (425,'server-mngt','更新服务信息','2019-11-09 16:53:21','2019-11-09 16:53:21',2,'李文彬','0'),(426,'server-mngt','同步服务器节点信息','2019-11-09 16:53:21','2019-11-09 16:53:21',2,'李文彬','0'),(427,'server-mngt','同步服务器节点信息','2019-11-09 16:53:23','2019-11-09 16:53:23',2,'李文彬','0'),(428,'server-mngt','删除应用节点','2019-11-09 17:01:55','2019-11-09 17:01:55',2,'0neBean','0');

/*Table structure for table `uag_user_info_60917091710` */

DROP TABLE IF EXISTS `uag_user_info_60917091710`;

CREATE TABLE `uag_user_info_60917091710` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键,自增',
  `username` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '用户名',
  `password` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '密码',
  `is_lock` char(1) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '0' COMMENT '是否锁定,0否1是',
  `nick_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '昵称',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `operator_id` int(11) NOT NULL DEFAULT '0' COMMENT '操作人ID',
  `operator_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '操作人姓名',
  `is_deleted` char(1) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '0' COMMENT '逻辑删除,0否1是',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

/*Data for the table `uag_user_info_60917091710` */

insert  into `uag_user_info_60917091710`(`id`,`username`,`password`,`is_lock`,`nick_name`,`create_time`,`update_time`,`operator_id`,`operator_name`,`is_deleted`) values (2,'15377777777','f0353b34d88b0036c8632ab058081905a5510b02c528cd2d5c5c911666d6ae670e8731fc9e506c2d','0','0neBean','2019-10-23 00:17:05','2019-11-09 17:04:44',2,'0neBean','0');

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

```



---



# 4. 应用管理

## 4.1 应用管理

在 Magnesium 里，应用是一个消费者（consumer）的角色，用来以设定的`鉴权模式`消费平台上与自己绑定API。  

Magnesium 上应用大致分为两种类型，分别是：   

### * 基础应用
选择该类型后，应用实则为云控台的菜单，需要完善菜单的地址和图标信息，就可以在云控制台访问该应用。

### * 普通应用
则对应了平台上的API消费者，也是现实意义上我们开发的应用程序  

用创建后得到的应用ID和秘钥可以消费平台上已有的API，而不需要一味的为其重复开发。

<br/>

---

<br/>


在服务中心中创建管理一个应用的过程大致如下：

打开如下入口：

云控制台 >> 服务中心 >> 应用管理 >> 新建

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/55.png)
点击新建应用

---

如选择基础应用，即平台的菜单应用，则需要完善菜单的地址和图标信息
![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/26.png)

---

如选择普通应用，则需要选择授权模式。
![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/56.png)

**授权模式分为4种，其含义大致如下**

* 授权码 (共享令牌) ：  
oauth授权码模式的鉴权方式，所有的请求都要带有预先获取的accessToken来访问  

* 私有令牌  (一设备一令牌 需要登录) ：  

oauth授权码模式的鉴权方式，在获取授权码时需要带上设备码，从而获取到的一机一码，可以做到单设备登陆
所以在正常访问其他API之前需要访问登陆API来

* 私有免登陆令牌 (一设备一令牌 免登陆) ：  

该模式同私有令牌模式相仿，但无需登陆，直接在获取授权码的同时直接带上用户ID即可  
适用于介入平台，但是用户体系不在平台中的的项目

* IP白名单+通行令牌 (管理后台模式) ：  

通过固定的token和ip白名单两种方式校验来验证  


---

App的链接状态直接决定了，应用是否可访问API，而应用的状态更改后，需要在列表页面点击`同步`按钮后方才生效。  

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/25.png)





## 4.2 API对应用授权

在 Magnesium 上，一个应用是否有权限访问一个API，需要由以下步骤来关联访问权限：

<br/>

当创建完成一个应用后点击列表上的接口管理，通过弹出的模态框来模糊搜索 `服务名`，并选中其中的API利用穿梭框交互绑定,绑定完成后，需要在应用的列表页点击 `同步` 按钮方才生效。

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/28.png)

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/29.png)



## 4.3 添加IP白名单

`IP白名单+通行令牌 (管理后台模式)` 授权模式下需要设置应用自己的IP到IP白名单，才能通过鉴权正常访问API，具体步骤如下：  

打开 `云控台-应用中心-应用详情-添加IP访问白名单` ,键入应用自身部署的IP，点击保存即可，设置完成后需要在应用列表页面，点击 `同步` 按钮，方才生效。

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/34.png)

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/35.png)



## 4.4 私有令牌未登录访问API

通常有些业务场景要求我们在未登录的状态下来访问部分API，例如发送短信验证，和登录接口本身。
所以我们需要在私有令牌应用创建未登录访问API的白名单，具体步骤如下：


点击编辑应用，进入应用详情页，点击`添加免登录访问接口白名单`，  
按照 `接口名称` 来将已绑定的API添加到未登录访问白名单中,绑定完成后需要在应用列表页点击 `同步` 按钮方才生效。

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/30.png)



---



# 5. 服务管理

## 5.1 服务管理

Magnesium 里应用与服务的关系参考这里：[Magnesium 设计思想](https://github.com/0nebean/Magnesium/wiki)

在 Magnesium 中 服务 即对应着一组运行的web service，  
通过 `服务节点名称` 与一组有物理访问地址的 `服务节点` 相关联，  
一个服务其中又管理着多个API接口，平台上的消费者（consumer）应用，即是通过服务来关联API。



<br/>


在应用中心中创建管理一个应用的过程大致如下：

打开如下入口：

云控制台 >> 服务中心 >> 服务管理 >> 新建  中填写相关信息即可创建一个服务。
在这里可以选择性为服务设置域名，开启ssl支持，指定https证书，新增或保存时，数据会直接同步到网关，实时生效。

<br/>


![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/54.png)

<br/>

## 5.2 API管理

服务创建完成后，点击`接口管理`按钮即可在其中添加API接口。

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/42.png)

<br/>

这里的 `接口地址` 指的是API实际访问的URI，`代理地址`指的是对外暴露的实际地址，因为平台对外暴露API通过统一的域名，所以代理地址的意义在于避免API的实际访问地址重复，而在整个平台代理地址是不可重复的，页面上已做表单校验去重。

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/43.png)

创建完成服务或添加接口后，都需要在 `应用管理` 页面点击 `同步` 按钮，方才生效。



---



#  6. 服务器节点管理

我们在Magnesium中将服务器节点分为两种类型，分别是：

* ### openresty

* ### Kubernetes-master 

分别用于向网关同步应用信息，和从k8s获取devops信息。

---

Magnesium 提供了 `公私钥模式` 和 `密码模式` 来连接linux服务器。

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/40.png)

Magnesium 至少需要配置 openresty 和 Kubernetes-master 节点各一个才能正常同步应用信息到网关，并获取k8s的部署推送。



---



# 7. 服务节点管理



服务节点即对应了服务器上运行的服务实例，Magnesium 通过配置服务节点来转发服务上的API到真实的地址上。

创建编辑一个服务节点的过程大致如下：

在 `云控台-服务中心-服务节点管理-新增`

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/57.png)

可以看到，这里只支持物理部署的节点添加，k8s的节点将由k8s自己推送，不可手动编辑。

<br/>

服务节点的名称并不是直接输入的，而是搜索得来，通过输入框下方按钮来添加维护节点名称，因为这里的名称需要和服务关联。

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/36.png)
![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/37.png)  

<br/>

搜索输入节点名称，填写正确的节点访问地址，直接保存即可创建一个应用节点。  

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/39.png)

创建完成一个服务节点后，需要在服务列表页面，点击 `同步` 按钮后，方才生效。



---

# 8. 租户中心

## 8.1 部署租户中心指南

租户中心是用来给 [Sodium](http://sodium.onebean.net) SaaS系统下发账号所用,该功能是选装功能,如果你没有该需求可以忽略。
该功能要求你要部署`tenant-mngt`模块，并且部署了[Sodium](http://sodium.onebean.net)SaaS应用。


部署`tenant-mngt`模块步骤如下：

- 首先你需要初始化数据库 [tenant-mngt数据库初始化脚本](https://github.com/0nebean/Magnesium/wiki/%E7%A7%9F%E6%88%B7%E4%B8%AD%E5%BF%83%E6%95%B0%E6%8D%AE%E5%BA%93%E5%88%9D%E5%A7%8B%E5%8C%96%E8%84%9A%E6%9C%AC)


<br/>

---

<br/>


- 其次需要在配置中心提交配置文件

`application.properties`
```properties
server.port = 8080
server.context-path = 
bootstrap.without.jdbc.datasource = false
#spring
spring.http.converters.preferred-json-mapper = fastjson
spring.http.encoding.charset = UTF-8
spring.http.encoding.enabled = true
spring.http.encoding.force = true
spring.http.multipart.maxFileSize = 100Mb
spring.http.multipart.maxRequestSize = 1000Mb
simple.client.http.request.factory.read.timeout = 5000
simple.client.http.request.factory.connect.timeout = 15000

#logging
logging.level.root = info

```
(mysql schema:mgnesium_tenant)
[public-conf.jdbc](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.jdbc.properties)  
[public-conf.spring-cloud](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.spring-cloud.properties)  
[public-conf.rabbitmq](https://github.com/0nebean/public.conf/blob/master/conf/public-conf.rabbitmq.properties)


<br/>

---

<br/>

- 最后提交代码利用gitlab—runner构建项目，发布到k8s上。

提交git tag 'dev_v.x.x.x'


<br/>

---

<br/>



- 配置访问地址
将以下sql中的租户中心域名替换，执行sql，并执行 [中台启动访问指南](https://github.com/0nebean/Magnesium/wiki/%E4%B8%AD%E5%8F%B0%E5%90%AF%E5%8A%A8%E8%AE%BF%E9%97%AE%E6%8C%87%E5%8D%97) 中的步骤10，即可通过云控制台的地址访问租户中心。




```sql
UPDATE  `t_server_info` t SET t.server_host = 'tenant-mngt.onebean.net' WHERE t.upsteam_node_name = 'tenant-mngt';
UPDATE  `t_server_info` t SET t.pc_page_url = 'http://tenant-mngt.onebean.net' WHERE t.app_name = '租户中心';
```



## 8.2 租户管理

[`Sodium`](https://0nebean.github.io/Sodium/)是带有权限控制的SaaS后台管理系统,SaaS租户之间的数据沙箱完全隔离，用户，角色，菜单等数据根据租户分表独立存储,互不干扰，真因为如此，租户账号的开通下发并不是由Sodium自身完成。

<br/>

`Magnesium`提供了一套开通下发租户账号的功能如下：


通过 `云控台-租户中心-租户管理` 可以添加编辑租户信息:

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/48.png)

<br/>


![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/45.png)

Magnesium 维护了一套城市配置信息，若改配置中没有你需要的城乡县镇，你可以在 `云控台-租户中心-城市管理` 自行添加。

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/47.png)



## 8.3 租户账号信息下发



成功创建租户信息之后，需要下发账号操作，方可在`Silicon`系统中登录，从界面上可以看出下发租户账户，分为两步:


* ### 激活租户账号后，将在 [`Silicon`](https://0nebean.github.io/Silicon/) 创建该租户ID分表的用户，角色，权限表。  

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/44.png)

<br/>

* ### 同步激活账号操作，则会在 Silicon 的租户登录列表中同步上已激活的租户。

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/46.png)

---



# 9.  用户中心

## 9.1 用户管理

Magnesium 为用户鉴权提供了独立的用户体系，但并不臃肿,用户体系，按照应用分表，数据层面完全隔离，但并没有冗余过多的用户业务信息,通过 `云控台-用户中心-用户管理` 界面可以直观的根据不同的应用来查看管理用户信息，这里维护的用户信息，根据不同的应用独立分表，互不干扰，用户信息的数据可以通过账号鉴权的[API来注册](https://github.com/0nebean/Magnesium/wiki/API%E9%89%B4%E6%9D%83#%E7%A7%81%E6%9C%89%E4%BB%A4%E7%89%8C%E4%B8%80%E8%AE%BE%E5%A4%87%E4%B8%80%E4%BB%A4%E7%89%8C-%E9%9C%80%E8%A6%81%E7%99%BB%E5%BD%95-1)，也可以通过管理后台直接添加。

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/49.png)


同API一样，在用户中心管理后台创建了账号之后，Magnesium会以topic的消息方式广播账号信息，业务系统只需要将自己的queue以应用中心的appId做bindingKey 绑定TopicExchange uag.create.account.fanout.exchange即可消费用户注册的信息，在业务系统中保存用户信息。


```java
@Configuration
public class RabbitMqQueueDefined {


    @Bean
    public Queue driverlessCarInitCustomAccount() {
        return new Queue(MqQueueNameEnum.DRIVERLESS_CAR_INIT_CUSTOM_ACCOUNT.getName());
    }

    @Bean
    TopicExchange uagCreateAccountFanoutExchange() {
        return new TopicExchange(MqExchangeNameEnum.UAG_CREATE_ACCOUNT_FANOUT_EXCHANGE.getName());
    }

    @Bean
    Binding bindingUagCreateAccountFanoutExchange(Queue driverlessCarInitCustomAccount, TopicExchange uagCreateAccountFanoutExchange) {
        return BindingBuilder.bind(driverlessCarInitCustomAccount).to(uagCreateAccountFanoutExchange).with("65099577991");
    }

}

```


创建账号的消息体格式:

```java
public class CreateAccountMqReq {
    private String uagUserId;
    private String uagUsername;

    public CreateAccountMqReq() {
    }

    public CreateAccountMqReq(String uagUserId, String uagUsername) {
        this.uagUserId = uagUserId;
        this.uagUsername = uagUsername;
    }

    public String getUagUserId() {
        return this.uagUserId;
    }

    public void setUagUserId(String uagUserId) {
        this.uagUserId = uagUserId;
    }

    public String getUagUsername() {
        return this.uagUsername;
    }

    public void setUagUsername(String uagUsername) {
        this.uagUsername = uagUsername;
    }
}

```

## 9.2 用户操作日志记录	

Magnesium 云控台本身是面向开发的工具，采用了松散的组件结构，邀请的账号模式，并未对权限有着严格的控制，相应的，在 `云控台-用户中心-用户操作日志` 里可以看到 云控台登录用户所有操作信息，都被记录在案，可以在这里根据用户名检索查阅。

<br/>

![](https://github.com/0nebean/chemical-el/blob/master/src/main/resources/img/50.png)

---

# 10. 消息中心

## 10.1 短信验证码发送



无论是短信验证码模式的单点登录，还是API模式的短信验证码登录注册，都需要发送短信功能来辅助登录。

<br/>


Magnesium 在 `message-center` 模块中预留两个短信通道发送的API：


<br/>

```
@Service
public class SmsMessageSenderServiceImpl implements SmsMessageSenderService {

    private final static Logger logger = LoggerFactory.getLogger(SmsMessageSenderServiceImpl.class);


    @Autowired
    private PushedMessageRecordService pushedMessageRecordService;

    @Override
    public Boolean sendSmsMsgByChannel1(SendSmsMsgReq req) {
        logger.info("请实现 sendSmsMsgByChannel1 发送方法");
        pushedMessageRecordService.saveSendSmsCodeRecord(req);
        return true;
    }


    @Override
    public Boolean sendSmsMsgByChannel2(SendSmsMsgReq req) {
        logger.info("请实现 sendSmsMsgByChannel2 发送方法");
        pushedMessageRecordService.saveSendSmsCodeRecord(req);
        return true;
    }

}


```

<br/>

这里的发送方法需要你自己接入第三方服务实现。

<br/>

通过切换配置文件中的发送通道配置，即可改变短信的发送通道。
```
#send channel
msg.send.channel = 2
```