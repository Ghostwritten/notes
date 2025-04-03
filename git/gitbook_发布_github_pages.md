# Gitbook 发布 Github Pages


---

# 1. github 创建仓库

例如：`git-handbook`

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a3ad1759d23fc0ede1ba8c6701bf24c.png)


# 2. 本地创建github仓库

## 2.1 linux

仓库名：`git-handbook`

初始化配置操作，推送[github pages](https://pages.github.com/)

```bash
$ mkdir git-handbook
$ cd git-handbook
$ gitbook init
$ gitbook build
$ ls _book/
gitbook  index.html  search_index.json
$  ls
_book  README.md  SUMMARY.md

$ git init
$ echo "*~" > .gitignore
$ echo "_book" >> .gitignore

$ ls
_book  gitbook  index.html  README.md  search_index.json  SUMMARY.md
$ git add --all
$ git commit -m "Publish book"
$ git remote add origin https://github.com/Ghostwritten/git-handbook.git
$ git push -u origin master

Username for 'https://github.com': ghostwritten
Password for 'https://ghostwritten@github.com':<token>
```

# 3. 配置 github pages

## 3.1 新建分支

```bash
git checkout --orphan gh-pages
```

## 3.2 删除不需要的文件

切换到pages分支后，我们需要将_books目录之外的文件都清理掉：

```bash
git rm --cached -r .
git clean -df
rm -rf *~
```

## 3.3 添加忽略文件

使用文本编辑器，创建名为`.gitignore`的文件，内容如下：

```bash
*~
_book
.DS_Store
```

通过.gitignore文件，本地仓库将忽略临时文件和_book文件夹。

## 3.4 复制_book文件夹到分支根目录

```bash
cp -r _book/* .
```

## 3.5 推送

```bash
git add .
git commit -m '更新说明'
git push -u origin gh-pages
```

master与gh-pages的不同之处：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0686c53b7d0afb3e246a9b5ebb31618f.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a7ad328a8a439518bc2918f44a34399.png)


## 3.6 验证

如果没有自定义的域名，你访问的方式地址为：*`http://<github名字>.github.io/git-handbook`*

这里我配置了自己的域名，`ghostwritten*.github.io`* 重定向到了个人站点 [`smoothies.com.cn`](http://smoothies.com.cn/)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16ec0e5922dfce49c527a6d836f81204.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/55b447badb3620f5bff5170d173022d9.png)


# 4. 推送至gitbook仓库

[注册gitbook](https://www.gitbook.com/)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/55fdfe196eaf34ebe6b5838557109bce.png)


![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-JraiYmau-1654254554721)(https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7e0c6659-b34f-4105-9d50-daa086d748e3/Untitled.png)\]](https://i-blog.csdnimg.cn/blog_migrate/75ad07af6d517503ed0af3a830fd72b2.png)


与仓库同名

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7f55634883711192f1832b78051fc86.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75de4764aa19251a0b31dcbbcb2ca0ac.png)


# 5. github action：master 同步 gh-pages

## 5.1 新建 Person Access Token
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a182dfc16df43a6f453fb084a1821aa9.png)


复制**`Person Access Token`**

## 5.2 创建 secret

创建secret：`ACTIONS_DEPLOY_KEY` ，value填写**`Person Access Token`**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc13c9ee06f870c1b225aefadbc68dad.png)


## 5.3 编写 .github/workflows/main.yml 

```bash
name: auto-generate-gitbook
on:                                 #在master分支上进行push时触发  
  push:
    branches:
    - master

jobs:
  main-to-gh-pages:
    runs-on: ubuntu-latest
        
    steps:                          
    - name: checkout master
      uses: actions/checkout@v2
      with:
        ref: master
            
    - name: install nodejs
      uses: actions/setup-node@v1
      
    - name: configue gitbook
      run: |
        npm install -g gitbook-cli          
        gitbook install
        npm install -g gitbook-summary
                
    - name: generate _book folder
      run: |
        book sm
        gitbook build
        cp SUMMARY.md _book
                
    - name: push _book to branch gh-pages 
      env:
        TOKEN: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        REF: github.com/${{ github.repository }}.git
        MYEMAIL: 1zoxun1@gmail.com                  # ！！记得修改为自己github设置的邮箱
        MYNAME: ${{ github.repository_owner }}          
      run: |
        cd _book
        git config --global user.email "${MYEMAIL}"
        git config --global user.name "${MYNAME}"
        git init
        git remote add origin https://${REF}
        git add . 
        git commit -m "Updated By Github Actions With Build ${{github.run_number}} of ${{github.workflow}} For Github Pages"
        git branch -M master
        git push --force --quiet "https://${TOKEN}@${REF}" master:gh-pages
```

## 5.4 推送 github

```bash
git add .
git commit -m "update **[.github](https://github.com/rootsongjc/kubernetes-handbook/tree/master/.github)/[workflows](https://github.com/rootsongjc/kubernetes-handbook/tree/master/.github/workflows)/main.yml**"
git push origin master
```

## 5.5 验证
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e121a524aae73221ab675f96c25106a2.png)

