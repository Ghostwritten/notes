#  Go 如何获取文件的大小

![](https://i-blog.csdnimg.cn/blog_migrate/9b665d4b35e14aa0837a45e5a2fda415.png)




## 1. Read字节方式
第一种，是最直观会想到的，也就是打开文件，把文件读取一遍。

```bash
func main() {
    file,err:=os.Open("water")
    if err ==nil {
        sum := 0
        buf:=make([]byte,2014)
        for  {
            n,err:=file.Read(buf)
            sum+=n
            if err==io.EOF {
                break
            }
        }
        fmt.Println("file size is ",sum)
    }
}
```

这种方式需要打开文件，通过for循环读取文件的字节内容，然后算出文件的大小，这样时也是最不能用的办法，因为效率低，代码量大。

## 2. ioutil方式
上面的代码比较啰嗦，这时候我们可能想到了使用ioutil包的ReadFile来代替，直接获得文件的内容，进而计算出文件的大小。

```bash
func main() {
    content,err:=ioutil.ReadFile("water")
    if err == nil {
        fmt.Println("file size is ",len(content))
    }
}
```

通过ioutil.ReadFile函数，我们三行代码就可以搞定，的确方便很多，但是效率慢的问题依然，存在，如果是个很大的文件呢？

## 3. Stat方法
继续再进一步，我们不读取文件的内容来计算了，我们通过文件的信息

```bash
func main() {
    file,err:=os.Open("water")

    if err == nil {
        fi,_:=file.Stat()
        fmt.Println("file size is ",fi.Size())
    }
}
```

这种方式不会再读取文件的内容，而是通过Stat方法直接获取，速度会非常快，尤其对于大文件尤其有用。但是它还不是我们今天要讲的终极办法，因为它还是会打开文件，会占用它。

## 4. 终极方案
好了，我们的终极方案终于要登场了，他的代码也非常简单。

```bash
func main() {
    fi,err:=os.Stat("water")
    if err ==nil {
        fmt.Println("file size is ",fi.Size(),err)
    }
}
```

是的，也只需要三行代码即可实现，这里使用的是os.Stat，通过他可以获得文件的元数据信息，现在我们看看它能获取到哪些信息。

## 5. 获取文件信息
通过os.Stat方法，我们可以获取文件的信息，比如文件大小、名字等。

```bash
func main() {
    fi,err:=os.Stat("water")
    if err ==nil {
        fmt.Println("name:",fi.Name())
        fmt.Println("size:",fi.Size())
        fmt.Println("is dir:",fi.IsDir())
        fmt.Println("mode::",fi.Mode())
        fmt.Println("modTime:",fi.ModTime())
    }
}
```

运行这段代码看下结果:

```bash
name: water
size: 403
is dir: false
mode:: -rw-r--r--
modTime: 2018-05-06 18:52:07 +0800 CST
```

以上就是可以获取到的文件信息，还包括判断是否是目录，权限模式和修改时间。所以我们对于文件的信息获取要使用os.Stat函数，它可以在不打开文件的情况下，高效获取文件信息。

## 6. 判断文件是否存在
`os.Stat`函数有两个返回值，一个是文件信息，一个是err，通过err我们可以判断文件是否存在。

首先，`err==nil`的时候，文件肯定是存在的；其次`err!=nil`的时候也不代表不存在，这时候我们就需要进行严密的判断。

```bash
func main() {
    _,err:=os.Stat(".")
    if err ==nil {
        fmt.Println("file exist")
    }else if os.IsNotExist(err){
        fmt.Println("file not exist")
    }else{
        fmt.Println(err)
    }
}
```

通过os.IsNotExist来判断一个文件不存在。最后else的可能性比较少，这个时候可以看下具体的错误是什么，再根据错误来判断文件是否存在。

## 7. 小结
`os.Stat`是一个非常好的函数，可以让我们非常高效的获取文件信息，所以在项目中尽可能的使用它。


