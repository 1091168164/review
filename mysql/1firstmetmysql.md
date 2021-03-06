<h2>初识mysql<h2>

<h5>启动mysql客户端程序</h5>
`mysql -h主机名  -u用户名 -p密码`


```
h：表示服务器进程所在计算机的域名或者IP地址，如果服务器进程就运行在本机的话，可以省略这个参数，或者填localhost或者127.0.0.1。也可以写作 --host=主机名的形式
u：表示用户名。也可以写作 --user=用户名的形式
p：表示密码。也可以写作 --password=密码的形式
```

<h5>关闭mysql客户端程序</h5>

<ol>
<li>quit</li>
<li>exit</li>
<li>\q</li>
</ol>

------------

<h5>mysql存储引擎</h5>
截止到服务器程序完成了查询优化为止，还没有真正的去访问真实的数据表，MySQL服务器把数据的存储和提取操作都封装到了一个叫存储引擎的模块里。我们知道表是由一行一行的记录组成的，但这只是一个逻辑上的概念，物理上如何表示记录，怎么从表中读取数据，怎么把数据写入具体的物理存储器上，这都是存储引擎负责的事情。为了实现不同的功能，MySQL提供了各式各样的存储引擎，不同存储引擎管理的表具体的存储结构可能不同，采用的存取算法也可能不同。


<h6>常用存储引擎<h6>
```
ARCHIVE：用于数据存档（行被插入后不能在修改）
BLACKHOLE:丢弃写操作，读操作会返回空内容
CSV:在存储数据时，以逗号分隔各个数据项
FEDERATED:用来访问远程表
InnoDB:具备外键支持功能的事务存储引擎
MEMORY：置于内存的表
MERGE:用来管理多个myISAM表构成的表合集
MyISAM：主要的非事务处理存储引擎
NDB：Mysql集群专用存储引擎
```
----
<h5>关于存储引擎的一些操作</h5>
`show ENGINES;`

![show engines](https://tianxinmao.oss-cn-hangzhou.aliyuncs.com/WechatIMG3.png)


<h6>其中的Support列表示该存储引擎是否可用，DEFAULT值代表是当前服务器程序的默认存储引擎。Comment列是对存储引擎的一个描述，英文的，将就着看吧。Transactions列代表该存储引擎是否支持事务处理。XA列代表着该存储引擎是否支持分布式事务。Savepoints代表着该存储引擎是否支持部分事务回滚。</h6>
