
#  Python 监控文件移动解压



--

##  1. 目的
最近在部署前端项目的时候，需要先将前端项目压缩包通过堡垒机上传到应用服务器的 /tmp 目录下，然后进入应用服务器中，使用 mv 命令将压缩文件移动到 Nginx 项目设定目录，最后使用 unzip 命令解压文件，以此完成项目的部署

仔细分析，大部分操作都是重复性的动作，人工去完成这些操作会大大降低工作效率

本篇文章将介绍如何利用 Python 监控文件夹，以此辅助完成服务的部署动作

##  2. 准备
这里要介绍一个 Python 依赖库`「 watchdog 」`

它可用于监控某个文件目录下的文件变化，包含：删除、修改、新增等操，每一个操作都会回调一个事件函数，我们可以在内部编写自定义的逻辑，以此满足我们的需求



```bash
# 安装依赖包
pip3 install watchdog
```

项目地址：

 - [https://pypi.org/project/watchdog/](https://pypi.org/project/watchdog/)

##  3. 代码

```bash
import os

from watchdog.observers import Observer
from watchdog.events import *
import ntpath
import shutil
import zipfile
import time


# 依赖
# pip3 install watchdog

def get_filename(filepath):
    """
    根据文件夹目录，获取文件名称（待后缀）
    :param filepath:
    :return:
    """
    return ntpath.basename(filepath)


# 移动
class FileMoveHandler(FileSystemEventHandler):
    def __init__(self):
        FileSystemEventHandler.__init__(self)

    def on_moved(self, event):
        # if event.is_directory:
        #     print("directory moved from {0} to {1}".format(event.src_path, event.dest_path))
        # else:
        #     print("file moved from {0} to {1}".format(event.src_path, event.dest_path))
        pass

    # 文件新建
    def on_created(self, event):
        # 新建文件夹
        if event.is_directory:
            # print("directory created:{0}".format(event.src_path))
            pass
        # 新建文件
        else:
            # print("file created:{0}".format(event.src_path))
            filename = get_filename(event.src_path)

            # 如果属于前端的4个项目压缩包，开始文件夹的操作
            if filename in watch_tags:
                self.start(filename)

    def on_deleted(self, event):
        # if event.is_directory:
        #     print("directory deleted:{0}".format(event.src_path))
        # else:
        #     print("file deleted:{0}".format(event.src_path))
        pass

    def on_modified(self, event):
        if event.is_directory:
            # print("directory modified:{0}".format(event.src_path))
            pass
        else:
            # print("file modified:{0}".format(event.src_path))
            filename = get_filename(event.src_path)
            if filename in watch_tags:
                self.start(filename)

    def start(self, filename):
        """
        文件处理逻辑
        :param filename:
        :return:
        """
        try:
            # 文件名不带后缀
            filename_without_suffix = filename.split(".")[0]

            # 源文件路径（压缩包文件）
            source_file_path = watch_folder + filename

            # 目标文件路径（压缩包文件）
            target_file_path = target_folder + filename

            # 目标项目文件夹（目标项目）
            target_project_path = target_folder + filename_without_suffix

            # 1、复制文件到目标文件夹
            print(f"拷贝源目录{source_file_path},目标文件夹:{target_folder}")
            # 删除目标文件夹下的压缩文件
            if os.path.exists(target_file_path):
                os.remove(target_file_path)
            # 移动文件到目标文件夹中
            shutil.move(source_file_path, target_folder)


            # 3、清空目标文件夹中内的所有文件夹（如果存在）
            # 如果不存在，新建一个文件夹
            if os.path.exists(target_project_path):
                shutil.rmtree(target_project_path, ignore_errors=True)


            print(f"项目{filename_without_suffix}移动成功！")
        except Exception as e:
            print("部署失败，错误原因:", str(e.args))


# 解压
class FileUnzipHandler(FileSystemEventHandler):
    def __init__(self):
        FileSystemEventHandler.__init__(self)

        # 文件新建

    def on_created(self, event):
        # 新建文件夹
        if event.is_directory:
            # print("directory created:{0}".format(event.src_path))
            pass
        # 新建文件
        else:
            # print("file created:{0}".format(event.src_path))
            filename = get_filename(event.src_path)

            # 如果属于前端的4个项目压缩包，开始文件夹的操作
            if filename in watch_tags:
                self.start(filename)

    def on_modified(self, event):
        if event.is_directory:
            # print("directory modified:{0}".format(event.src_path))
            pass
        else:
            # print("file modified:{0}".format(event.src_path))
            filename = get_filename(event.src_path)
            if filename in watch_tags:
                self.start(filename)

    def start(self, filename):
        # 文件名不带后缀
        filename_without_suffix = filename.split(".")[0]
        # 目标文件路径（压缩包文件）
        target_file_path = target_folder + filename

        # 目标项目文件夹（目标项目）
        target_project_path = target_folder + filename_without_suffix
        r = zipfile.is_zipfile(target_file_path)
        if r:
            fz = zipfile.ZipFile(target_file_path, 'r')
            for file in fz.namelist():
                # 写入到这个文件夹下
                fz.extract(file, target_folder)
        else:
            # print('This is not zip')
            pass

        # 5、删除压缩包
        # os.remove(target_file_path)


if __name__ == "__main__":
    # 待监听的文件夹目录
    watch_folder = "/tmp/"

    # 项目目标文件夹目录
    target_folder = "/home/project/frontend/"

    # 监听文件夹名称，即：项目压缩包名称
    watch_tags = ['proj1.zip', 'proj2.zip', 'proj3.zip', 'proj4.zip']

    # 创建一个监听器，用来监听文件夹目录
    observer = Observer()

    # 创建两个事件处理对象
    move_handler = FileMoveHandler()
    unzip_handler = FileUnzipHandler()

    # 启动监控任务
    # 参数分别是：观察者、监听目录、是否监听子目录
    observer.schedule(move_handler, watch_folder, True)
    observer.schedule(unzip_handler, target_folder, True)
    observer.start()
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()

# 在后台运行  nohup python3 -u watch_folder.py > watch_folder.log 2>&1 &
# 查看日志：watch_folder.log


```
## 4. 执行
```bash
$ nohup python3 -u watch_folder.py > watch_folder.log 2>&1 &
$ ls /tmp/
$ ls /home/project/frontend/
$ mv /root/proj2.zip  /tmp/
$ ls /home/project/frontend/
proj2  proj2.zip
$ cat  watch_folder.log
拷贝源目录/tmp/proj2.zip,目标文件夹: /home/project/frontend/
项目proj2移动成功！
```

参考：

 - [实战 | 如何用 Python 自动化监控文件夹完成服务部署！](https://mp.weixin.qq.com/s/JzOV1GkZ-XoqbERo6aDsGA)

