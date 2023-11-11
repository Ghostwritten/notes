


---
## 1. 背景

最近准备搭建新博客,所以想把所有csdn的文章下载下来，但实在太多，不可能一篇一篇去导出，所以写了一个批量导出脚本，尽管csdn的在线编辑、发布、专栏、自定义模块、模板等比较成熟,但实在没有一点美感,这一点令人比较失望，一开始我比较在意在线编辑速度快，笔记更新方便，检索也还算可以,前几天[阿里云开发者社区](https://developer.aliyun.com/)、[infoQ中国社区](https://www.infoq.cn/)运营人员相继邀请我去他们平台发布文章，但我更想尝试本地利用[Obsidian](https://obsidian.md/)工具编写笔记试试，并同步[github](https://github.com/)或[gitee](https://gitee.com/)仓库存储,博客也许会以github page依托利用[hexo](https://hexo.io/zh-cn/)、[Jekyll](https://jekyllcn.com/)等工具发布，如果还可以同步[notion](https://www.notion.so/)或[云雀](https://www.yuque.com/)就更完美。

## 2. 功能
1. 通过传入id确立个人用户主页；
2. 创建个人博客目录，专栏目录；
3. 获取专栏URL、名称、篇数量；
4. 依靠专栏URL获取对应的多页文章URL、标题；
5. 遍历文章URL通过cookie获取文章内容并转换markdown格式。

## 3. 下载

```bash
$ git clone https://github.com/Ghostwritten/csdn_to_md.git 
```

## 4. 配置

chrome浏览器登陆csdn平台，按"F12"找到自己网页cookie,选择部分cookie内容复制至csdn_to_md.py脚本109行。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-CPYqwJNL-1651508805129)(https://github.com/Ghostwritten/csdn_to_md/blob/main/cookie.png)]

##  5. 代码
**github仓库**：[csdn_to_md](https://github.com/Ghostwritten/csdn_to_md)

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import json
import os
import uuid
import time
import requests
import datetime
import argparse
import re
from bs4 import BeautifulSoup


def request_blog_column(id):

    headers = {
      'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.75 Safari/537.36'
    }
    
    urls = 'https://blog.csdn.net/' + id
    reply = requests.get(url=urls,headers=headers)
    parse = BeautifulSoup(reply.content, "lxml")
    spans = parse.find_all('a', attrs={'class':'special-column-name'})

    blog_columns = []
    
    for span in spans:
          href = re.findall(r'href=\"(.*?)\".*?',str(span),re.S)
          href = ''.join(href)

          headers = {
          'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.75 Safari/537.36'
          }
          blog_column_reply = requests.get(url=href,headers=headers)
          blogs_num = re.findall(r'<a class="clearfix special-column-name" target="_blank" href=\"'+ href +'\".*?<span class="special-column-num">(.+?)篇</span>',blog_column_reply.text,re.S)
          blogs_column_num = str(blogs_num[0])


          blog_column = span.text.strip()

          blog_column_dir = './' + str(id) + '/' + str(blog_column)          
          if not os.path.exists(blog_column_dir):
            os.mkdir(blog_column_dir)

          blog_id = href.split("_")[-1]
          blog_id = blog_id.split(".")[0]
          blog_columns.append([href,blog_column,blog_id,blogs_column_num])
          
 

    return blog_columns

def request_blog_list(id):
    blog_columns = request_blog_column(id)
    blogs = []
    for blog_column in blog_columns:
        blog_column_url = blog_column[0]
        blog_column_name = blog_column[1]
        blog_column_id = blog_column[2]
        blog_column_num = int(blog_column[3])
     

        if blog_column_num > 40: 
           page_num = round(blog_column_num/40)
           for i in range(page_num,0,-1):
               blog_column_url = blog_column[0]
               url_str = blog_column_url.split('.html')[0]
               blog_column_url = url_str + '_' + str(i) + '.html'
               append_blog_info(blog_column_url,blog_column_name,blogs)
           blog_column_url = blog_column[0]
           blogs = append_blog_info(blog_column_url,blog_column_name,blogs)
               
        else:
           blogs = append_blog_info(blog_column_url,blog_column_name,blogs)

    return blogs


def append_blog_info(blog_column_url,blog_column_name,blogs):
    headers = {
      'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.75 Safari/537.36'
    }
    reply = requests.get(url=blog_column_url,headers=headers)
    blog_span = BeautifulSoup(reply.content, "lxml")
    blogs_list = blog_span.find_all('ul', attrs={'class':'column_article_list'})
    for arch_blog_info in blogs_list:
        blogs_list = arch_blog_info.find_all('li')
        for blog_info in blogs_list:
	    
            blog_url = blog_info.find('a', attrs={'target':'_blank'})['href']
            blog_title = blog_info.find('h2', attrs={'class':"title"}).get_text().strip().replace(" ", "_").replace('/','_')
            blog_dict = {'column': blog_column_name,'url': blog_url, 'title': blog_title}
            blogs.append(blog_dict)
    return blogs 

def request_md(id):

    blogs = request_blog_list(id)
    
    for blog_dict in blogs:
        blog_url = blog_dict['url']
        blog_title = blog_dict['title']
        blog_column = blog_dict['column']
        blog_id = blog_url.split("/")[-1] 
        url = f"https://blog-console-api.csdn.net/v1/editor/getArticle?id={blog_id}"
        headers = {
        
         "Cookie": "uuid_tt_dd=20_18815108270-1651420958061-407001;dc_session_id=10_1651420958062.474827;acw_tc=276077c116514209580398703eb6ccd1972794b4689cf3b5039f85506a6146;UserName=hahalelehehe; UserInfo=30ee073e05d84844b34a829ef0d80541; UserToken=30ee073e05d84844b34a829ef0d80541; UserNick=ghostwritten; AU=5F9; BT=1651420447587; Hm_up_6bcd52f51e9b3dce32bec4a3997715ac=%7B%22islogin%22%3A%7B%22value%22%3A%221%22%2C%22scope%22%3A1%7D%2C%22isonline%22%3A%7B%22value%22%3A%221%22%2C%22scope%22%3A1%7D%2C%22isvip%22%3A%7B%22value%22%3A%221%22%2C%22scope%22%3A1%7D%2C%22uid_%22%3A%7B%22value%22%3A%22xixihahalelehehe%22%2C%22scope%22%3A1%7D%7D; log_Id_click=1111; c_pref=https%3A//i.csdn.net/; c_ref=https%3A//blog.csdn.net/xixihahalelehehe%3Fspm%3D1010.2135.3001.5343; c_page_id=default; dc_tos=rb7pev; log_Id_pv=1346; Hm_lpvt_6bcd52f51e9b3dce32bec4a3997715ac=1651422057; log_Id_view=2182;",
         "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.87 Safari/537.36"
           
        }
        data = {"id": blog_id}
        reply = requests.get(url, headers=headers,data=data)
        reply_data = reply.json()

        try:
           key = "key" + str(uuid.uuid4())
           content = reply_data["data"]["markdowncontent"].replace("", "")
           blog_title_dir = './' + str(id) + '/' + str(blog_column) + '/' + str(blog_title) + '.md'
           with open(blog_title_dir, "w", encoding="utf-8") as f:
               f.write(content)
             
           print("download blog markdown blog:" + '【' + blog_column + '】' + str(blog_title))
        except Exception as e:
           print("***********************************")
           print(e)
           print(url)




def read_from_json(filename):
    jsonfile = open(filename, "r",encoding='utf-8')
    jsondata = jsonfile.read()
    return json.loads(jsondata)

def write_to_json(filename, data):
    jsonfile = open(filename, "w")
    jsonfile.write(data)



def main():
    parser = argparse.ArgumentParser()

    parser.add_argument('-i', '--id', dest='id',type=str,  required=True, help='csdn name')

    args = parser.parse_args()
    
    csdn_id = args.id

    name_dir = './' + str(csdn_id)
    if not os.path.exists(name_dir):
       os.mkdir(name_dir)

    request_md(csdn_id)

if __name__ == '__main__':
    main()
```

## 5. 演示
* [观看视频](https://www.bilibili.com/video/bv1bL4y1c7UK)

```bash
$ python3 csdn_to_md.py -i  xixihahalelehehe
download blog markdown blog:【helm】helm_快速学习手册
download blog markdown blog:【helm】如何开发一个完整的Helm_charts应用实例
download blog markdown blog:【helm】helm_将yaml文件转换json的插件helm-schema-gen
download blog markdown blog:【helm】helm_NOTES.txt
download blog markdown blog:【helm】helm_test_测试详解
download blog markdown blog:【helm】helm_charts_入门指南
download blog markdown blog:【helm】openshift_Certified_Helm_Charts_实践
download blog markdown blog:【helm】Helm_Values.yaml
......


$ cd xixihahalelehehe 

$ /xixihahalelehehe# tree 
.
├── ansible
│   ├── anible_【模块】_notify.md
│   ├── ansbile【模块】replace_替换.md
│   ├── ansbile_模块开发-自定义模块.md
│   ├── ansible_assert_模块.md
│   ├── ansible_become配置.md
│   ├── ansible_cron_模块.md
│   ├── ansible_debug模块.md
│   ├── ansible_delegate_to_模块.md
│   ├── ansible_file模块详解.md
│   ├── ansible_gather_facts配置.md
│   ├── ansible_hosts_and_groups配置.md
│   ├── ansible_jinja2详解.md
│   ├── ansible-playbook_role角色.md
│   ├── ansible-playbook实战.md
│   ├── ansible_script模块.md
│   ├── ansible_set_fact模块.md
│   ├── ansible_URI模块.md
│   ├── ansible【任务】安装httpd.md
│   ├── ansible变量.md
│   ├── ansible_安装.md
│   ├── ansible_快速学习手册.md
│   ├── ansible【模块】add_host.md
│   ├── ansible【模块】blockinfile.md
│   ├── ansible_【模块】find.md
│   ├── ansible【模块】include_tasks.md
│   ├── ansible【模块】linefile_文件行处理.md
│   ├── ansible【模块】modprobe.md
│   ├── ansible【模块】pause.md
│   ├── ansible_【模块】sysctl.md
│   ├── ansible【模块】systemd.md
│   ├── ansible【模块】template.md
│   ├── ansible【模块】yum.md
│   ├── ansible_系统选择性执行脚本.md
│   ├── ansible远程容器机种方法.md
│   └── ansible_配置.md
├── blog
│   ├── github如何搭建一个博客.md
│   ├── jekyll的一个主题TeXt-theme拆解.md
│   ├── jekyll配置管理github博客.md
│   ├��─ 如何使用jekyll插件.md
│   ├── 如何安装jekyll并搭建一个博客.md
│   └── 如何购买域名.md
├── c++
│   └── makefile入门.md
├── camera
│   ├── A7R2_图标列表.md
│   ├── sony_A7R2介绍.md
│   └── SONY_A7R2_基础操作.md
├── Cisco
│   ├── 运维之思科篇_-----1.VLAN_、_Trunk_、_以太通道及DHCP.md
│   ├── 运维之思科篇_-----2.vlan间通讯_、_动态路由.md
│   ├── 运维之思科篇_-----3.HSRP（热备份路由协议），STP（生成树协议），PVST（增强版PST）.md
│   ├── 运维之思科篇_-----4._标准与扩展ACL_、_命名ACL.md
│   ├── 运维之思科篇_-----5._NAT及静态转换_、_动态转换及PAT.md
│   ├── 运维之思科篇_-----6..md
│   └── 运维之思科篇_-----6.思科项目练习.md

```
## 6. 技术

- [python json](https://blog.csdn.net/xixihahalelehehe/article/details/106550900)
- [python os](https://blog.csdn.net/xixihahalelehehe/article/details/104253123)
- [python time](https://blog.csdn.net/xixihahalelehehe/article/details/108998768)
- [python request](https://blog.csdn.net/xixihahalelehehe/article/details/108996025)
- [python argparse](https://blog.csdn.net/xixihahalelehehe/article/details/121199110)
- [python re](https://blog.csdn.net/xixihahalelehehe/article/details/106247378)
- [python bs4](https://blog.csdn.net/xixihahalelehehe/article/details/124152439)

- [python split()](https://blog.csdn.net/xixihahalelehehe/article/details/124547771)
- [python list 列表](https://blog.csdn.net/xixihahalelehehe/article/details/104437743)
- [python 计算之除法](https://blog.csdn.net/xixihahalelehehe/article/details/124549366)
- [python range()](https://ghostwritten.blog.csdn.net/article/details/124549150)

## 7. 参考
- [https://blog.csdn.net/pang787559613/article/details/105444286](https://blog.csdn.net/pang787559613/article/details/105444286)


![在这里插入图片描述](https://img-blog.csdnimg.cn/245df5936335481ca96b3cac102251f1.gif#pic_center)

