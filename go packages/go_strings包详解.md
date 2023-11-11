
扩展：
[玩转字符串处理操作](https://www.jb51.net/article/175919.htm)

```bash
[root@localhost strings]# cat strings.go
package main

import (
        "bytes"
        "fmt"
        "strings"
)

func main() {
        s := "Hello,World!!!!!"

        //Count计算字符串sep在s中的非重叠个数：2
        //func Count(s, sep string) int
        fmt.Println(strings.Count(s, "!!"))

        //Contains判断字符串s中是否包含子串substr：true
        //func Contains(s, substr string) bool
        fmt.Println(strings.Contains(s, "Hello"))

        //ContainsAny判断字符串s中是否包含chars中的任何一个字符：true
        //func ContainsAny(s, chars string) bool
        fmt.Println(strings.ContainsAny(s, "HJK"))

        //ContainsRune判断字符串s中是否包含字符r：true
        //Go语言的单引号一般用来表示「rune literal」，即码点字面量
        //func ContainsRune(s string, r rune) bool
        fmt.Println(strings.ContainsRune(s, 'H'))

        //Index返回子串sep在字符串s中第一次出现的位置,如果找不到，则返回-1：6
        //func Index(s, sep string) int
        fmt.Println(strings.Index(s, "W"))

        //LastIndex返回子串sep在字符串s中最后一次出现的位置,如果找不到，则返回-1：9
        //func LastIndex(s, sep string) int
        fmt.Println(strings.LastIndex(s, "l"))

        //IndexRune返回字符r在字符串s中第一次出现的位置,如果找不到，则返回-1：6
        //func IndexRune(s string, r rune) int
        fmt.Println(strings.IndexRune(s, 'W'))

        //IndexAny返回字符串chars中的任何一个字符在字符串s中第一次出现的位置：8
        //如果找不到，则返回-1
        //func IndexAny(s, chars string) int
        fmt.Println(strings.IndexAny(s, "rd"))

        //LastIndexAny返回字符串chars中的任何一个字符在字符串s中最后一次出现的位置：10
        //如果找不到，则返回-1
        //func LastIndexAny(s, chars string) int
        fmt.Println(strings.LastIndexAny(s, "rd"))

        //Split以sep为分隔符，将s切分成多个子切片：[Hello World!!!!!]
        //func Split(s, sep string) []string
        fmt.Println(strings.Split(s, ","))

        //Join将a中的子串连接成一个字符串，用sep分隔：Monday|Tuesday|Wednesday
        //func Join(a []string, sep string) string
        ss := []string{"Monday", "Tuesday", "Wednesday"}
        fmt.Println(strings.Join(ss, "|"))

        //HasPrefix判断字符串s是否以prefix开头：true
        //func HasPrefix(s, prefix string) bool
        fmt.Println(strings.HasPrefix(s, "He"))

        //HasSuffix判断字符串s是否以prefix结尾：true
        //func HasSuffix(s, suffix string) bool
        fmt.Println(strings.HasSuffix(s, "!!"))

        //Repeat将count个字符串s连接成一个新的字符串
        //func Repeat(s string, count int) string
        fmt.Println(strings.Repeat(s, 2))

        //ToUpper将s中的所有字符修改为其大写格式：HELLO,WORLD!!!!!
        //func ToUpper(s string) string
        fmt.Println(strings.ToUpper(s))

        //ToLower将s中的所有字符修改为其小写格式：hello,world!!!!!
        //func ToLower(s string) string
        fmt.Println(strings.ToLower(s))

        //Trim将删除s首尾连续的包含在cutset中的字符：世界
        //func Trim(s string, cutset string) string
        sss := " Hello 世界! "
        fmt.Println(strings.Trim(sss, " Helo!"))

        //TrimLeft将删除s头部连续的包含在cutset中的字符：世界！
        //func TrimLeft(s string, cutset string) string
        fmt.Println(strings.TrimLeft(sss, " Helo"))

        //TrimRight将删除s尾部连续的包含在cutset中的字符： Hello
        //func TrimRight(s string, cutset string) string
        fmt.Println(strings.TrimRight(sss, " 世界!"))

        //TrimSpace将删除s首尾连续的的空白字符：Hello 世界!
        //func TrimSpace(s string) string
        fmt.Println(strings.TrimSpace(sss))

        //TrimPrefix删除s头部的prefix字符串：,World!!!!!
        //func TrimPrefix(s, prefix string) string
        fmt.Println(strings.TrimPrefix(s, "Hello"))

        //TrimSuffix删除s尾部的suffix字符串：Hello,World!
        //func TrimSuffix(s, suffix string) string
        fmt.Println(strings.TrimSuffix(s, "!!!!"))

        //Replace返回s的副本，并将副本中的old字符串替换为new字符串
        //替换次数为 n 次，如果 n 为 -1，则全部替换
        //func Replace(s, old, new string, n int) string
        fmt.Println(strings.Replace(s, "!", "|", -1))

        //EqualFold判断s和t是否相等。忽略大小写，同时它还会对特殊字符进行转换：true
        //比如将“ϕ”转换为“Φ”、将“Ǆ”转换为“ǅ”等，然后再进行比较
        //func EqualFold(s, t string) bool
        s1 := "Hello 世界! ϕ Ǆ"
        s2 := "hello 世界! Φ ǅ"
        fmt.Println(strings.EqualFold(s1, s2))

        //通过字符串s创建strings.Reader对象
        //func NewReader(s string) *Reader { return &Reader{s, 0, -1} }
        r := strings.NewReader(s)

        //Len返回r.i之后的所有数据的字节长度:16
        //func (r *Reader) Len() int
        fmt.Println(r.Len())

        //Read将r.i之后的所有数据写入到b中（如果b的容量足够大）
        //func (r *Reader) Read(b []byte) (n int, err error)
        b := make([]byte, 5)
        for n, _ := r.Read(b); n > 0; n, _ = r.Read(b) {
                fmt.Println(string(b[:n])) //"Hello", ",Worl", "d!!!!", "!",
        }

        //ReadAt将off之后的所有数据写入到b中（如果 b 的容量足够大）
        //func (r *Reader) ReadAt(b []byte, off int64) (n int, err error)
        n, _ := r.ReadAt(b, 6)
        fmt.Println(string(b[:n])) //World

        //ReadByte将r.i之后的一个字节写入到返回值b中
        //func (r *Reader) ReadByte() (b byte, err error)
        r.ReadByte()

        //Seek用来移动r中的索引位置，offset：要移动的偏移量，负数表示反向移动
        //whence：从那里开始移动，0：起始位置，1：当前位置，2：结尾位置
        //func (r *Reader) Seek(offset int64, whence int) (int64, error)
        r1 := strings.NewReader(s)
        b1 := make([]byte, 5)
        r1.Seek(6, 0) // 移动索引位置到第7个字节
        r1.Read(b1)   // 开始读取
        fmt.Printf("%q\n", b1)
        r1.Seek(-5, 1) // 将索引位置移回去
        r1.Read(b1)    // 继续读取
        fmt.Printf("%q\n", b1)

        //WriteTo 将 r.i 之后的数据写入接口 w 中
        //func (r *Reader) WriteTo(w io.Writer) (n int64, err error)
        r2 := strings.NewReader(s)
        buf := bytes.NewBuffer(nil)
        r2.WriteTo(buf)
        fmt.Printf("%q\n", buf) //"Hello,World!!!!!"

        //NewReplacer 通过“替换列表”创建一个 Replacer 对象。
        //按照“替换列表”中的顺序进行替换，只替换非重叠部分。
        //如果参数的个数不是偶数，则抛出异常。
        //func NewReplacer(oldnew ...string) *Replacer

        //Replace 返回对 s 进行“查找和替换”后的结果
        //Replace 使用的是 Boyer-Moore 算法，速度很快
        //func (r *Replacer) Replace(s string) string
        srp := strings.NewReplacer("Hello", "你好", "World", "世界", "!", "！")
        s = "Hello World!Hello World!hello world!"
        rst := srp.Replace(s)
        fmt.Print(rst) //你好 世界！你好 世界！hello world！
}
```
```go
[root@localhost strings]# go run strings.go
2
true
true
true
6
9
6
8
10
[Hello World!!!!!]
Monday|Tuesday|Wednesday
true
true
Hello,World!!!!!Hello,World!!!!!
HELLO,WORLD!!!!!
hello,world!!!!!
世界
世界! 
 Hello
Hello 世界!
,World!!!!!
Hello,World!
Hello,World|||||
true
16
Hello
,Worl
d!!!!
!
World
"World"
"World"
"Hello,World!!!!!"
你好 世界！你好 世界！hello world！
```
##    strings.Split

```bash
s := strings.Split("", "")
    fmt.Println(s, len(s))
    s = strings.Split("abc,abc", "")
    fmt.Println(s, len(s))
    s = strings.Split("", ",")
    fmt.Println(s, len(s))
    s = strings.Split("abc,abc", ",")
    fmt.Println(s, len(s))
    s = strings.Split("abc,abc", "|")
    fmt.Println(s, len(s))
    fmt.Println(len(""))
    fmt.Println(len([]string{""}))
    str := ""
    fmt.Println(str[0])
```
输出

```bash
[] 0
[a b c , a b c] 7
[] 1
[abc abc] 2
[abc,abc] 1
0
1
panic: runtime error: index out of range

goroutine 1 [running]:
main.main()
        D:/gitlab/cronserver/test/test.go:22 +0x5f6
exit status 2
```

