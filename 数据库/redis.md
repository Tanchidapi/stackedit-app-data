### Redis和MYSQL的区别
MYSQL是关系型数据库，核心是数据的持久性存储和复杂查询；
Redis是内存键值数据库，核心是极速读写和数据结构服务
### 常用命令
常见的数据类型有基础的string、list、set、sortedset、hash和高级的消息队列stream、地理空间geospatial、hyperloglog、位图bitmap、位域bitfield
**redis中的数据都是以键值对的形式存储的**，且大小写敏感，以二进制存储和显示，因此默认不支持中文，可以在启动cli时加上raw参数表示显示以原始形式显示内容，下面以string作为常见操作的说明
set key value 插入数据
get key  获取数据
del key 删除键
exists key 查看键是否存在
keys mode 查看所有匹配mode的键
flushall 删除所有键
ttl key 查看键过期时间，-2表示过期
expire key time 设置过期时间
setex key value time 设置过期时间
setinx key value 如果键值对不存在则插入
### 列表list
类似于deque双端队列，可以在两端插入、删除数据，也可以删除指定范围外的数据，可以实现简单的消息队列，常用命令如下：
lpush name value 在name列表的左边插入value
rpush name value 同上，不过是在右边
lpop name 删除左边第一个元素
rpop name 同上，删除右边
上述两个pop命令可以在最后加上数字来一次性删除多个元素
lrange name startidx endidx 显示起始位置和结束位置之间的所有数据，endidx取-1时表示最后一个元素
llen name 显示列表长度
ltrim name start end 用于删除start到end以外的数据，只保留其之间的
### 集合set
不能重复且不像list是有顺序的，常用命令如下：
sadd name value 向名为name的集合中插入value
smembers 显示name集合中的所有数据
sismember name value 判断value是否在集合中
srem name value 删除集合中数据value
sinter 取交集
sunion 取并集
sdiff 取差集
### 有序集合sortedset
有序集合中的每个元素会关联一个浮点型的分数，按照分数对集合中的元素进行从小到大排序，元素唯一但是分数可重复，常用命令如下：
zadd name score value 向名为name的有序集合插入分数为score的value
zrange name start end （withscores） 显示有序集合中的元素（同时显示分数）
zscore name value 查看成员分数
zrank name value 查看成员排名
zrevrank name value 查看成员排名，从大到小
zrem name value 删除成员
### 哈希hash
字符类型的字段和值的映射表，存储键值对对象，常见命令如下：
hset name field value 向名为name的哈希中插入键值对
hget name field 获取field对应的值
hgetall 显示所有的键值对
hdel name field 删除对应键值对
hexists name field 判断是否存在某个键值对
hkeys/hlen 获取所有键/数量
### 发布订阅模式
通过publish命令将消息发布至指定的频道，通过subscribe来订阅对应频道，如：
在一个终端中用命令`subscribe erii`用于接收消息，另一个终端中使用`publish erii sakura`来发送消息到对应频道
订阅频道的终端可以有多个
发布订阅模式的局限性在于消息无法持久化，无法记录历史消息等，利用stream流可以解决上述问题
### 消息队列stream
是一个轻量级的消息队列，常见命令如下：
xadd name * key value 星号表示自动生成一个消息的id，该命令用于向name消息队列插入一条消息，内容为key value
xlen name 查看name中消息的数量
xrange name start end 查看name中消息的详细内容，start和end使用-和+可以表示所有消息
xdel name id 删除编号为id的消息
xtrim name maxlen 0 删除所有消息
id的格式是整数-整数，第一个整数表示一个时间戳，第二个整数表示一个序列号，无论手动还是自动，都要保证id递增
xread count num block time name start 一次读取num条消息，从start开始，如果没有消息就阻塞time（ms），start使用`$`符号表示获取从现在开始以后的最新消息。读取操作是可以重复进行的
xgroup creat name groupname id 创建一个消费者组，消息队列名称name，组名称groupname
xinfo groups name 查看name对应的消费者组的信息
xgroup createconsumer name groupname consumername 创建对应消息队列对应消费者组的消费者并指定名字
xreadgroup group groupname consumername count num block time streams name > 使用消费者组中的指定消费者读取num条消息，阻塞时间time，`>`表示从消息队列中读取最新消息
### 地理空间geospatial
用于存储地理位置空间并支持对地理位置进行各种运算，常用命令如下：
geoadd name longitude latitude locationname 在name地理空间中添加locationname的地理位置信息，经纬度为longitude和latitude
geopos name locationname 获取对应位置的地理位置信息
geodist name locationname1 locationname2 计算两个位置的直线距离，默认单位米
geosearch name frommember locationname byradius radiusnum 查找locationname为圆心，radiusnum为半径的园内的location，也可以使用bybox表示使用矩形范围
### 基数统计算法HyperLogLog
基数是指集合中不重复的元素的个数，hyperloglog的原理是使用随机算法来计算，通过牺牲一定精确度换取更小的内存消耗。优点是占用内存小，缺点是会有一定的误差，适用于对精确度要求不高且数据量很大的统计工作，如统计某个网站的uv或是某个词的搜索次数，常用命令如下：
pfadd name elements··· 添加元素elements到name中
pfcount name 计算name的基数
pfmerge resultname name1 name2 合并name1和name2到resultname中
### 位图bitmap
位图是string类型的扩展，可以使用一个string类型来模拟一个bit数组，数组下标就是偏移量，值只有0和1，支持位运算，常用命令如下：
setbit mapname offset bitvalue 设置mapname位图offset的位置的值为value
getbit mapname offset 获取mapname的offset位置的bit值
可以直接通过和string一样的set命令来一次性设置多个值
set mapname "value" 有个技巧是value可以使用十六进制来一次设置四位的值
bitcount mapname 统计mapname中有几个bit为1
bitpos mapname value start end 返回mapname中第一个出现value的位置，可以从指定的start到end的范围内查找
### 位域bitfield
位域可以将很多小的整数存储到一个较大的位图，以实现更高效的使用内存，常用命令如下：
bitfield idkey set type #position value 创建一个idkey的位域，并将position位置设置为值value
get idkey 用于查看内存中对应idkey的情况，会显示十六进制的编码
bitfield idkey get type #position 获取idkey的position位置的值
### 事务
即在一个请求中执行多条命令，通过multi和exec来开启并执行，在redis中，事务不保证所有的命令都执行成功，redis保证以下三点：1.所有命令在exec执行前都会被放入到一个队列中缓存起来。2.收到exec命令后事务开始执行，任何一个命令执行失败不影响其他命令。3.事务执行过程中，其他客户端提交的命令请求不会被插入到事务的执行序列中
### 持久化
redis基于内存，持久化有两种方式：1.RDB（Redis Database）2.AOF（Append Only File）
RDB通过在指定时间间隔内将内存中的数据快照写入磁盘实现持久化，是某个时间点上数据的完整副本，更适合用于做备份，可以通过save文件修改配置或者手动save命令来写入

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjY1Njc4MTEsLTEyMTg5OTU3OTksLTQ1Mj
QyMTAzOSwxNjc3MzU1ODczLDIwNTI5MTIwMjAsLTQ5NjEwMzg3
OCwtMTE2ODM3Nzg0MiwxMjc3MzE0Njk0LDY5NDM5NzI5NCwtNT
c2NjM0MjI4LC04ODY5NTQ1NzksMTYwNjQ1NjY3MywtMTAzMjM3
NjQ1LDY4MzczNzMyOSwtOTQzMDMzMDkxLC0yMDg4NzQ2NjEyXX
0=
-->