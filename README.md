# graphdatabase
图数据库的笔记之

# Neo4j图数据库初识
## 目录
1. 什么是图数据库
2. 为什么使用图数据库
3. Neo4j的下载安装
4. Cypher查询语言
5. Neo4j的各类API 
6. 事务
7. Neo4j数据建模
8. 大规模数据导入neo4j

## 一.什么是图数据库
  - 关键词：存储图结构数据，NoSQL<br>
  - Neo4j的基本要素(构造单元)：结点，关系，属性<br>
![image](https://github.com/Sevenkili/raphdatabase/raw/master/image/1.png)<br>
![image](https://github.com/Sevenkili/raphdatabase/raw/master/image/2.png)<br>


## 二.为什么使用图数据库
最大优势：查询的高性能<br>
举例说明：<br>
RDBMS-MySQL VS. Graph DB-Neo4j<br>
在关系型数据库中的社交网络关系数据存储<br>
![image](https://github.com/Sevenkili/raphdatabase/raw/master/image/3.png)<br>
![image](https://github.com/Sevenkili/raphdatabase/raw/master/image/4.png)<br>

查询某个用户的朋友的朋友：使用一次inner join操作
比如查询2的朋友的朋友：<br>
```
select * from t_user_friend uf1 inner join t_user_friend uf2 on uf1.user_1 = uf2.user_2 where uf1.user_2 = 2;
```
![image](https://github.com/Sevenkili/raphdatabase/raw/master/image/5.png)<br>
一般要做去重操作。可以得知2的朋友1的朋友是4。<br>
如果要查询某个用户的朋友的朋友的朋友，即3度关系<br>
```
select distinct * from t_user_friend uf1 inner join t_user_friend uf2 on uf1.user_1 = uf2.user_2
inner join t_user_friend uf3 on uf2.user_1 = uf3.user_2  
where uf1.user_2 = 2;
```
![image](https://github.com/Sevenkili/raphdatabase/raw/master/image/6.png)<br>
也就是说，在大量复杂的数据条件下，想要查询更深的关系，必须要使用更多的join操作，而大量的
join操作严重影响关系型数据库的性能。<br>

图数据库正好擅长多对多的关系，不用表，列，外键，直接将用户存储为结点，以用户之间的关系来
组织这些结点，查询的时候直接使用图遍历的算法，非常高效。 <br>

## 三.Neo4j的下载安装
下载地址：<br>
https://neo4j.com/download/community-edition/ <br>
![image](https://github.com/Sevenkili/raphdatabase/raw/master/image/7.png)<br>
个人通常使用社区版，安装Neo4j前必须要安装java虚拟机，Neo4j安装有两种方式：<br>
- 压缩版安装（.tar,.zip）
- 安装版安装（dmg,exe）推荐

Neo4j支持本地访问和远程访问：<br>
- 远程连接有三种方式：
  - Bolt连接方式
  - HTTP连接方式
  - HTTPS连接方式（ HTTPS需要有安全证书支持，让数据库开放在外网中访问时使用）

使用时修改配置文件neo4j.conf，网上有教程，官网上也有。<br>
安装后启动服务器，然后我们就可以通过Web控制台来访问服务器了。<br>
http://localhost:7474<br>
http://服务器IP:7474<br>
快速体验：在Web控制台中使用Cypher查询语言进行体验<br>

## 四.Cypher查询语言
关系型数据库拥有SQL查询语言， Structured Query Language。<br>
Neo4j拥有CQL，即Cypher Query Language，关键字借鉴了SQL，SPARQL和python等语言。<br>

Cypher可以在哪些地方使用？<br>
- neo4j-shell
- Web控制台
- Neo4j API
- Rest API

Cypher语法包括哪些内容？<br>
- 读写
- 索引
- 约束
- 标签
- 只读查询
- case子句
- 遍历路径
- 函数
- CALL调用存储过程

https://neo4j.com/docs/cypher-refcard/current/<br>
https://neo4j.com/docs/developer-manual/3.2/cypher/<br>

## 五.Neo4j的各类API 

Neo4j的两种使用方式：嵌入式模式，服务器模式<br>
- 嵌入式模式：
  - 只需要引用Neo4j的开发包。一般在maven项目的pom.xml文件中引入即可，不需要开启neo4j服务器。
  - Neo4j API
- 服务器模式：
  - 必须先安装和启动neo4j服务器，然后引入neo4j的驱动程序。同样在maven项目的pom.xml文件中引入即可。
  - REST API
- 遍历框架：
  - Traversal framework Java API
  https://neo4j.com/docs/java-reference/3.2/<br>

## 六.事务
使用neo4j API时，无论是什么操作都需要开启事务管理。也就是说操作数据库的程序都必须写在如下的代码段中：<br>
```
try( Transaction tx = graphDb.beginTx()){
……
tx.success();
}
```
使用完数据库后一定不要忘了关闭数据库。<br>

为了保持数据的完整性和保证良好的事务行为，Neo4j也支持ACID特性：<br>
- 原子性：一整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- 一致性：在事务开始之前和事务结束以后，数据库的完整性限制没有被破坏。 
- 隔离性：两个事务的执行是互不干扰的，一个事务不可能看到其他事务运行时，中间某一时刻的数据。
- 持久性：在事务完成以后，该事务对数据库所作的更改便持久地保存在数据库之中，并不会被回復。
- 除了上面的事务特性，Neo4j还有自己的特殊规定。

## 七.Neo4j数据建模
以不同方式，建模策略利用结点、关系、属性和标签建模。<br>
- 特定领域的简单的模型可以使用Neo4j-ogm进行构建
  http://blog.csdn.net/xqclll/article/details/60578982
- 使用 Java 核心 API

## 八.大规模数据导入neo4j
- Cypher中的LOAD CSV
- 官方提供的Java API —— Batch Inserter
- Batch Import 工具 https://my.oschina.net/u/2538940/blog/883829
- 官方提供的 neo4j-import 工具

http://blog.csdn.net/xingxiupaioxue/article/details/71747284
