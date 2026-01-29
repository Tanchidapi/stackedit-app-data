### 常用命令
常见的数据类型有基础的string、list、set、sortedset、hash和高级的消息队列stream、地理空间geospatial、hyperloglog、位图bitmap、位域bitfield
redis中的数据都是以键值对的形式存储的，且大小写敏感，以二进制存储和显示，因此默认不支持中文，可以在启动cli时加上raw参数表示显示以原始形式显示内容，下面以string作为常见操作的说明
set key value 插入数据
get key  获取数据
del key 删除键
exists key 查看键是否存在
keys mode 查看所有匹配mode的键
flushall 删除所有键
ttl key 查看键过期时间，-2表示过期
expire key time 设置过期时间

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NzE0MTc3MjQsNjgzNzM3MzI5LC05ND
MwMzMwOTEsLTIwODg3NDY2MTJdfQ==
-->