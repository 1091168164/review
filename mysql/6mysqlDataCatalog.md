<h2>mysql的数据目录</h2>

<h5>如何确定MySQL中的数据目录</h5>
那说了半天，到底MySQL把数据都存到哪个路径下呢？其实数据目录对应着一个系统变量datadir，我们在使用客户端与服务器建立连接之后查看这个系统变量的值就可以了：

```sql
mysql> SHOW VARIABLES LIKE 'datadir';
+---------------+-----------------------+
| Variable_name | Value                 |
+---------------+-----------------------+
| datadir       | /usr/local/var/mysql/ |
+---------------+-----------------------+
1 row in set (0.00 sec)
```

