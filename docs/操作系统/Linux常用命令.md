# Linux 命令大全

### 系统帮助
#### man
格式化并显示帮助文档


- man 本身也是一种命令，在第 7 章 `man 7 man` 就可以查看章节详情
- `man -a passwd` 查看全所有的命令的文档



> 在docker下使用centos7容器来学习linux的时候，遇到了采用yum安装好的man-pages无法使用的问题，在网上多方寻找，终于在stackoverflow的回答中中找到了答案。



1. vim /etc/yum.conf 将 tsflags=nodocs 这一行删除或注释掉。
1. 重启docker中的centos7容器（不一定需要这一步）
1. 运行命令yum -y install man-pages安装，此时就会发现man命令可用了



#### help


- 内部命令 `help cd`
- 外部命令 `ls --help`



`type cd` 区分 cd 命令是内部还是外部


#### info


info 比 help 更加详细，作为 help 的补充


### 文件管理


#### 文件查看


- pwd 显示当前目录
- cd 打开目录
   - 绝对路径 `/etc/sysconfig/network-scripts`
   - 相对路径 `cd sysconfig`
   - 相对路径 ../
   - 回到上一个目录 `cd -`
   - 回到上级目录 `cd ..`
- ls
   - l 长格式显示文件
      - 文件类型 权限 文件个数 用户 用户组
      - `d`开头 文件夹
      - `-` 文件
   - a 显示隐藏文件
   - r 逆序显示
   - t 按照时间顺序显示
   - R 递归显示
   - lh 文件大小使用 单位 计算



```
lrwxrwxrwx.  1 root    root     7 Oct 15  2017 bin -> usr/bin
dr-xr-xr-x.  5 root    root  4096 Nov  3  2018 boot
```


命令可以组合使用比如逆序长格式显示 `ls -lr`
`ls /usr /home` 同时查看 usr 和 home 的目录


清屏 `clear` 或者快捷键 ctrl + l


`/` 根目录


`/root` root 用户的目录


#### 创建与删除目录


mkdir 创建目录


```bash
# 在根目录下创建目录 a
mkdir /a 
# 在当前目录下创建目录 a
mkdir  a 
# 一次性创建多级目录
mkdir -p /a/b/c/d 
 # 递归查看全部文件夹
ls -R
```


rmdir 删除目录 **只能删除非空目录**，通常我们都会使用 rm 命令代替


```java
`rm -r` 删除目录无论是否为空
`rm -rf` 删除非空目录不确认
```


#### 复制和移动目录


cp 复制文件


```
# 复制目录
cp - r 
# 显示进程
cp - v 
# 保留原有信息，时间不会变化
cp - p 
# p 的基础上实现了递归复制
cp - a
```


mv 移动和重命名


```
# 重命名 a 为 b
mv a b
```


#### 通配符的简单使用


`*` 匹配所有的字符


`?` 匹配一个字符


#### 文本的查看


`cat` 显示文本内容


`head` 查看文件开头，默认 10 行


`tail` 查看文件结尾
`tail -f` 实时更新


`wc -l` 统计文本内容，查看行数


#### 打包压缩


tar 打包 **注意 tar 命令不需要引导符**


```
tar cf /tmp/etc-backup.tar /erc 
# 打包并压缩，使用双扩展名的目的是为了告诉别人所用的压缩方式 gzip 压缩效率更高
tar czf /tmp/etc-backup.tar.gz /erc 
# 使用 bzip2 压缩比例更好
tar cjf /tmp/etc-backup.tar.bz2 /erc 

# 解包到 root 目录
tar xf /tmp/etc-backup.tar -C /root
# gzip 解压
tar zxf
# bzip2 解压
tar jxf
```


tips: tbz2 和 tgz 就是 tar.bz2 和 tar.gz 的缩写


### 文本编辑器 VI


- 插入模式
- 正常模式
- 命令模式
- 可视模式



#### 插入模式


- i：当前位置
- I: 这一行的开头
- a: 光标位置的后面
- A: 这一行的结尾
- o: 下一行
- O：上一行



#### 正常模式：


- hjkl 控制上下左右
- yy 复制整行
- 3yy 复制 3 行
- p 粘贴
- y$ 光标到本行结尾
- dd 剪切本行
- d$ 光标到本行结尾
- u 撤销
- ctrl r 重做
- x 删除单个字符
- r 替换单个字符
- G 跳转最后一行
- g 跳转第一行
- ^ 跳转开头
- $ 跳转结尾



#### 命令模式 :


- set nu 显示行号
- w 保存
- q 退出
- q! 不保存退出
- ！执行 linux 命令
- /x 查找 x 字符，n 下一个，N 上一个



替换


```
# 将 old 替换成 new，多次替换
%s/old/new/g 
# 2 到 5 行的替换  
2,5s/old/new
# 2 到 5 行的替换，多次替换
2,5s/old/new/g
```


vim /etc/vimrc 修改 配置文件永远保留


#### 可视模式 v


- v 字符
- V 行
- ctrl v 块



进入可视模式选中后进入插入模式 编辑或者 b 删除


### 用户，权限管理


```bash
# 插入用户
useadd 
# 查看用户信息
id root 
# 设置密码，不输入用户名，默认当前用户
passwd 
# 保留目录，删除用户
userdel 
# 完全删除用户，包含用户家目录
userdel -r 
# 修改用户组
usermod -g 
# 修改家目录
usermod -d  
# 修改生命周期
chage 
# 添加用户组
groupadd
# 删除用户组
groupdel
```


用户的创建


- home 目录下会创建一个与用户名的目录作为用户的家目录
- 如果不特别指明用户组，那么还会创建同名用户组



#### 用户配置文件


目录： `/etc/passwd`
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590117320341-55b7a74a-9735-4d13-8d3c-09785b8f5eb3.png#align=left&display=inline&height=887&margin=%5Bobject%20Object%5D&originHeight=887&originWidth=1335&size=0&status=done&style=none&width=1335)


```bash
# 用户user1 
# x是否需要密码验证
# 0 表示是 root 用戶，用户id 表示用户身份
# 用户组
# 用户家目录
# 用户登录成功后的命令解释器
```


用户密码相关信息：`etc/shadow`
加密保存信息，即便密码相同，加密后仍然不同


用户组信息：`/etc/group`


#### 权限


```bash
# 切换用户（不完全切换）
su 
# 完全切换，普通用户的切换需要密码
su - 
# 修改用户权限
visudo
# 以其他用户执行配置的权限
sudo
```


#### 文件权限


![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590117310750-2a3b4ef8-0293-4d4b-8f85-44bdbb0d0c1c.png#align=left&display=inline&height=152&margin=%5Bobject%20Object%5D&originHeight=152&originWidth=775&size=0&status=done&style=none&width=775)


```bash
# d：文件夹
# rwx：当前用户的权限
# r-x: 当前用户组的权限
# r-x：其他用户的权限
# 6:文件夹中包含文件数目
# root：所属用户
# root：所属用户组
# 4096: 文件夹大小
# Oct 15  2017:修改、创建时间
drwxr-xr-x.  6 root root     4096 Oct 15  2017 yum
```


- r = 4
- w = 2
- x = 1



0（没有权限）；4（读取权限）；5（4+1 | 读取+执行）；6（4+2 | 读取+写入）；7（4+2+1 | 读取+写入+执行）


`rwxr-xr-x` 就可以用 755 来表示

| 符号 | 文件类型 | r | w | x |
| --- | --- | --- | --- | --- |
| - | 普通文件 | 读 | 写 | 执行 |
| d | 目录文件 | 显示目录内文件名（x） | 修改目录内文件名（x） | 进入目录 |
| b | 块特殊文件：硬盘，块设备 |  |  |  |
| c | 字符特殊文件：终端，字符设备 |  |  |  |
| l | 符号链接：快捷方式 |  |  |  |
| f | 命名管道 |  |  |  |
| s | 套接字文件 |  |  |  |



chown：修改属主和属组


chmod：修改权限


- u:修改权限
- a:全部权限
- o:设置权限
