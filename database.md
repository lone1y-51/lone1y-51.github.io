- [[#mongo|mongo]]
	- [[#mongo#uri配置|uri配置]]
- [[#redis|redis]]
	- [[#redis#scan误区|scan误区]]
- [[#tendis|tendis]]
	- [[#tendis#连接tendis报错信息里没有节点ip，只有port|连接tendis报错信息里没有节点ip，只有port]]


# mongo

## uri配置

1. `readPreference`配置项最好配置为`secondaryPreferred`优先读从库(`primaryPreferred`优先读主库)。配置为`secondary`或者`primary`时，当从库或者主库挂掉，即便集群可用，也会导致业务不可用。

# redis

## scan误区

> 增量式迭代命令并不保证每次执行都返回某个给定数量的元素。
> 
> 
> 增量式命令甚至可能会返回零个元素， 但只要命令返回的游标不是 `0` ， 应用程序就不应该将迭代视作结束。
> 
> 不过命令返回的元素数量总是符合一定规则的， 在实际中：
> 
> - 对于一个大数据集来说， 增量式迭代命令每次最多可能会返回数十个元素；
> - 而对于一个足够小的数据集来说， 如果这个数据集的底层表示为编码数据结构（encoded data structure，适用于是小集合键、小哈希键和小有序集合键）， 那么增量迭代命令将在一次调用中返回数据集中的所有元素。
> 
> 最后， 用户可以通过增量式迭代命令提供的 `COUNT` 选项来指定每次迭代返回元素的最大值。
# tendis

## 连接tendis报错信息里没有节点ip，只有port
这个问题猜测是tendis的一个bug，集群的节点没有正式加入到集群中（哪怕是单节点也会出现），需要重新执行一下meet
`cluster meet {ip} {port}`
> meet操作是将节点加入到集群用的