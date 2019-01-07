### Hadoop安装注意事项
* 修改主机名（虚拟机）
```
vi /etc/hostname // 打开文件，更改主机名，并重启
vi /etc/hosts // 追加IP 主机名
```
* 关闭防火墙（虚拟机）
```
service iptables stop // 立即生效
chkconfig iptables off // 重启后生效
```
* 多次格式化namenode时，选N

### Hadoop远程调试注意事项
* ping虚拟机IP，在hosts文件中添加虚拟机IP（客户端）
* 安装Hadoop，并配置环境变量（客户端）
* 复制虚拟机中的core-site.xml、hdfs-site.xml、log4j.properties三个文件
* 将core-site.xml中的主机名称改为虚拟机的IP地址
* 下载windows环境下所需文件：winutils.exe
* 获取写操作权限（在环境变量中添加HADOOP_USER_NAME=虚拟机用户名或将当前系统的帐号修改为虚拟机用户名）
