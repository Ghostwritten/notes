# Gitbook 持续更新 Github Pages


---

## 1. 新增 md

```bash
$ ls
_book  gitbook  index.html  README.md  search_index.json  SUMMARY.md
$ ls _book/
gitbook  index.html  search_index.json

新增Gihub目录，以及文章
$ ls
_book  gitbook  Github  index.html  README.md  search_index.json  SUMMARY.md

$ ls Github/
1_github_introduce.md  2_github_local_pull_github.md  3_github_action.md  4_github_page.md 
```


## 2. 定制 book.json
或许偶尔你会手动更新book.json,比如：新增插件、分享链接等等。
```bash
$ cat book.json 
{
  "title": "Gitbook Docs",
  "author": "宗勋 - zongxun",
  "description": "这是一本关于Git、Github、Gitlab、Gitbook、GitOps的书籍",
  "language": "zh-hans",
  "links": {
     "sharing": {
         "all": null,
         "facebook": null,
         "google": null,
         "twitter": null,
         "weibo": null
     },
     "sidebar": {
        "zongxun's Blog": "https://smoothies.com.cn"
     }
  },
  "plugins": [
    "3-ba",
    "accordion",
    "advanced-emoji",
    "anchor-navigation-ex",
    "baidu-tongji",
    "code",
    "change_girls",
    "custom-favicon",
    "donate",
    "chapter-fold",
    "edit-link",
    "flexible-alerts",
    "github-buttons",
    "github",
    "lightbox",
    "insert-logo",
    "musicxml",
    "prism",
    "pageview-count",
    "-highlight",
    "-search",
    "-lunr",
    "rss",
    "search-plus",
    "splitter",
    "-sharing",
    "sharing-plus",
    "sidebar-style",
    "theme-comscore",
    "tbfed-pagefooter"
  ],
  "pluginsConfig": {

    "github": {
      "url": "https://github.com/Ghostwritten"
    },

    "github-buttons": {
      "buttons": [
        {
          "user": "Ghostwritten",
          "repo": "gitbook-docs", 
          "type": "star",
          "count": true,
          "size": "small"
        }
      ]
    },

    "change_girls" : {
      "time" : 10,
      "urls" : [
          "https://www.bizhishe.com/d/file/2019-08-26/1566827846505876.jpg", "https://www.bizhishe.com/d/file/2019-07-24/1563977671157231.jpg", "https://www.bizhishe.com/d/file/2019-07-14/1563116649970786.jpg"
      ]
    },

    "chapter-fold":{},

    "favicon": "assets/imgs/1_girl.ico",

    "donate": {
      "button": "打赏",
      "alipayText": "支付宝打赏",
      "wechatText": "微信打赏",
      "alipay": "https://github.com/Ghostwritten/gitbook-docs/blob/gh-pages/assets/imgs/aplipay.png?raw=true",
      "wechat": "https://github.com/Ghostwritten/gitbook-docs/blob/gh-pages/assets/imgs/wechat.png?raw=true"
    },

    "edit-link": {
     "base": "https://github.com/Ghostwritten/gitbook-docs/edit/master/",
     "label": "Edit"
    },

    "prism": {
      "lang": {
        "shell": "bash"
      }
    },
    "tbfed-pagefooter": {
      "copyright":"Copyright &copy ghostwritten 浙ICP备2020032454号 2022",
      "modify_label": "该文件修订时间：",
      "modify_format": "YYYY-MM-DD HH:mm:ss"
    },
    "baidu-tongji": {
      "token": "55e7dfe47f4dc1c018d4042fdfa62565"
    },
    "anchor-navigation-ex": {
      "showLevel": false
    },

    "sidebar-style": {
       "title": "《Gitbook Docs》",
       "author": "zongxun"
    },

    "flexible-alerts": {
      "note": {
        "label": "Note"
      },
      "tip": {
        "label": "Tip"
      },
      "warning": {
        "label": "Warning"
      },
      "danger": {
        "label": "Danger"
      }
    },

    "3-ba": {
        "token": "9ffc0dce8d7079aceab6b0bc18eb626b"
    },

    "insert-logo": {
      "url": "https://www.bizhishe.com/d/file/2019-07-14/1563116649268975.jpg",
      "style": "background: none; max-height: 100px; min-height: 100px"
    },

    "rss": {
      "title": "Gitbook Docs",
      "description": "This is the best book ever.",
      "author": "Zong Xun",
      "site_url": "https://smoothies.com.cn/gitbook-docs/",
      "managingEditor": "writer@smoothies.com.cn (Zong Xun)",
      "webMaster": "webmaster@smoothies.com.cn (Zong Xun)",
      "categories": [
        "gitbook"
      ]
    },

    "sharing": {
      "douban": false,
      "facebook": true,
      "google": false,
      "pocket": false,
      "qq": false,
      "qzone": false,
      "twitter": true,
      "weibo": false,
    "all": [
       "facebook", "google", "twitter"
    ]
   }
  }
}
$ ls
_book  book.json  gitbook  Github  index.html  README.md  search_index.json  SUMMARY.md
```

更多细节请参考[pages定制](https://www.notion.so/gitbook-book-json-4f76e78c59df41ed8d20e3a2cc3ee602)


## 3. 脚本发布
新建 `Person Access Token`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6439407e4c04d6631d63232e7f9b5eca.png)
复制token
```bash
#!/bin/bash

# author: ghostwritten
# date:   01/06 2022
# description: deploy Github Pages

# ##############################################################################
set -o nounset

FILE_NAME="update.sh"
FILE_VERSION="v1.0"
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"


if [ $# != 1 ] ; then
  echo "USAGE: $0 something "
  echo " e.g.: $0 update github pages"
  exit 1;
fi

update=$1
#token=$2


user='Ghostwritten'
email='1zoxun1@gmail.com'
repo="github.com/${user}/gitbook-docs.git"

book sm
python3 gitbook-auto-summary.py  .

dirs=`grep -E '\- ' SUMMARY-GitBook-auto-summary.md  | awk '{print $2}'`

for dir in $dirs
do
  dir_README=`grep -E "\[${dir}\]" SUMMARY.md | sed 's/^[ \t]*//g'`
  dir_README=${dir_README//\//\\\/}
  dir_README=${dir_README//\[/\\[}
  dir_README=${dir_README//\]/\\]}
  dir_README=${dir_README//\(/\\\(}
  dir_README=${dir_README//\)/\\\)}
  dir_README=${dir_README//\-/\\\-}
  sed -r -i "s#\\- ${dir}\$#$dir_README#g" SUMMARY-GitBook-auto-summary.md
done

cp -r SUMMARY-GitBook-auto-summary.md SUMMARY.md

gitbook build 

git remote add origin https://${repo}
git add .
git commit -m "${update}"
git push origin master

cd _book
git init
git remote add origin https://${repo}
git add . 
git commit -m "${update} For Github Pages"
git branch -M master
git push --force --quiet "https://${TOKEN}@${repo}" master:gh-pages
```
执行：

```bash
$ bash deploy.sh "update <something>"
```

