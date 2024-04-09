##  Mac下的多账户配置

- 下面是在 Mac 下使用 ed25519 加密算法生成 SSH 密钥对，并配置两个 GitHub 账号的详细步骤。

### 步骤一：设置第一个 GitHub 账号ed25519加密方式

**1.生成 ed25519 类型的 SSH 密钥对:**

在终端中输入以下命令：

```
ssh-keygen -t ed25519 -C "your_email@example.com"

ssh-keygen -t ed25519 -C "quanqiuhaike@gmail.com"
```

**2、保存密钥对**：

按照提示输入文件保存的路径和名称，例如：`/Users/your_username/.ssh/id_ed25519_first`。这将创建一个名为 `id_ed25519_first` 的私钥文件和一个名为 `id_ed25519_first.pub` 的公钥文件。

**3、将公钥添加到 GitHub 账号：**

将 `id_ed25519_first.pub` 文件中的内容复制到 GitHub 账号的 SSH 设置中。

### 步骤二：设置第二个 GitHub 账号



**1、 生成 ed25519 类型的 SSH 密钥对：**

在终端中输入以下命令：

```
ssh-keygen -t ed25519 -C "your_email@example.com" -f /Users/your_username/.ssh/id_ed25519_second
```

**2、将公钥添加到 GitHub 账号：**

将 `id_ed25519_second.pub` 文件中的内容复制到第二个 GitHub 账号的 SSH 设置中。



###  步骤三：配置 SSH 主机别名



编辑 `~/.ssh/config` 文件以配置别名和对应的密钥。

**1、打开终端并输入以下命令：**

```
nano ~/.ssh/config
```

**2、 在打开的文件中添加以下内容：**

```
# 第一个 GitHub 账号**
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_first

# 第二个 GitHub 账号**

Host github-second
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_second
```



确保将 `~/.ssh/id_ed25519_first` 和 `~/.ssh/id_ed25519_second` 替换为实际的私钥文件路径。

**3、保存并关闭文件（在 Nano 编辑器中按下 `Ctrl + X`，然后按 `Y` 确认保存，最后按 `Enter` 退出）。**



### 步骤四：验证配置是否成功

- 测试第一个 GitHub 账号是否配置成功：

```
ssh -T git@github.com
```



- 测试第二个 GitHub 账号是否配置成功：

```
ssh -T git@github-second
```

以上步骤完成后，你的 Mac 电脑上已经配置了两个 GitHub 账号，并且每个账号都有自己的 ed25519 类型的 SSH 密钥对。

### 步骤五：PUSH、PULL、Clone代码权限问题

- 由于config设置了别名，连接测试的时候会更改别名，比如`ssh -T git@github.com`改为`ssh -T git@chengengineer`

- clone：`git clone git@github.com:quanqiuheike/blogs.git`

  ```
  `git clone git@github.com:quanqiuheike/blogs.git`
  
  `git clone git@quanqiuheike:quanqiuheike/blogs.git`
  
  `git clone quanqiuheike:quanqiuheike/blogs.git`
  ```

- push: 需要通过`git remote -v`获取远程仓库地址

  ```
  origin  git@quanqiuheike:quanqiuheike/blogs.git (fetch)
  
  origin  git@quanqiuheike:quanqiuheike/blogs.git (push)
  ```

  - 更改远程的url：`git remote set-url origin  ssh地址`

  ```
  git remote set-url origin git@chengengineer:chengengineer/blogs.git
  
  git remote set-url origin git@quanqiuheike:quanqiuheike/blogs.git
  
  git remote set-url origin quanqiuheike:quanqiuheike/blogs.git
  ```

  

### 步骤一：设置第一个 GitHub 账号RSA加密方式

**1、生成 SSH 密钥对**：

打开终端并输入以下命令：

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

ssh-keygen -t rsa -b 4096 -C "quanqiuhaike@gmail.com"
```

在这个命令中：

\- `-t rsa` 指定生成 RSA 密钥对。

\- `-b 4096` 指定密钥长度为 4096 位，这是一种更安全的选项。

\- `-C "your_email@example.com"` 是你的 GitHub 邮箱地址。

**2、保存密钥对：**

在提示中输入文件保存的路径和名称，例如：`/Users/your_username/.ssh/id_rsa_first`。这将创建一个名为 `id_rsa_first` 的私钥文件和一个名为 `id_rsa_first.pub` 的公钥文件。

**3、将公钥添加到 GitHub 账号：**

将 `id_rsa_first.pub` 文件中的内容复制到 GitHub 账号的 SSH 设置中。

### 步骤二：设置第二个 GitHub 账号



**1、 生成 SSH 密钥对**：

为了避免冲突，我们将指定新的路径和文件名。在终端中输入以下命令：

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f /Users/your_username/.ssh/id_rsa_second


ssh-keygen -t rsa -b 4096 -C "1831814662@qq.com" -f /Users/your_username/.ssh/id_rsa_second
```

这个命令与之前的命令相似，只是增加了 `-f` 选项来指定新的路径和文件名。

**2、将公钥添加到 GitHub 账号：**

将 `id_rsa_second.pub` 文件中的内容复制到第二个 GitHub 账号的 SSH 设置中。

### 步骤三：配置 SSH 主机别名

由于你有两个不同的 SSH 密钥对，你需要告诉 SSH 在连接 GitHub 时使用哪个密钥。为此，你可以通过编辑 `~/.ssh/config` 文件来指定别名和使用的密钥。

**1、打开终端并输入以下命令：**

```
nano ~/.ssh/config
```

**2、在打开的文件中添加以下内容：**

```
# 第一个 GitHub 账号**

Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_first


# 第二个 GitHub 账号

Host github-second
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_second
```



确保将 `~/.ssh/id_rsa_first` 和 `~/.ssh/id_rsa_second` 替换为实际的私钥文件路径。



**3、 保存并关闭文件（在 Nano 编辑器中按下 `Ctrl + X`，然后按 `Y` 确认保存，最后按 `Enter` 退出）。**



### 步骤四：验证配置是否成功

**1、 测试第一个 GitHub 账号是否配置成功：**

```
  ssh -T git@github.com
```

如果成功，会显示你的 GitHub 用户名。



**2、 测试第二个 GitHub 账号是否配置成功：**

```
  ssh -T git@github-second
```

同样，如果成功，会显示你的 GitHub 用户名。

现在，你的 Mac 电脑上已经配置了两个 GitHub 账号，并且每个账号都有自己的 SSH 密钥对。



```
git config --global user.name "Your Name"

git config --global user.email "your.email@example.com"


git config --global user.name "quanqiuheike"

git config --global user.email "cqxengineer@gmail.com"


<!-- 查看当前全局Git配置：确保其中的 user.email 和 user.name 设置为您当前要提交代码的GitHub账户的信息。-->

git config --global --list
```