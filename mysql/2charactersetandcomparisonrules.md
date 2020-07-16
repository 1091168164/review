<h2>启动项和配置文件<h2>
 ```
 禁用TCP/IP网络通信
`mysql --skip-networking`
`mysql --skip_networking`

结果：
 mysql -h127.0.0.1 -uroot -p
Enter password:

ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1' (61)
```

<h2>字符集和比较规则<h2>
<h5>字符集介绍</h5>
将一个字符映射成一个二进制数据的过程也叫做编码，讲一个二进制数映射到一个字符集的过程叫做解码。