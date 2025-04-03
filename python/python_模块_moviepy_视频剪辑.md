
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/91fc1e8e4a8d49f5721ad7fea721cdd5.png)


 - [python 3.10.0源码编译安装](https://ghostwritten.blog.csdn.net/article/details/122587523)
 - [python pip 安装](https://ghostwritten.blog.csdn.net/article/details/104273575)

打开新世界大门了，视频剪辑还能这样玩，我真tm奥特了。

##  1. 安装 moviepy

```bash
pip install moviepy
```

## 2. 视频剪辑

```bash
from moviepy.editor import*


# 剪辑50-60秒的音乐 00:01:20 - 01:07:10
video =CompositeVideoClip([VideoFileClip("E:\视频\美剧\边缘世界\边缘世界第一季01集.mp4").subclip(70,80)])


# 写入剪辑完成的音乐
video.write_videofile("E:\视频\美剧\边缘世界\边缘世界第一季01集_1.mp4")
```

## 3. 视频拼接

```bash
from moviepy.editor importVideoFileClip, concatenate_videoclips

clip1 =VideoFileClip("myvideo.mp4")

# 结合剪辑，你甚至能够完全自动化剪辑拼接视频的操作
clip2 =VideoFileClip("myvideo2.mp4").subclip(50,60)
clip3 =VideoFileClip("myvideo3.mp4")

final_clip = concatenate_videoclips([clip1,clip2,clip3])
final_clip.write_videofile("my_concatenation.mp4")
```
## 4. 逐帧变化
那你能完成针对每一帧图像的快速图像处理吗？PR 可是做得到的哦”

那当然可以，教你如何反转视频每一帧的绿色和蓝色通道：

```bash
from moviepy.editor importVideoFileClip

my_clip =VideoFileClip("videoplayback.mp4")


def scroll(get_frame, t):
    """
    处理每一帧图像
    """

    frame = get_frame(t)
    frame_region = frame[:,:,[0,2,1]]
    return frame_region


modifiedClip = my_clip.fl(scroll)

modifiedClip.write_videofile("test.mp4")
```
## 5. 导出GIF

```bash
from moviepy.editor import*

# 剪辑50-60秒的音乐 00:00:50 - 00:00:60
video = CompositeVideoClip([VideoFileClip("videoplayback.mp4").subclip(50,60)])

my_clip.write_gif('test.gif', fps=12)
```

参考：

 - [https://zulko.github.io/moviepy/#](https://zulko.github.io/moviepy/#)
 - [https://zulko.github.io/](https://zulko.github.io/)

