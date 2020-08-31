<h2>B+ tree索引的使用</h2>
我们前边详细、详细又详细的唠叨了InnoDB存储引擎的B+树索引，我们必须熟悉下边这些结论：
+ 每个索引都对应一棵B+树，B+树分为好多层，最下边一层是叶子节点，其余的是内节点。所有用户记录都存储在B+树的叶子节点，所有目录项记录都存储在内节点。
+ InnoDB存储引擎会自动为主键（如果没有它会自动帮我们添加）建立聚簇索引，聚簇索引的叶子节点包含完整的用户记录。
+ 我们可以为自己感兴趣的列建立二级索引，二级索引的叶子节点包含的用户记录由索引列 + 主键组成，所以如果想通过二级索引来查找完整的用户记录的话，需要通过回表操作，也就是在通过二级索引找到主键值之后再到聚簇索引中查找完整的用户记录。
+ B+树中每层节点都是按照索引列值从小到大的顺序排序而组成了双向链表，而且每个页内的记录（不论是用户记录还是目录项记录）都是按照索引列的值从小到大的顺序而形成了一个单链表。如果是联合索引的话，则页面和记录先按照联合索引前边的列排序，如果该列值相同，再按照联合索引后边的列排序。
+ 通过索引查找记录是从B+树的根节点开始，一层一层向下搜索。由于每个页面都按照索引列的值建立了Page Directory（页目录），所以在这些页面中的查找非常快。

<h5>索引的代价</h5>
+ 空间上的代价
    这个是显而易见的，每建立一个索引都要为它建立一棵B+树，每一棵B+树的每一个节点都是一个数据页，一个页默认会占用16KB的存储空间，一棵很大的B+树由许多数据页组成，那可是很大的一片存储空间呢。


+ 时间上的代价
    每次对表中的数据进行增、删、改操作时，都需要去修改各个B+树索引。而且我们讲过，B+树每层节点都是按照索引列的值从小到大的顺序排序而组成了双向链表。不论是叶子节点中的记录，还是内节点中的记录（也就是不论是用户记录还是目录项记录）都是按照索引列的值从小到大的顺序而形成了一个单向链表。而增、删、改操作可能会对节点和记录的排序造成破坏，所以存储引擎需要额外的时间进行一些记录移位，页面分裂、页面回收啥的操作来维护好节点和记录的排序

<h5>B+树索引适用的条件</h5>
```sql
CREATE TABLE person_info(
    id INT NOT NULL auto_increment,
    name VARCHAR(100) NOT NULL,
    birthday DATE NOT NULL,
    phone_number CHAR(11) NOT NULL,
    country varchar(100) NOT NULL,
    PRIMARY KEY (id),
    KEY idx_name_birthday_phone_number (name, birthday, phone_number)
);
```

对于这个person_info表我们需要注意两点：
+ 表中的主键是id列，它存储一个自动递增的整数。所以InnoDB存储引擎会自动为id列建立聚簇索引。
+ 我们额外定义了一个二级索引idx_name_birthday_phone_number，它是由3个列组成的联合索引。所以在这个索引对应的B+树的叶子节点处存储的用户记录只保留name、birthday、phone_number这三个列的值以及主键id的值，并不会保存country列的值。

从这两点注意中我们可以再次看到，一个表中有多少索引就会建立多少棵B+树，person_info表会为聚簇索引和idx_name_birthday_phone_number索引建立2棵B+树。下边我们画一下索引idx_name_birthday_phone_number的示意图，不过既然我们已经掌握了InnoDB的B+树索引原理，那我们在画图的时候为了让图更加清晰，所以在省略一些不必要的部分，比如记录的额外信息，各页面的页号等等，其中内节点中目录项记录的页号信息我们用箭头来代替，在记录结构中只保留name、birthday、phone_number、id这四个列的真实数据值，所以示意图就长这样（留心的同学看出来了，这其实和《高性能MySQL》里举的例子的图差不多，我觉得这个例子特别好，所以就借鉴了一下）：

![name_birthday_phone的聚簇索引](https://tianxinmao.oss-cn-hangzhou.aliyuncs.com/study/index.jpg)

为了方便大家理解，我们特意标明了哪些是内节点，哪些是叶子节点。再次强调一下，内节点中存储的是目录项记录，叶子节点中存储的是用户记录（由于不是聚簇索引，所以用户记录是不完整的，缺少country列的值）。从图中可以看出，这个idx_name_birthday_phone_number索引对应的B+树中页面和记录的排序方式就是这样的：
+ 先按照name列的值进行排序。
+ 如果name列的值相同，则按照birthday列的值进行排序。
+ 如果birthday列的值也相同，则按照phone_number的值进行排序。

这个排序方式十分、特别、非常、巨、very very very重要，因为只要页面和记录是排好序的，我们就可以通过二分法来快速定位查找。下边的内容都仰仗这个图了，大家对照着图理解。

<h5>全值匹配</h5>
如果我们的搜索条件中的列和索引列一致的话，这种情况就称为全值匹配，比方说下边这个查找语句：
```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday = '1990-09-27' AND phone_number = '15123983239';
```
我们建立的idx_name_birthday_phone_number索引包含的3个列在这个查询语句中都展现出来了。大家可以想象一下这个查询过程：
+ 因为B+树的数据页和记录先是按照name列的值进行排序的，所以先可以很快定位name列的值是Ashburn的记录位置。
+ 在name列相同的记录里又是按照birthday列的值进行排序的，所以在name列的值是Ashburn的记录里又可以快速定位birthday列的值是'1990-09-27'的记录。
+ 如果很不幸，name和birthday列的值都是相同的，那记录是按照phone_number列的值排序的，所以联合索引中的三个列都可能被用到。

有的同学也许有个疑问，WHERE子句中的几个搜索条件的顺序对查询结果有啥影响么？也就是说如果我们调换name、birthday、phone_number这几个搜索列的顺序对查询的执行过程有影响么？比方说写成下边这样：
```sql
SELECT * FROM person_info WHERE birthday = '1990-09-27' AND phone_number = '15123983239' AND name = 'Ashburn';
```
答案是：没影响哈。MySQL有一个叫查询优化器的东东，会分析这些搜索条件并且按照可以使用的索引中列的顺序来决定先使用哪个搜索条件，后使用哪个搜索条件。我们后边儿会有专门的章节来介绍查询优化器，敬请期待。

<h5>匹配左边的列</h5>
其实在我们的搜索语句中也可以不用包含全部联合索引中的列，只包含左边的就行，比方说下边的查询语句：
```sql
SELECT * FROM person_info WHERE name = 'Ashburn';
```
或者包含多个左边的列也行：
```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday = '1990-09-27';
```
那为什么搜索条件中必须出现左边的列才可以使用到这个B+树索引呢？比如下边的语句就用不到这个B+树索引么？
```sql
SELECT * FROM person_info WHERE birthday = '1990-09-27';
```
是的，的确用不到，因为B+树的数据页和记录先是按照name列的值排序的，在name列的值相同的情况下才使用birthday列进行排序，也就是说name列的值不同的记录中birthday的值可能是无序的。而现在你跳过name列直接根据birthday的值去查找

但是需要特别注意的一点是，如果我们想使用联合索引中尽可能多的列，搜索条件中的各个列必须是联合索引中从最左边连续的列。比方说联合索引idx_name_birthday_phone_number中列的定义顺序是name、birthday、phone_number，如果我们的搜索条件中只有name和phone_number，而没有中间的birthday，比方说这样：
```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND phone_number = '15123983239';
```
这样只能用到name列的索引，birthday和phone_number的索引就用不上了，因为name值相同的记录先按照birthday的值进行排序，birthday值相同的记录才按照phone_number值进行排序。

<h5>匹配列前缀</h5>
我们前边说过为某个列建立索引的意思其实就是在对应的B+树的记录中使用该列的值进行排序，比方说person_info表上建立的联合索引idx_name_birthday_phone_number会先用name列的值进行排序，所以这个联合索引对应的B+树中的记录的name列的排列就是这样的：
```
Aaron
Aaron
...
Aaron
Asa
Ashburn
...
Ashburn
Baird
Barlow
...
Barlow
```
字符串排序的本质就是比较哪个字符串大一点儿，哪个字符串小一点，比较字符串大小就用到了该列的字符集和比较规则，这个我们前边儿唠叨过，就不多唠叨了。这里需要注意的是，一般的比较规则都是逐个比较字符的大小，也就是说我们比较两个字符串的大小的过程其实是这样的：
+ 先比较字符串的第一个字符，第一个字符小的那个字符串就比较小。
+ 如果两个字符串的第一个字符相同，那就再比较第二个字符，第二个字符比较小的那个字符串就比较小。
+ 如果两个字符串的第二个字符也相同，那就接着比较第三个字符，依此类推。

也就是说这些字符串的前n个字符，也就是前缀都是排好序的，所以对于字符串类型的索引列来说，我们只匹配它的前缀也是可以快速定位记录的，比方说我们想查询名字以'As'开头的记录，那就可以这么写查询语句：
```sql
SELECT * FROM person_info WHERE name LIKE 'As%';
```
但是需要注意的是，如果只给出后缀或者中间的某个字符串，比如这样：
```sql
SELECT * FROM person_info WHERE name LIKE '%As%';
```

<h5>匹配范围值</h5>
回头看我们idx_name_birthday_phone_number索引的B+树示意图，所有记录都是按照索引列的值从小到大的顺序排好序的，所以这极大的方便我们查找索引列的值在某个范围内的记录。比方说下边这个查询语句：
```sql
SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow';
```

由于B+树中的数据页和记录是先按name列排序的，所以我们上边的查询过程其实是这样的：
+ 找到name值为Asa的记录。
+ 找到name值为Barlow的记录。
+ 哦啦，由于所有记录都是由链表连起来的（记录之间用单链表，数据页之间用双链表），所以他们之间的记录都可以很容易的取出来喽～
+ 找到这些记录的主键值，再到聚簇索引中回表查找完整的记录。

不过在使用联合进行范围查找的时候需要注意，如果对多个列同时进行范围查找的话，只有对索引最左边的那个列进行范围查找的时候才能用到B+树索引，比方说这样：
```sql
SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow' AND birthday > '1980-01-01';
```
上边这个查询可以分成两个部分：
+ 通过条件name > 'Asa' AND name < 'Barlow'来对name进行范围，查找的结果可能有多条name值不同的记录
+ 对这些name值不同的记录继续通过birthday > '1980-01-01'条件继续过滤。

这样子对于联合索引idx_name_birthday_phone_number来说，只能用到name列的部分，而用不到birthday列的部分，因为只有name值相同的情况下才能用birthday列的值进行排序，而这个查询中通过name进行范围查找的记录中可能并不是按照birthday列进行排序的，所以在搜索条件中继续以birthday列进行查找时是用不到这个B+树索引的。

<h5>精确匹配某一列并范围匹配另外一列</h5>
对于同一个联合索引来说，虽然对多个列都进行范围查找时只能用到最左边那个索引列，但是如果左边的列是精确查找，则右边的列可以进行范围查找，比方说这样：
```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday > '1980-01-01' AND birthday < '2000-12-31' AND phone_number > '15100000000';
```
这个查询的条件可以分为3个部分：
+ name = 'Ashburn'，对name列进行精确查找，当然可以使用B+树索引了。
+ birthday > '1980-01-01' AND birthday < '2000-12-31'，由于name列是精确查找，所以通过name = 'Ashburn'条件查找后得到的结果的name值都是相同的，它们会再按照birthday的值进行排序。所以此时对birthday列进行范围查找是可以用到B+树索引的。
+ phone_number > '15100000000'，通过birthday的范围查找的记录的birthday的值可能不同，所以这个条件无法再利用B+树索引了，只能遍历上一步查询得到的记录。

同理，下边的查询也是可能用到这个idx_name_birthday_phone_number联合索引的：
```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday = '1980-01-01' AND phone_number > '15100000000';
```
<h5>用于排序</h5>
我们在写查询语句的时候经常需要对查询出来的记录通过ORDER BY子句按照某种规则进行排序。一般情况下，我们只能把记录都加载到内存中，再用一些排序算法，比如快速排序、归并排序、吧啦吧啦排序等等在内存中对这些记录进行排序，有的时候可能查询的结果集太大以至于不能在内存中进行排序的话，还可能暂时借助磁盘的空间来存放中间结果，排序操作完成后再把排好序的结果集返回到客户端。在MySQL中，把这种在内存中或者磁盘上进行排序的方式统称为文件排序（英文名：filesort），跟文件这个词儿一沾边儿，就显得这些排序操作非常慢了

但是如果ORDER BY子句里使用到了我们的索引列，就有可能省去在内存或文件中排序的步骤，比如下边这个简单的查询语句：
```sql
SELECT * FROM person_info ORDER BY name, birthday, phone_number LIMIT 10;
```

这个查询的结果集需要先按照name值排序，如果记录的name值相同，则需要按照birthday来排序，如果birthday的值相同，则需要按照phone_number排序。大家可以回过头去看我们建立的idx_name_birthday_phone_number索引的示意图，因为这个B+树索引本身就是按照上述规则排好序的，所以直接从索引中提取数据，然后进行回表操作取出该索引中不包含的列就好了。简单吧？是的，索引就是这么牛逼。

<h5>使用联合索引进行排序注意事项</h5>
对于联合索引有个问题需要注意，ORDER BY的子句后边的列的顺序也必须按照索引列的顺序给出，如果给出ORDER BY phone_number, birthday, name的顺序，那也是用不了B+树索引，这种颠倒顺序就不能使用索引的原因我们上边详细说过了，这就不赘述了。
同理，ORDER BY name、ORDER BY name, birthday这种匹配索引左边的列的形式可以使用部分的B+树索引。当联合索引左边列的值为常量，也可以使用后边的列进行排序，比如这样：
```sql
SELECT * FROM person_info WHERE name = 'A' ORDER BY birthday, phone_number LIMIT 10;
```
这个查询能使用联合索引进行排序是因为name列的值相同的记录是按照birthday, phone_number排序的

<h5>不可以使用索引进行排序的几种情况-ASC、DESC混用</h5>
对于使用联合索引进行排序的场景，我们要求各个排序列的排序顺序是一致的，也就是要么各个列都是ASC规则排序，要么都是DESC规则排序。


