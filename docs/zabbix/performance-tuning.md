![Zabbix](../imgs/zabbix/202011061125.png)
#### Zabbix性能调优
<hr>

##### 1. 数据库调优
( 1 ) Zabbix是写多读少的业务，所以要提高Zabbix写入的性能。MyISAM 引擎时不可以使用的，至少要使用 InnoDB 引擎 ( mysql 5.5 )，或者使用 TokuDB 引擎 ( mysql 5.7 )，当然也可以把数据库的硬盘升级为SSD硬盘。

( 2 ) 去掉没有用的监控项，增加监控项的取值间隔，减少历史数据保存周期

( 3 ) 针对Zabbix历史数据和趋势图的表进行周期性分表

( 4 ) 把被动模式修改为主动模式，增加 zabbix-proxy 将监控项的采集数据集中写入，而不是每个 Zabbix Agent 都向最终的 Zabbix Server 发数据
<hr>

##### 2. 进程调优
( 1 ) 针对Zabbix-Server进程调优，进程忙碌的时候就增加进程数量

将扫描IP网段的频率时值为10s，这样会增加自动发现进程的压力：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602214041801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602214049405.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

Zabbix的进程监控都在 <font>图形</font> 里面，可以选择查看数据采集进程的情况，如自动发现进程。这里不难发现，自动发现进程的已经到了100%：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602215731375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

下面通过调优来降低自动发现忙碌程度：
```python
# 这里把自动发现进程数设置为6个(0~250)
[root@Zabbix-server ~]$ vim /etc/zabbix/zabbix_server.conf 
StartDiscoverers=6

# 重启Zabbix Server
[root@Zabbix-server ~]$ systemctl start zabbix-server.service
```
不建议上来就把值设置为最大，因为 <font>每增加一个进程需要消耗一定内存空间的</font>。然后再看自动发现的忙碌程度就降低了非常多：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602220910367.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

调优的原则是：哪个进程比较忙就增加进程数量，不建议加到几十倍，最好要一点点去调。<font>要保证曲线平稳，而不是高低起伏大</font>。

( 2 ) 针对Zabbix-Server缓存调优，剩余内存少的时候增加缓存值

缓存的作用是用来存储监控的主机，<font>这些目标主机数据是从数据库查出来的然后放到内存中的</font>。 当然，<font>取值的监控项</font> 和 <font>触发器指标</font> 也都是存在 Zabbix Server 的缓存中。如果缓存不够用了，Zabbix Server 就会出问题。Zabbix Server 默认的缓存是 8M。为了测试缓存调优效果，这里先把缓存调为最低，
```python
# 修改缓存为128K(128K-8G)
[root@Zabbix-server ~]$ vim /etc/zabbix/zabbix_server.conf 
CacheSize=128K

# 重启Zabbix Server
[root@Zabbix-server ~]$ systemctl restart zabbix-server.service
```
这个时候发现，Zabbix Server 挂掉：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602222212311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
```python
# 查看Zabbix Server日志
[root@Zabbix-server ~]$ tailf /var/log/zabbix/zabbix_server.log
 67257:20200602:102244.619 __mem_malloc: skipped 1 asked 1816 skip_min 1104 skip_max 1104
 67257:20200602:102244.619 [file:dbconfig.c,line:94] __zbx_mem_realloc(): out of memory (requested 1816 bytes)
 67257:20200602:102244.619 [file:dbconfig.c,line:94] __zbx_mem_realloc(): please increase CacheSize configuration parameter
```
意思也就是内存不足( out of memory )，让我们增加 CacheSize 配置参数。如果以后有这样的报警，需要把缓存调整大一些。除了 <font>内部进程、采集进程</font> 还有缓存使用率：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602222925219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

大于75%的时候会报警，如果缓存不够就需要增加缓存！