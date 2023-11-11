#  Linux Command tput 终端操作
tags: 编辑器



---
##  1. 简介
tput 命令将通过 terminfo 数据库对您的终端会话进行初始化和操作。通过使用 tput，您可以更改几项终端功能，如移动或更改光标、更改文本属性，以及清除终端屏幕的特定区域。 


与 UNIX 中的大多数命令一样，tput 命令既可以用在 shell 命令行中也可以用在 shell 脚本中。为让您更好地理解 tput，本文首先从命令行讲起，然后紧接着讲述 shell 脚本示例。

## 2. 什么是 terminfo 数据库
UNIX 系统上的 terminfo 数据库用于定义终端和打印机的属性及功能，包括各设备（例如，终端和打印机）的行数和列数以及要发送至该设备的文本的属性。UNIX 中的几个常用程序都依赖 terminfo 数据库提供这些属性以及许多其他内容，其中包括 vi 和 emacs 编辑器以及 curses 和 man 程序。

与 UNIX 中的大多数命令一样，tput 命令既可以用在 shell 命令行中也可以用在 shell 脚本中。为让您更好地理解 tput，本文首先从命令行讲起，然后紧接着讲述 shell 脚本示例。


##  3. 参数

```bash
(1)字符串输出参数设置 
　　bel       警铃 
　　blink     闪烁模式 
　　bold      粗体 
　　civis     隐藏光标 
　　clear     清屏 
　　cnorm     不隐藏光标 
　　cup       移动光标到屏幕位置（x，y） 
　　el        清除到行尾 
　　ell       清除到行首 
　　smso      启动突出模式 
　　rmso      停止突出模式 
　　smul      开始下划线模式 
　　rmul      结束下划线模式 
　　sc        保存当前光标位置 
　　rc        恢复光标到最后保存位置 
　　sgr0      正常屏幕 
　　rev       逆转视图 
(2)数字输出参数设置 
　　cols      列数目 
　　ittab     设置宽度 
　　lines     屏幕行数 
(3)布尔输出参数设置 
　　chts      光标不可见 
　　hs        具有状态行  
（4）景色
   setaf ColorNumber## 设置前景色
   setab ColorNumber ##设置背景色
```
##  4. 属性
###  4.1 光标属性

在 UNIX shell 脚本中或在命令行中，移动光标或更改光标属性可能是非常有用的。有些情况下，您可能需要输入敏感信息（如密码），或在屏幕上两个不同的区域输入信息。在此类情况下，使用 tput 可能会对您有所帮助。

```bash
tput clear # 清屏
tput sc # 保存当前光标位置
tput cup 10 13 # 将光标移动到 x y
tput civis # 光标不可见
tput cnorm # 光标可见
tput rc # 显示输出，返回光标位置

exit 0
```
###  4.2 移动光标

使用 tput 可以方便地实现在各设备上移动光标的位置。通过在 tput 中使用 cup 选项，或光标位置，您可以在设备的各行和各列中将光标移动到任意 X 或 Y 坐标。设备左上角的坐标为 (0,0)。

要在设备上将光标移动到第 5 列 (X) 的第 1 行 (Y)，只需执行 tput cup 5 1。另一个示例是 tput cup 23 45，此命令将使光标移动到第 23 列上的第 45 行。

###  4.3 移动光标并显示信息
另一种有用的光标定位技巧是移动光标，执行用于显示信息的命令，然后返回到前一光标位置：

```bash
(tput sc ; tput cup 23 45 ; echo “Input from tput/echo at 23/45” ; tput rc)
```

下面我们分析一下 subshell 命令：

```bash
tput sc
```

必须首先保存当前的光标位置。要保存当前的光标位置，请包括 sc 选项或“save cursor position”。

```bash
tput cup 23 45
```

在保存了光标位置后，光标坐标将移动到 (23,45)。

```bash
echo “Input from tput/echo at 23/45”
```

将信息显示到 stdout 中。

```bash
tput rc
```

在显示了这些信息之后，光标必须返回到使用 tput sc 保存的原始位置。要使光标返回到其上次保存的位置，请包括 rc 选项或“restore cursor position”。

> 注意：由于本文首先详细介绍了通过命令行执行 tput，因此您可能会觉得在自己的 subshell中执行命令要比单独执行每条命令然后在每条命令执行之前显示提示更简洁。

###  4.4 更改光标的属性

在向某一设备显示数据时，很多时候您并不希望看到光标。将光标转换为不可见可以使数据滚动时的屏幕看起来更整洁。要使光标不可见，请使用 civis 选项（例如，`tput civis`）。在数据完全显示之后，您可以使用 `tput cnorm` 选项将光标再次转变为可见。

###  4/5 文本属性
更改文本的显示方式可以让用户注意到菜单中的一组词或警惕用户注意某些重要的内容。您可以通过以下方式更改文本属性：使文本加粗、在文本下方添加下划线、更改背景颜色和前景颜色，以及逆转颜色方案等。

要更改文本的颜色，请使用 setb 选项（用于设置背景颜色）和 setf 选项（用于设置前景颜色）以及在 terminfo 数据库中分配的颜色数值。通常情况下，分配的数值与颜色的对应关系如下，但是可能会因 UNIX 系统的不同而异：

0：黑色
1：蓝色
2：绿色
3：青色
4：红色
5：洋红色
6：黄色
7：白色

执行以下示例命令可以将背景颜色更改为黄色，将前景颜色更改为红色：

```bash
tput setb 6 tput setf 4
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a46b018c77be4693b6d5b0af5b263250.png)
要反显当前的颜色方案，只需执行tput rev。

有时，仅为文本着色还不够，也就是说，您想要通过另一种方式引起用户的注意。可以通过两种方式达到这一目的：一是将文本设置为粗体，二是为文本添加下划线。

要将文本更改为粗体，请使用 `bold` 选项。要开始添加下划线，请使用 `smul` 选项。在完成显示带下划线的文本后，请使用 `rmul` 选项。

##  5. 实例
### 5.1 变量格式
```bash
#加粗
bold=$(tput bold)
#下划线
underline=$(tput sgr 0 1)
#重置规则
reset=$(tput sgr0)
#红色
red=$(tput setaf 1)
#绿色
green=$(tput setaf 2)
```

### 5.2 使字符串有颜色，底色，加粗

```bash
#!/bin/bash
printf $(tput setaf 2; tput bold)'color show\n\n'$(tput sgr0)

for((i=0; i<=7; i++)); do
    echo $(tput setaf $i)"show me the money"$(tput sgr0)
done

printf '\n'$(tput setaf 2; tput setab 0; tput bold)'background color show'$(tput sgr0)'\n\n'

for((i=0,j=7; i<=7; i++,j--)); do
    echo $(tput setaf $i; tput setab $j; tput bold)"show me the money"$(tput sgr0)
done

exit 0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3d7ec0a227da451797729bc917a5f301.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_13,color_FFFFFF,t_70,g_se,x_16)
### 5.3 格式传参输出

```bash
#!/bin/bash

# $1 str       print string
# $2 color     0-7 设置颜色
# $3 bgcolor   0-7 设置背景颜色
# $4 bold      0-1 设置粗体
# $5 underline 0-1 设置下划线

function format_output(){
    str=$1
    color=$2
    bgcolor=$3
    bold=$4
    underline=$5
    normal=$(tput sgr0)

    case "$color" in
        0|1|2|3|4|5|6|7)
            setcolor=$(tput setaf $color;) ;;
        *)
            setcolor="" ;;
    esac

    case "$bgcolor" in
        0|1|2|3|4|5|6|7)
            setbgcolor=$(tput setab $bgcolor;) ;;
        *)
            setbgcolor="" ;;
    esac

    if [ "$bold" = "1" ]; then
        setbold=$(tput bold;)
    else
        setbold=""
    fi

    if [ "$underline" = "1" ]; then
        setunderline=$(tput smul;)
    else
        setunderline=""
    fi

    printf "$setcolor$setbgcolor$setbold$setunderline$str$normal\n"
}

format_output "Yesterday Once more" 2 5 1 1

exit 0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/da271b6d90a54f52b0eb0e8d4f843978.png)
###  5.4 光标定位输出

```bash
#!/bin/bash
# clear the screen
tput clear
# Move cursor to screen location X,Y (top left is 0,0)
tput cup 3 15
# set a foreground colour using ANSI escape
tput setaf 3
echo "XYX Corp LTD."
tput sgr0
tput cup 5 17
# Set reverse video mode
tput rev
echo "M A I N - M E N U"
tput sgr0
tput cup 7 15
echo "1\. User Management"
tput cup 8 15
echo "2\. service Management"
tput cup 9 15
echo "3\. Process Management"
tput cup 10 15
echo "4\. Backup"
# Set bold mode
tput bold
tput cup 12 15
read -p "Enter your choice [1-4] " choice
tput clear
tput sgr0
tput rc

exit 0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e15daa51887d4b9c828f498660d85724.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_19,color_FFFFFF,t_70,g_se,x_16)
###  5.5 输出艺术字体

```bash
root@test1:~/shell/tput# apt install figlet
root@test1:~/shell/tput# wget http://www.figlet.org/fonts/roman.flf
--2021-11-18 15:44:37--  http://www.figlet.org/fonts/roman.flf
Resolving www.figlet.org (www.figlet.org)... 188.226.162.120
Connecting to www.figlet.org (www.figlet.org)|188.226.162.120|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 12948 (13K) [text/plain]
Saving to: ‘roman.flf’

roman.flf                                                      100%[====================================================================================================================================================>]  12.64K  --.-KB/s    in 0s      

2021-11-18 15:44:38 (70.6 MB/s) - ‘roman.flf’ saved [12948/12948]

root@test1:~/shell/tput# echo; tput setaf 1; figlet -c  Happy; tput setaf 2; figlet -c  Holidays; tput sgr0;
```

                         _   _                         
                        | | | | __ _ _ __  _ __  _   _ 
                        | |_| |/ _` | '_ \| '_ \| | | |
                        |  _  | (_| | |_) | |_) | |_| |
                        |_| |_|\__,_| .__/| .__/ \__, |
                                    |_|   |_|    |___/ 
                     _   _       _ _     _                 
                    | | | | ___ | (_) __| | __ _ _   _ ___ 
                    | |_| |/ _ \| | |/ _` |/ _` | | | / __|
                    |  _  | (_) | | | (_| | (_| | |_| \__ \
                    |_| |_|\___/|_|_|\__,_|\__,_|\__, |___/
                                                 |___/     
```bash
root@test1:~/shell/tput# echo; tput setaf 1; figlet -c -f roman Happy; tput setaf 2; figlet -c -f roman Holidays; tput sgr0;
```

```bash
           ooooo   ooooo                                             
           `888'   `888'                                             
            888     888   .oooo.   oo.ooooo.  oo.ooooo.  oooo    ooo 
            888ooooo888  `P  )88b   888' `88b  888' `88b  `88.  .8'  
            888     888   .oP"888   888   888  888   888   `88..8'   
            888     888  d8(  888   888   888  888   888    `888'    
           o888o   o888o `Y888""8o  888bod8P'  888bod8P'     .8'     
                                    888        888       .o..P'      
                                   o888o      o888o      `Y8P'       
                                                                     
 ooooo   ooooo           oooo   o8o        .o8                                 
 `888'   `888'           `888   `"'       "888                                 
  888     888   .ooooo.   888  oooo   .oooo888   .oooo.   oooo    ooo  .oooo.o 
  888ooooo888  d88' `88b  888  `888  d88' `888  `P  )88b   `88.  .8'  d88(  "8 
  888     888  888   888  888   888  888   888   .oP"888    `88..8'   `"Y88b.  
  888     888  888   888  888   888  888   888  d8(  888     `888'    o.  )88b 
 o888o   o888o `Y8bod8P' o888o o888o `Y8bod88P" `Y888""8o     .8'     8""888P' 
                                                          .o..P'               
                                                          `Y8P'                
                                                                               
```


### 5.6 时钟倒计时

```bash
  #!/bin/bash

    # tclock - Display a clock in a terminal

    BG_BLUE="$(tput setab 4)"
    FG_BLACK="$(tput setaf 0)"
    FG_WHITE="$(tput setaf 7)"

    terminal_size() { # Calculate the size of the terminal
      
      terminal_cols="$(tput cols)"
      terminal_rows="$(tput lines)"
    }

    banner_size() {

      # Because there are different versions of banner, we need to
      # calculate the size of our banner's output

      banner_cols=0
      banner_rows=0
      
      while read; do
        [[ ${#REPLY} -gt $banner_cols ]] && banner_cols=${#REPLY}
        ((++banner_rows))
      done < <(banner "12:34 PM")
    }

    display_clock() {
      
      # Since we are putting the clock in the center of the terminal,
      # we need to read each line of banner's output and place it in the
      # right spot.
      
      local row=$clock_row
      
      while read; do
        tput cup $row $clock_col
        echo -n "$REPLY"
        ((++row))
      done < <(banner "$(date +'%I:%M %p')")
    }

    # Set a trap to restore terminal on Ctrl-c (exit).
    # Reset character attributes, make cursor visible, and restore
    # previous screen contents (if possible).

    trap 'tput sgr0; tput cnorm; tput rmcup || clear; exit 0' SIGINT

    # Save screen contents and make cursor invisible
    tput smcup; tput civis

    # Calculate sizes and positions
    terminal_size
    banner_size
    clock_row=$(((terminal_rows - banner_rows) / 2))
    clock_col=$(((terminal_cols - banner_cols) / 2))
    progress_row=$((clock_row + banner_rows + 1))
    progress_col=$(((terminal_cols - 60) / 2))

    # In case the terminal cannot paint the screen with a background
    # color (tmux has this problem), create a screen-size string of 
    # spaces so we can paint the screen the hard way.

    blank_screen=
    for ((i=0; i < (terminal_cols * terminal_rows); ++i)); do
      blank_screen="${blank_screen} "
    done

    # Set the foreground and background colors and go!
    echo -n ${BG_BLUE}${FG_WHITE}
    while true; do

      # Set the background and draw the clock
      
      if tput bce; then # Paint the screen the easy way if bce is supported
        clear
      else # Do it the hard way
        tput home
        echo -n "$blank_screen"
      fi
      tput cup $clock_row $clock_col
      display_clock
      
      # Draw a black progress bar then fill it in white
      tput cup $progress_row $progress_col
      echo -n ${FG_BLACK}
      echo -n "###########################################################"
      tput cup $progress_row $progress_col
      echo -n ${FG_WHITE}

      # Advance the progress bar every second until a minute is used up
      for ((i = $(date +%S);i < 60; ++i)); do
        echo -n "#"
        sleep 1
      done
    done
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/237295b1bf174717b906c3b18b9f0790.gif#pic_center)


参考：

 - [IBM tput Command](https://www.ibm.com/docs/en/aix/7.2?topic=t-tput-command)
 - [tput: Portable Terminal Control](https://www.gnu.org/software/termutils/manual/termutils-2.0/html_chapter/tput_1.html)
 - [tput command](http://linuxcommand.org/lc3_adv_tput.php)
 - [figlet command](http://www.figlet.org/figlet-man.html)


