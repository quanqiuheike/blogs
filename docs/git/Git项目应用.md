### 1、…or create a new repository on the command line

#### 创建新的Git仓库命令

```
echo "# CC" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:quanqiuheike/CC.git
git push -u origin main
```

### 2、…or push an existing repository from the command line

#### 通过命令推送已存在的Git仓库到

```
git remote add origin git@github.com:quanqiuheike/CC.git
git branch -M main
git push -u origin main
```

