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
基数是指集合中不重复的元素的个数
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MDI4NTA4NDksMTY3NzM1NTg3MywyMD
UyOTEyMDIwLC00OTYxMDM4NzgsLTExNjgzNzc4NDIsMTI3NzMx
NDY5NCw2OTQzOTcyOTQsLTU3NjYzNDIyOCwtODg2OTU0NTc5LD
E2MDY0NTY2NzMsLTEwMzIzNzY0NSw2ODM3MzczMjksLTk0MzAz
MzA5MSwtMjA4ODc0NjYxMl19
-->