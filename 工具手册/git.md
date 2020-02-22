[TOC]
## git操作相关手册

### 基本命令
- 网上资料比较多

### 遇到的问题
#### 项目问题
- 将本地的仓库关联到对应的远程仓库
```bash
echo "# go-test" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:dxxxue/go-test.git
git push -u origin master
```

#### 理解问题