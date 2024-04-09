##  当你需要在电脑上配置SSH连接到两个GitHub账号时，从头到尾的解决方案包括以下步骤：



##  Window下的多账户配置



###  步骤 1: 生成第一个SSH密钥对



1.**打开Git Bash：**



- 在Windows中，点击开始菜单，找到并打开Git Bash。



2. **生成SSH密钥对：**



- 运行以下命令来生成第一个SSH密钥对：

  ```
  ssh-keygen -t ed25519 -C "your_email@example.com"
  
  
  ssh-keygen -t ed25519 -C "quanqiuhaike@gmail.com"
  
  /c/Users/chengqiuxia/.ssh/id_ed25519_chengengineer
  
  
  ssh-keygen -t ed25519 -C "cqxengineer@gmail.com"
  
  /c/Users/chengqiuxia/.ssh/id_ed25519_quanqiuheike
  ```

  

>  将`your_email@example.com`替换为你的第一个GitHub账号注册邮箱。

- 输入该命令后会出现该提示

  ```
  Generating public/private ed25519 key pair.
  
  Enter file in which to save the key (/c/Users/xxxx/.ssh/id_ed25519): I:\ssh\githubkey
  
  可以再该提示后面输入想要的路径和密钥对名字，输入路径：比如 I:\ssh\github，其中githubkey是密钥对名字，ssh作为文件夹是需要创建好存在的，githubkey文件是自动创建
  
  格式比如：C:\foldername\keyname
  
  或者直接复制提示的格式更改文件名即可：/c/Users/chengqiuxia/.ssh/id_ed25519_quanqiuheike
  ```



- 完整案例可以参考为

  ```
  $ ssh-keygen -t ed25519 -C "quanqiuhaike@gmail.com"
  
  Generating public/private ed25519 key pair.
  
  Enter file in which to save the key (/c/Users/chengqiuxia/.ssh/id_ed25519): /c/Users/chengqiuxia/.ssh/id_ed25519_quanqiuheike
  ```

-  `/c/Users/chengqiuxia/.ssh/id_ed25519_chengengineer`

-  `/c/Users/chengqiuxia/.ssh/id_ed25519_quanqiuheike`

###  步骤 2: 添加第一个SSH密钥到GitHub账号

1. **复制公钥：**

- 在Git Bash中运行以下命令来复制第一个SSH密钥对的公钥内容：

  ```
  cat ~/.ssh/id_ed25519.pub
       
  cat ~/.ssh/id_ed25519.pub
  ```

-  复制输出的公钥内容，以备后用，cat在Git Bash中才能识别。

2. **添加公钥到GitHub账号：**

- 在浏览器中登录到你的第一个GitHub账号。

- 转到GitHub的"Settings" -> "SSH and GPG keys"页面。

- 点击"New SSH key"或者"Add SSH key"按钮。

- 在"Title"字段中输入一个描述性的名称，然后将你复制的公钥粘贴到"Key"字段中。

- 点击"Add SSH key"按钮完成添加。

###  步骤 3: 生成第二个SSH密钥对并指定路径

1. 生成第二个SSH密钥对：可参考1

- 运行以下命令来生成第二个SSH密钥对，并指定路径和文件名：

```
  ssh-keygen -t ed25519 -f /c/Users/YourUsername/.ssh/id_ed25519_second -C "your_second_email@example.com"

 ssh-keygen -t ed25519 -f /c/Users/chengqiuxia/.ssh/id_ed25519_cqxengineer -C "cqxengineer@gmail.com"
```

> 将`YourUsername`替换为你的Windows用户名，`your_second_email@example.com`替换为你的第二个GitHub账号注册邮箱。



### 步骤 4: 添加第二个SSH密钥到GitHub账号



1. **复制公钥：**

- 在Git Bash中运行以下命令来复制第二个SSH密钥对的公钥内容：

```
  cat /c/Users/chengqiuxia/.ssh/id_ed25519_second.pub

  cat /c/Users/chengqiuxia/.ssh/id_ed25519_chengengineer.pub
```

- 复制输出的公钥内容，以备后用。

2. **添加公钥到GitHub账号：**

- 在浏览器中登录到你的第二个GitHub账号。

- 转到GitHub的"Settings" -> "SSH and GPG keys"页面。

- 点击"New SSH key"或者"Add SSH key"按钮。

- 在"Title"字段中输入一个描述性的名称，然后将你复制的公钥粘贴到"Key"字段中。

- 点击"Add SSH key"按钮完成添加。



### 步骤 5: 配置SSH配置文件

1. **打开或创建SSH配置文件：**

- 运行以下命令打开SSH配置文件：

```
   code ~/.ssh/config
```

- 如果文件不存在，则会创建一个新文件。

2. **配置别名和连接参数：**

- 在SSH配置文件中添加以下内容用于配置第一个GitHub账号：

```
Host github.com
 	 HostName github.com
     User git
	 IdentityFile ~/.ssh/id_ed25519
	 
Host chengengineer
	HostName github.com
 	User chengengineer
	IdentityFile ~/.ssh/id_ed25519_chengengineer
```

- 添加以下内容用于配置第二个GitHub账号：

```
Host github-second
  	 HostName github.com
 	 User git
 	 IdentityFile ~/.ssh/id_ed25519_second

Host github-second
 	HostName github.com
  	User git
 	IdentityFile ~/.ssh/id_ed25519_second

Host quanqiuheike
 	HostName github.com
	User quanqiuheike
	IdentityFile ~/.ssh/id_ed25519_cqxengineer
```



###  步骤 6: 测试连接



**1.测试第一个GitHub账号：**

- 运行以下命令测试第一个GitHub账号的SSH连接：

- config中的Host对应的为别名，测试以Host为准

- Host github.com 更改为Host quanqiuheike时，链接测试也变为ssh -T quanqiuheike

- 否则权限问题：git@github.com: Permission denied (publickey).

```
ssh -T git@github.com

ssh -T git@chengengineer

ssh -T git@quanqiuheike
```



**2.测试第二个GitHub账号:**

- 运行以下命令测试第二个GitHub账号的SSH连接：

```
  ssh -T git@github-second

  ssh -T git@github-chengengineer

  ssh -T git@quanqiuheike
```



**3.确认连接：**

- 如果一切正常，你应该会收到一条消息，确认你已经成功连接到GitHub。如果提示你要确认连接，请输入"Yes"确认即可。



### 步骤7：PUSH推送提交异常经典问题

- 如果SSH正常添加到github，本地git@<github.com此处看是否为别名>

```
git@github.com: Permission denied (publickey).

fatal: Could not read from remote repository.
```

- 或者：

```
Git 最著名报错 “ERROR: Permission to XXX.git denied to user”解决方案

ERROR: Permission to quanqiuheike/blogs.git denied to chengengineer. 

fatal: Could not read from remote repository.Please make sure you have the correct access rights and the repository exists.
```

* 如果是新建的仓库，直接使用替换后的URL克隆即可。
* 如果已经使用原地址克隆过了，可以使用命令修改

- 需要根据在.ssh配置的别名，重新更改远程的URL

-  通过`git remote -v`获取远程ssh地址,将git@github.com改为git@别名或者别名

```
原本的为：git@github.com:quanqiuheike/blogs.git

需要将git@github.com改为git@quanqiuheike或者直接是quanqiuheike即可

git remote set-url origin git@quanqiuheike:quanqiuheike/blogs.git

git remote set-url origin quanqiuheike:quanqiuheike/blogs.git
```



#### :Clone、Pull、Push遇到的问题 

-  **clone** 

```
 git clone git@github.com:quanqiuheike/blogs.git

 git clone git@quanqiuheike:quanqiuheike/blogs.git

 git clone quanqiuheike:quanqiuheike/blogs.git
```

- **pull**

```
 git clone git@github.com:quanqiuheike/blogs.git

 git clone git@quanqiuheike:quanqiuheike/blogs.git

 git clone quanqiuheike:quanqiuheike/blogs.git 
```

- **push**

```
 git clone git@github.com:quanqiuheike/blogs.git

 git clone git@quanqiuheike:quanqiuheike/blogs.git

 git clone quanqiuheike:quanqiuheike/blogs.git
```

