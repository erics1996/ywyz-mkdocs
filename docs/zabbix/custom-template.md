![Zabbix](../imgs/zabbix/202011061125.png)
#### Zabbix自定义模板
<hr>

##### 1. 设置模板监控项
创建模板之前需要为模板创建监控项。这里以TCP连接的11种状态为监控项，首先需要找到这些监控项：
```python
# TCP连接的11种状态
[root@agent ~]$ man netstat
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601190842136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
```python
[root@agent ~]$ cd /etc/zabbix/zabbix_agentd.d/
# 把监控项目放到文件中便于批量添加到监控项配置文件中
[root@agent zabbix_agentd.d]$ vim a.txt
[root@agent zabbix_agentd.d]$ cat a.txt 
ESTABLISHED
SYN_SENT
SYN_RECV
FIN_WAIT1
FIN_WAIT2
TIME_WAIT
CLOSE
CLOSE_WAIT
LAST_ACK
LISTEN
CLOSING

# 创建监控项配置文件
[root@agent zabbix_agentd.d]$ vim zbx_tcp.conf
[root@agent zabbix_agentd.d]$ >zbx_tcp.conf

# 查看tcp连接的11种状态
[root@agent zabbix_agentd.d]$ netstat -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN     
tcp        0      0 10.0.0.3:10050          10.0.0.2:51796          TIME_WAIT  
tcp        0      0 10.0.0.3:10050          10.0.0.2:51716          TIME_WAIT  
tcp        0      0 10.0.0.3:10050          10.0.0.2:51806          TIME_WAIT  
tcp        0      0 10.0.0.3:10050          10.0.0.2:51680          TIME_WAIT  
tcp        0      0 10.0.0.3:10050          10.0.0.2:51782          TIME_WAIT  
......

# LISTEN状态的数目
[root@agent zabbix_agentd.d]$ netstat -ant|grep -c LISTEN
6
# TIME_WAIT状态的数目
[root@agent zabbix_agentd.d]$ netstat -ant|grep -c TIME_WAIT
68

# 批量为监控项配置文件添加内容
[root@agent zabbix_agentd.d]$ for n in `cat a.txt`;do echo "UserParameter=$n,netstat -ant|grep -c $n">>zbx_tcp.conf;done;

# 重启zabbix agent
[root@agent zabbix_agentd.d]$ systemctl restart zabbix-agent

# 重启zabbix server
[root@Zabbix-server ~]$ systemctl restart zabbix-server

# 获取agent上的监控项的值
[root@Zabbix-server ~]$ zabbix_get -s 10.0.0.3 -k LISTEN
6
[root@Zabbix-server ~]$ zabbix_get -s 10.0.0.3 -k TIME_WAIT
63
```
在 Web 界面上添加这些监控项：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601175136170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601175443541.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

使用克隆方便添加与之前类似的内容：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601175625686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601175745936.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

完成添加：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601180357283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 2. 创建模板
创建模板，可以属于最大的群组：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601181218585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 3. 监控项关联模板
批处理将监控项关联模板：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601181637355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601181536364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

><font>模板也是特殊的主机!!!</font>

<hr>

##### 4. 添加应用集
将所有监控项添加应用集：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601183803407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601183813302.png)
<hr>

##### 5. 监控主机关联模板
监控主机可以关联很多监控不同类型监控项的模板：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601184043600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

设置不支持的监控项恢复时间为30s：
![在这里插入图片描述](https://img-blog.csdnimg.cn/202006011847059.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

查看监控的最新数据：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020060118483893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 6. 导入模板注意事项
使用别人的模板，不止需要xml格式的模板，还需要模板对应的.conf结尾的监控项配置文件，以及取值脚本等。导入的模板如果和之前的重名，则需要修改名字。网上有很多模板，可以通过下面的链接获取：

( 1 ) [https://share.zabbix.com/](https://share.zabbix.com/)

( 2 ) [https://zabbix.org/wiki/Main_Page](https://zabbix.org/wiki/Main_Page)

( 3 ) [https://github.com/monitoringartist/zabbix-community-repos](https://github.com/monitoringartist/zabbix-community-repos)