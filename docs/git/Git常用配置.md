## [Git 知识大全](https://gitee.com/help/categories/43)

### 配置Git仓用户和邮箱
配置全局用户及邮箱的命令分别为:
`git config --golbal user.name "cc"`
`git config --golbal user.email "email"`
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

###  SSH生成密钥
```bash
ssh-keygen -t rsa -C 登录邮箱
ssh-keygen -t rsa -C 1831814662@qq.com

```
### 查看公钥
```bash
cat ~/.ssh/id_rsa.pub
```
### 查看配置是否正确
```bash
 ssh -T git@github.com
 ```
 