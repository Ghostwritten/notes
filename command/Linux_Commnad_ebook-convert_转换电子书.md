#  Linux Commnad ebook-convert 转换电子书



---
![在这里插入图片描述](https://img-blog.csdnimg.cn/b4d5304777df40e09c5072abb4c6f8f6.gif#pic_center)


## 1. 简介
Ebook-convert 命令允许您将电子书从一种格式转换为另一种格式。您还可以在转换为另一种格式时更改电子书的外观。外观选项允许您更改字体属性、设置自定义 CSS 样式、更改内容对齐方式、删除现有 CSS 样式、嵌入字体、删除空行、修改缩进、更改边距、修改行高和替换标点字符。ebook-convert 命令还允许您使用名为启发式处理的选项更改书籍结构和布局。您可以使用此选项删除连字符、修改标题、居中内容、更改错误缩进、删除空白段落等。

Ebook-convert 命令也可用于搜索和替换电子书的内容。它的一个选项还允许您更改封面图像。您还可以使用它来修改书籍元数据，如作者姓名、标题、出版年份等。它还包括一个修改目录的选项。以下是 ebook-convert 命令的一些示例：

##  2. 安装

```bash
 apt install calibre
```
安装后，您现在应该在系统上拥有 `ebook-convert` 和 `ebook-polish` 命令

##  3. 实例
### 3.1 epub 转换与 Kindle 兼容的 azw3 

```bash
ebook-convert file.epub file.azw3
```
如果您从本地文件夹执行 ebook-convert 命令的预编译二进制文件，请运行以下命令：

```bash
 ./ebook-convert file.epub file.azw3
```
###   3.2 .epub 转换 .mobi 

```c
ebook-convert file.epub file.mobi
```
要更改电子书内容的对齐方式，请使用以下格式的命令：

```bash
 ebook-convert file.epub file.azw3 --change-justification justify
```
 `–change-justification` 参数接受 `left` 、 `right` 、 `original` 和 `justify` 作为可能的值。您可以从这里了解有关所有外观和感觉选项的[更多信息](https://manual.calibre-ebook.com/generated/en/ebook-convert.html#look-and-feel)。如果您想一次使用多个选项，请使用以下格式的命令：

```bash
ebook-convert file.epub file.azw3 --change-justification justify --remove-paragraph-spacing
```

如果您想使用启发式处理功能，您必须先启用它
```bash
ebook-convert file.epub file.azw3 --enable-heuristics --disable-dehyphenate
```

##  4. Ebook-polish 命令
ebook-poilsh 命令仅适用于 epub 和 azw3 文件格式。它可用于修改现有电子书文件的属性和样式。与 ebook-convert 命令不同，它不会将电子书文件转换为另一种格式，而是对作为参数提供的现有电子书文件进行更改。

您可以使用 ebook-polish 命令添加和删除软连字符、更改封面图像、嵌入自定义字体、压缩图像等。下面是一个带有多个选项的 ebook-polish 命令示例，其中 `file.epub` 被抛光为 `polish_file.epub` 文件。

```bash
ebook-polish --add-soft-hyphens --upgrade -book file.epub Polish_file.epub
```

### 4.1  一次批量转换多个电子书文件的脚本
可以一次将多个 epub 文件批量转换为azw3文件

```bash
#! /bin/bash


function convert () {

        filename="$1"

        extension="${filename##*.}"

        root="${filename%.*}"

        outputExtension=".azw3"

        convertedName="${root}_converted${outputExtension}"

        polishedName="${root}_converted_and_polished${outputExtension}"


        echo ""

        echo "++++++ Converting book: $filename ++++++"

        ./ebook-convert "$filename" "$convertedName" \

        --change-justification justify \

        --margin-left 0 \

        --margin-right 0 \

        --margin-top 0 \

        --margin-bottom 0 \

        --remove-paragraph-spacing \

        --remove-paragraph-spacing-indent-size 1.0 \

        --filter-css font-family

        sleep 0.1


        echo ""

        echo "++++++ Polishing book: $convertedName ++++++"

        ./ebook-polish --add-soft-hyphens --upgrade-book "$convertedName" "$polishedName"

        sleep 0.1


        echo ""

        echo "++++++ Removing obsolete file: $convertedName ++++++"

        rm "$convertedName"

        sleep 0.1


        echo ""

        echo "++++++ Done, final book is: $polishedName ++++++"

        echo ""

}


for name in "$@"

do

        echo "++++++ Staring conversion of: "$name" ++++++"

            convert "$name"

            sleep 0.1

done
```
执行：

```bash
$ ./convert_epub_to_azw3.sh file.epub

$ ./convert_epub_to_azw3.sh *.epub
```

参考：

 - [ebook-convert command](https://manpages.ubuntu.com/manpages/bionic/man1/ebook-convert.1.html#name) 
 - [Convert and Optimize Ebooks in Linux](https://linuxhint.com/convert-and-optimize-ebooks-in-linux/)

