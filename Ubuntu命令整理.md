#### 用户命令
-- 将当前用户加入到docker用户组
sudo gpasswd -r ${USER} docker
--从docker用户组删除当前用户
sudo gpasswd -d ${USER} docker

#### TCP/IP命令
-- 查看TCP连接情况，加c参数可以持续输出
netstat -nat|grep 10.1.1.9
