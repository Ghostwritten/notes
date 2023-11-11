## fmt介绍
软件包fmt实现了格式化的I / O，其功能类似于C的printf和scanf。格式'动词'来自C，但更简单。
## 占位符
### 通用
| 占位符  | 说明                  | 示例                       | 输出                        |
|------|---------------------|--------------------------|---------------------------|
| %v   | 相应值的默认格式            | Printf\("%v",person \)   | \{zhangsan\}              |
| %\+v | 类似%v，但输出结构体时会添加字段名式 | Printf\("%\+v",person \) | \{Name:zhangsan\}         |
| %\#v | 相应值的Go语法表示          | Printf\("\#v",person \)  | main\.Person=\{zhangsan\} |
| %T   | 相应值的类型的Go语法表示       | Printf\("%T",person \)   | main\.Person              |
| %%   | 字面上的百分号，并非值的占位符     | Printf\("%%"\)           | %                         |
### 布尔值
| 布尔占位符 | 说明           | 示例                  | 输出   |
|-------|--------------|---------------------|------|
| %t    | 单词true或false | Printf\("%t",true\) | true |

### 整数
| 占位符 | 说明                                | 示例                    | 输出      |
|-----|-----------------------------------|-----------------------|---------|
| %b  | 二进制表示                             | Printf\("%b",5\)      | 101     |
| %c  | 该值对应的unicode码值                    | Printf\("%c",0x4E2d\) | 中       |
| %d  | 十进制表示                             | Printf\("%d",0x12\)   | 18      |
| %o  | 八进制表示                             | Printf\("%o",10\)     | 12      |
| %q  | 单引号围绕的字符字面值，由Go语法安全的转译            | Printf\("%q",0x4E2d\) | '中'     |
| %x  | 十六进制表示，字母形式为小写a\-f                | Printf\("%x",13\)     | d       |
| %X  | 十六进制表示，字母形式为大写A\-F                | Printf\("%X",13\)     | D       |
| %U  | 表示为Unicode格式：U\+1234，等价于"U\+%04X" | Printf\("%U",0x4E2d\) | U\+4E2D |
### 浮点数与复数的两个组分
| 占位符 | 说明                                                  | 示例                    | 输出                    |
|-----|-----------------------------------------------------|-----------------------|-----------------------|
| %b  | 无小数部分、指数为二的幂的科学计数法，与strconv\.FormatFloat的'b'转换格式一致。 | Printf\("%b",10\.20\) | 5742089524897382p\-49 |
| %e  | 科学计数法，如\-1234\.456e\+78                             | Printf\("%e",10\.20\) | 1\.020000e\+01        |
| %E  | 科学计数法，如\-1234\.456E\+78                             | Printf\("%E",10\.20\) | 1\.020000E\+01        |
| %f  | 有小数部分但无指数部分，如123\.456                               | Printf\("%f",10\.20\) | 10\.200000            |
| %g  | 根据实际情况采用%e或%f格式（以获得更简洁、准确的输出）                       | Printf\("%g",10\.20\) | 10\.2                 |
| %G  | 根据实际情况采用%E或%F格式（以获得更简洁、准确的输出）                       | Printf\("%G",10\.20\) | \(10\.2\+2i\)         |
### 字符串和[]byte
| 占位符 | 说明                           | 示例                                | 输出           |
|-----|------------------------------|-----------------------------------|--------------|
| %s  | 输出字符串表示\(string类型或\[\]byte\) | Printf\("%s",\[\]byte\("Go语言"\)\) | Go语言         |
| %q  | 双引号围绕的字符串，由Go语法安全的转译         | Printf\("%q","Go语言"\)             | "Go语言"       |
| %x  | 十六进制，小写字母，每字节两个字符            | Printf\("%x","golang"\)           | 676f6c616e67 |
| %X  | 十六进制，大写字母，每字节两个字符            | Printf\("%X","golang"\)           | 676F6C616E67 |
### 指针
| 占位符 | 说明           | 示例                     | 输出           |
|-----|--------------|------------------------|--------------|
| %P  | 十六进制表示，前缀 0x | Printf\("%p",&person\) | 0xc0420341c0 |
### 其他
| 占位符 | 说明                                                    | 示例                            | 输出               |
|-----|-------------------------------------------------------|-------------------------------|------------------|
| \+  | 总是输出数值的正负号；对%q（%\+q）会生成全部是ASCII字符的输出（通过转义）            | Printf\("%\+q","中文"\)         | "\\u4e2d\\u6587" |
| \-  | 在输出右边填充空白而不是默认的左边（即从默认的右对齐切换为左对齐）；                    |                               |                  |
| \#  | 切换格式：八进制数前加0（%\#o）                                    | Printf\("%\#0",46\)           |                  |
|     | 十六进制数前加0x（%\#x）或0X（%\#X）                              | Printf\("%\#x",46\)           | 0x2e             |
|     | 指针去掉前面的0x（%\#p）；）                                     | fmt\.Printf\("%\#p",&person\) | c0420441b0       |
|     | 对%q（%\#q），如果strconv\.CanBackquote返回真会输出反引号括起来的未转义字符串； | Printf\("%\#q",'中'\)          | '中'              |
|     | 对%U（%\#U），如果字符是可打印的，会在输出Unicode格式、空格、单引号括起来的go字面值；    | Printf\("%\#U",'中'\)          | U\+4E2D '中'      |
| ' ' | \(空格\)为数值中省略的正负号流出空白\(% d\);                          | Printf\("% d",16\)            |  16              |
|     | 以十六进制\(% x,% X\)打印字符串或切片时，在字节之间用空格隔开                  | Printf\("% x","abc"\)         | 61 62 63         |
| 0   | 使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面                        |                               |                  |
## 用法详解
[Errorf](https://godoc.org/fmt#example-Errorf)
[Fprint](https://godoc.org/fmt#example-Fprint)
[Fprintf](https://godoc.org/fmt#example-Fprintf)
[Fprintln](https://godoc.org/fmt#example-Fprintln)
[Fscanf](https://godoc.org/fmt#example-Fscanf)
[Fscanln](https://godoc.org/fmt#example-Fprintln)
[GoStringer](https://godoc.org/fmt#example-GoStringer)
[Print](https://godoc.org/fmt#example-Print)
[Printf](https://godoc.org/fmt#example-Println)
[Println](https://godoc.org/fmt#example-Println)
[Sprint](https://godoc.org/fmt#example-Sprint)
[Sprintf](https://godoc.org/fmt#example-Sprintf)
[Sprintln](https://godoc.org/fmt#example-Sprintln)
[Sscanf](https://godoc.org/fmt#example-Sscanf)
[Stringer](https://godoc.org/fmt#example-Stringer)
[package (Formats)](https://godoc.org/fmt#example-package--Formats)
[package (Printers)](https://godoc.org/fmt#example-package--Printers)
