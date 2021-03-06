![Zabbix](../imgs/zabbix/202011061125.png)
#### Zabbix自动监控
<hr>

##### 1. 自动化监控概述
自动化监控有两种方式，一种是自动发现，也就是根据 ip 一个个去扫描，但是 <font>效率是比较低的</font>。另外一种自动化监控的方式是 Zabbix 自带的 <font>自动注册</font>。下面将通过自动注册来完成自动化监控。
<hr>

##### 2. 配置自动注册的动作

设置 <font>动作</font> 和一系列动作相关的操作。<font>动作是自动注册的前提，不配置动作就不能完成自动注册，当把动作禁用后，那么就相当于把自动注册给关闭了。</font>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602005156782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020060200541889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602005641655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602010204999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

完成一个自动注册的动作后，可以直接克隆在这个动作基础上作修改进添加新的的动作。这里添加了两个动作，一个是监控web主机的动作，另一个是监控数据库主机的动作：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602010419865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602010601274.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602103143178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

动作添加完成后，可以先删除所有主机，等配置好 Zabbix Agent 的配置文件并重启后，监控界面就会显示被监控的主机。
<hr>

##### 3. 配置Zabbix Agent的配置文件
配置 Zabbix Agent 配置文件的目的是设置元数据，与在 Web 界面上设置的一致，用来唯一标识一台机器。先来配置 Zabbix Server 所在机器上的 Agent，测试动作是否设置成功。
```js
[root@Zabbix-server ~]$ cd /etc/zabbix/
[root@Zabbix-server zabbix]$ vim zabbix_agentd.conf
[root@Zabbix-server zabbix]$ grep -Ev '^$|#' zabbix_agentd.conf
...
Server=127.0.0.1			 # 所在的主机IP
ServerActive=127.0.0.1       # 与Server一样
Hostname=Zabbix server		 # 主机名（会在web界面显示）
HostMetadata=db    	     # 元数据
...
[root@Zabbix-server zabbix]$ systemctl restart zabbix-agent
```
重启之后，可以发现新增加了一台监控的主机（事先已经删除了所有监控主机）：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602094539745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

10.0.0.3的机器上也做同样的设置，只是需要修改Server必须是Zabbix Server所在的机器，也就是10.0.0.2，还有ServerActive=10.0.0.2。主机的名字可以自定义，元数据这里可以使用db。
```python
[root@agent zabbix]$ grep -Ev '^$|#' zabbix_agentd.conf 
...
Server=10.0.0.2
ServerActive=10.0.0.2
Hostname=10.0.0.3
HostMetadata=web
...
[root@agent zabbix]$ systemctl restart zabbix-agent
```
设置之后也会把这台主机监控起来：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602102354704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 4. 动作日志
动作执行后，由于设置了发送邮件，会向管理员发送邮件，在动作日志中可以查看动作的执行日志：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602114233568.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 5. 报警时间设置
这部分是补充内容，当要求某些项目在指定时间不报警，可以右两种方法。第一种方法是全局设置，不推荐：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602104200768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

第二种是针对监控项设置报警时间：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602104516754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

不可以在触发器里配置。