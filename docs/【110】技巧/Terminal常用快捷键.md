## 文件定位

`cd`  进入到某个文件路径下
 `cd ~/Desktop`  进入桌面位置
 `cd /User/用户名/Desktop`   桌面的位置
 `pwd`   当前文件路径
 `cd ..` 返回上一级目录
 `find *.txt` 查找当前目录下所有的txt文件
 `cd -`　返回上一个访问的目录
 `cd ~`　返回root用户位置

## 文件操作

`mvdir dir1 dir2`     移动或重命名一个目录
 `ctrl+c` 终止
 `ls -l` 查看当前目录下的文件夹/文件  参数：-l 详细信息，-a 包括隐藏文件
 `mkdir` 新建文件夹
 `touch`  新建文件
 `cp ~/Desktop/folder/test.txt ~/Desktop`     把 test.txt 拷贝到桌面
 `rm`删除文件   `rmdir`删除文件夹
 `clear` 清除屏幕或窗口内容
 `file` 显示文件类型
 `rmdir ***`  删除目录（空目录）此删除不会出现在废纸篓里
 `rm -rf ***` 删除目录（非空或者空目录都可以删除）推荐使用 此删除不会出现在废纸篓里
 `unrar` 解压 rar         `unzip`  解压 zip
 `mv a.txt b.txt` 把名为a的txt文件重命名为b
 `cp test.txt test2.txt` 拷贝 拷贝一个test.txt文件并重新命名为test2.txt

### 关机

`sudo shutdown -h now`  立刻关机
 `sudo shutdown -h +10`  十分钟后关机
 `sudo shutdown -h 20:00`  晚上八点关机

### 重启

`sudo shutdown -r now`
 `history` 列出最近执行过的

### 系统

`ifconfig`查看本机 IP 等配置信息

> linux应用 里面的 Vi/Vim 基本操作:
>  vi/vim 分三种模式: 指令模式,编辑模式,选择模式. 只有在编辑模式下才能进行输入(不是绝对).
>  默认是"指令模式", 这个模式只支持vi/vim的指令



```ruby
1.  在默认的"指令模式"下按 i 进入编辑模式
2.  在非指令模式下按 ESC 返回指令模式
3.  在"指令模式"下输入:
    :w保存当前文件
    :q 退出编辑,如果文件为保存需要用强制模式
    :q!强制退出不保存修改
    :wq组合指令, 保存并退出
4.  在"指令模式"下移动:
    h左
    j下
    k上
    l右

# 翻页
Shift + f(front)下一页
Shift + b(back)上一页
```

# 权限相关

`sudo`提升当前操作权限
 `passwd [username]`修改用户密码, 一般用来激活root用户(管理员)
 `chown [target][user]`修改制定目标的拥有者
 `chmod 755 [command]`提高指定文件的执行权限

## Terminal操作技巧

`Ctrl + a`光标移动到行首
 `Ctrl + e`移动到行尾
 `Double Tab`可以列出代选命令/代选文件
 `Ctrl + c`强制推出当前操作
 `Command + K`清屏



### Mac常用技巧

##### 1、如何查看Mac的Java_home地址

```shell
// 进到lib目录
cd /usr/libexec

// 运行java_home即可
./java_home
```

##### 2、Mac如何查看git安装目录

```shell
// 输入命令即可查看到安装路径
which git
//举一反三
which java 
```

##### 3、Mac查看sshkey

```shell
// 列出所有ssh文件夹的内容
ls -a ~/.ssh

// 查看SSH的KEY值   退出vim编辑：先输入esc，再输入： 再输入wq
 vim ~/.ssh/id_rsa.pub 

//举一反三，打开任意文件夹下的文件列表
ls -a ~/任意文件名

```

##### 4、Mac编辑.bash_profile

```shell
// 输入
cd ~

// 创建.bash_profile
touch .bash_profile

// 编辑.bash_profile文件
open -e .bash_profile

// 更新刚配置的环境变量
source .bash_profile

// 查看是否生效
echo $PATH

```





























