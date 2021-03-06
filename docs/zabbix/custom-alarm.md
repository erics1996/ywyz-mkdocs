![Zabbix](../imgs/zabbix/202011061125.png)
#### Zabbix自定义报警
<hr>

##### 1. 自定义监控项
在 <font>监控主机</font> 上配置自定义监控项：
```python
# 查看当前登录用户
[root@agent ~]$ who
root     pts/0        2020-05-20 21:29 (10.0.0.1)
root     pts/1        2020-05-20 21:35 (10.0.0.1)
# 查看当前用户登录数
[root@agent ~]$ who|wc -l
2
# 设置自定义监控项login_users，命令是who|wc -l
[root@agent ~]$ vim /etc/zabbix/zabbix_agentd.conf 
UserParameter=login_users,who|wc -l
# 重启zabbix监控主机
[root@agent ~]$ systemctl restart zabbix-agent
```
在zabbix server机器上获取设置的监控项的值：
```python
# 下载zabbix-get.x86_64
[root@Zabbix-server ~]$ yum install zabbix-get.x86_64 -y
# 获取监控项的值
[root@Zabbix-server ~]$ zabbix_get -s 10.0.0.3 -k login_users
```
添加监控项，可以把 <font>历史数据保留时长</font> 设置的小一些：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521112123313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

可以看到我们自定义的监控项：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521112123384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 2. 创建触发器
这里要注意表达式部分监控项的名字要使用我们自定义的：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521112516368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 3. 自定义邮件发件人
这里需要注意的是163邮箱需要开启SMTP服务，才能拿到授权码：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521120428211.png)

授权码写在密码部分：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521115300242.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 4. 自定义邮件收件人
点击用户头像，到 <font>用户基本资料</font> 下的 <font>报警媒介</font> 下添加 Email 类型的报警媒介：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521114137159.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 5. 启用触发器的动作
到 <font>配置</font> 选项下的 <font>动作</font> 项中将状态修改为已启用的状态：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521114522775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 6. 触发报警
当触发“当前系统用户的登录数”触发器的时候，会向接收邮箱发送消息。这里可以查看已经发送的报警邮件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521120038386.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

邮箱也会接收到报警消息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521120124541.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 7. 自定义邮件报警信息
默认的邮件报警信息除了宏部分，其它均为英文且比较乱。所以，为了方便查看，需要自定义邮件报警信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521120847989.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

自定义邮件报警信息，我这里使用的模板是：
```python
服务器: {HOST.NAME}发生：{EVENT.NAME}故障！
{
告警主机： {HOST.NAME}
告警地址：{HOST.IP}
监控项目：{ITEM.NAME}
监控取值：{ITEM.LASTVALUE}
告警等级：{TRIGGER.SEVERITY}
当前状态：{TRIGGER.STATUS}
告警信息：{TRIGGER.NAME}
告警时间：{EVENT.DATE} {EVENT.TIME}
事件ID：{EVENT.ID}
}
```
下面换成我们定义的信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521121917798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

当触发报警后，可以在 **`动作日志`** 中查看到发送给接收邮件的报警信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521122352496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

检查邮箱后发现邮箱也收到了对应的报警信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521122430240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)
<hr>

##### 8. 自定义微信报警
写一个微信发送消息的脚本，这里需要用到企业微信。需要修改企业微信ID、机器人密码和应用ID，这些在企业微信管理都可以看到。下面是微信发送消息的脚本：
```python
# 导入相关模块
import requests, os, sys, logging, json

# 定义日志的格式
logging.basicConfig(
    filename=os.path.join('/tmp', 'weixin.log'),
    format='%(asctime)s - %(name)s - %(levelname)s - %(module)s：%(message)s',
    datefmt='%Y-%m-%d %H:%M:%S',
    level=logging.DEBUG,
)
# 企业微信id
corpid = ''
# 机器人密码
appsecret = ''
# 应用id
agentid = 1
# 获取accesstoken
token_url = 'https://qyapi.weixin.qq.com/cgi-bin/gettonken?corpid' + corpid + '&corpsecret=' + appsecret
req = requests.get(token_url)
accesstoken = req.json()['access_token']
# 发送消息
send_msg_url = 'https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=' + accesstoken
touser = sys.argv[1]
subject = sys.argv[2]
message = subject + '\n\n' + sys.argv[3]
params = {
    'touser': touser,
    'msgtype': 'text',
    'agentid': agentid,
    'text': {
        'content': message
    },
    'safe': 0
}
req = requests.post(send_msg_url, data=json.dumps(params))
# 把结果记录的日志中
logging.info('sendto:' + touser + ';;subject:' + subject + ';;message:' + message)
```
把脚本放到 zabbix server 中的指定的目录下：
```python
# 查看脚本应该放到哪个位置
[root@Zabbix-server ~]$ grep -Ev '^$|#' /etc/zabbix/zabbix_server.conf 
...
AlertScriptsPath=/usr/lib/zabbix/alertscripts
...
```
添加微信报警媒介：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521211303929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

指定收件人：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521211828247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

添加完成后启用微信报警媒介，可以暂时把Email报警媒介停用：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052121194134.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoYW5sb24=,size_16,color_FFFFFF,t_70)

下面执行这个脚本开始向微信发送报警信息：
```python
[root@Zabbix-server ~]$ cd /usr/lib/zabbix/alertscripts/
[root@Zabbix-server alertscripts]$ python weixin.py 用户名 标题 报警内容 
```