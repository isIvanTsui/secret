<h1 align="center">连接虚拟机里的Redis</h1>

## 修改虚拟机网络连接方式为桥接模式

![image-20210830171944384](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210830171944384.png)

## 修改redis.conf

`bind` 改为`0.0.0.0`，设置所有机器可访问

## 查看6379端口是否开启

输入`firewall-cmd --query-port=6379/tcp`，如果返回no说明没有开启

输入`firewall-cmd --add-port=6379/tcp` （加` --permanent `永久有效），返回success说明开启成功

![img](https://raw.githubusercontent.com/isIvanTsui/img/master/7444195-8d6ddbbcb85738dd.png)

