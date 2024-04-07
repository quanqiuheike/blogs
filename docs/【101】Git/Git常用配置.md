## [Git 知识大全](https://gitee.com/help/categories/43)

### 配置Git仓用户和邮箱
配置全局用户及邮箱的命令分别为:
`git config --global user.name "cc"`
`git config --global user.email "email"`
为单一的仓库配置用户名和邮箱的命令分别为：
`git config user.name "username"`
`git config user.email "email"。`

### 查看
```
user：git config --global user.name
email：git config --global user.email
查看单一仓库的用户名和邮箱的命令分别为：
user：git config user.name
email：git config user.email

```
---

### 查看当前的 remotegit remote -v 
```bash
origin  https://github.com/yourUserName/ProjectName.git (fetch)
origin  https://github.com/yourUserName/ProjectName.git (push)
origin  git@gitee.com:cqx.hacker/qq.git (fetch)
origin  git@gitee.com:cqx.hacker/qq.git (push)
```
### 切换 http 到 ssh
```bash
git remote set-url origin git@github.com:yourUserName/ProjectName.git
git remote set-url origin git@gitee.com:cqx.hacker/qq.git
```
### 从ssh切换至https 
```bash
git remote set-url origin(远程仓库名称) https://email/username/ProjectName.git 
```
---
#### SSH生成密钥新版本更新
[SSH生成密钥官方文档参考:于2022年更改模式](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
> note:GitHub improved security by dropping older, insecure key types on March 15, 2022.
* 步骤1: Open Git Bash.
* 步骤2 :

`ssh-keygen -t ed25519 -C "your_email@example.com"`
`ssh-keygen -t ed25519 -C "quanqiuhaike@gmail.com"`
ssh-keygen -t ed25519 -f ~/.ssh/github cuowu
Saving key "ssh-keygen -t ed25519 -f ~/.ssh/github" failed: No such file or directory
`ssh-keygen -t ed25519 -C "quanqiuhaike@gmail.com" -f "github_ed25519"`生成的目录未知
> Note: If you are using a legacy system that doesn't support the Ed25519 algorithm, use:

 `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
* 步骤三：
 根据提示，passphrase可以随意输入或者不输入直接enter，两次确认
* 步骤4：
根据github中的setting找到ssh添加即可
`C:\Users\用户名\.ssh\id_ed25519.pub`找到该文件打开复制添加
 * 测试是否连接成功
 `ssh -T git@github.com`
 ```
 $  ssh -T git@github.com
Hi quanxxx! You've successfully authenticated, but GitHub does not provide shell access.

 ```
 * 首次进行远程提交的时候可能遇到
 ```
 The authenticity of host 'github.com (20.205.243.166)' can't be established.
ECDSA key fingerprint is SHA256:p2QA......M.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,20.205.243.166' (ECDSA) to the list of known hosts.
Everything up-to-date
 ```

* 可以通过`ping github.com`确认ip是否为github的官网

```java
 C:\Users\cxa>ping github.com

正在 Ping github.com [20.205.243.166] 具有 32 字节的数据:

来自 20.205.243.166 的回复: 字节=32 时间=82ms TTL=110

来自 20.205.243.166 的回复: 字节=32 时间=86ms TTL=110

来自 20.205.243.166 的回复: 字节=32 时间=82ms TTL=110

来自 20.205.243.166 的回复: 字节=32 时间=82ms TTL=110

20.205.243.166 的 Ping 统计信息:

​    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，

往返行程的估计时间(以毫秒为单位):

​    最短 = 82ms，最长 = 86ms，平均 = 83ms
```


---

###  SSH生成密钥(以下为老版本的使用)
```bash
ssh-keygen -t rsa -C 登录邮箱
ssh-keygen -t rsa -C 1831814662@qq.com

```
### 查看公钥
```bash
cat ~/.ssh/id_rsa.pub
```
### 打开公钥所在文件夹

```bash
open ~/.ssh
```

找到并打开`id_rsa.pub`文件，全选其中的内容粘贴到网页的Key中。

### 查看配置是否正确

```bash
 ssh -T git@github.com
```

