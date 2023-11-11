## 简介
Ginkgo是一个BDD风格的Go测试框架，旨在帮助您有效地编写富有表现力的综合测试。

## 安装

```go
$ go get github.com/onsi/ginkgo/ginkgo
$ go get github.com/onsi/gomega/...
```

## ginkgo入门测试
```go
$ go env |grep -i gopath
GOPATH="/usr/local/gopath"

$ cd /usr/local/gopath/src
$ mkdir books
$ cd books
$ ginkgo bootstrap
$ ls 
books_suite_test.go
$ cat books_suite_test.go

package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
    "testing"
)

func TestBooks(t *testing.T) {
    RegisterFailHandler(Fail)
    RunSpecs(t, "Books Suite")
}

$ vim books.go
package books

type Book struct {
    Title  string
    Author string
    Pages  int
}

func (b *Book) CategoryByLength() string {

    if b.Pages >= 300 {
        return "NOVEL"
    }

    return "SHORT STORY"
}

$ ginkgo 
Failed to compile books:

go: cannot find main module; see 'go help modules'

Ginkgo ran 1 suite in 53.915004ms
Test Suite Failed

$ go mod init
go: creating new go.mod: module books


$ ginkgo 
Running Suite: Books Suite
==========================
Random Seed: 1595924522
Will run 0 of 0 specs


Ran 0 of 0 Specs in 0.000 seconds
SUCCESS! -- 0 Passed | 0 Failed | 0 Pending | 0 Skipped
PASS

Ginkgo ran 1 suite in 1.028037658s
Test Suite Passed
```
分析：

 - Go允许我们在books包同目录下指定books_test包。使用books_test而不是books可以让我们不破坏books包的封装：你的测试需要导入books包并从外部访问它，就像导入其它任何包一样。这是进入包，测试其内部结构并进行更多行为测试的首选。当然，您可以选择不使用此功能- 只需把package books_test改为package books即可。
 - 通过.将ginkgo和gomega包导入了测试的顶级命名空间
 - TestBooks是一个testing测试。当您运行go test或ginkgo命令时，Go测试运行器将运行此功能。
 - RegisterFailHandler(Fail): Ginkgo 测试通过调用Fail(description string)功能来表示失败。我们使用RegisterFailHandler将此函数传递给Gomega。这是Ginkgo和Gomega之间的唯一连接点。
 - RunSpecs(t *testing.T, suiteDescription string)通知Ginkgo启动测试套件。如果您的任何specs失败，Ginkgo将自动使testing.T失败。


## 给套件添加Specs
一个空的测试套件不是很有趣。虽然您可以开始直接将测试添加到books_suite_test.go中，但您可能更愿意将测试分成单独的文件（特别是对于包含多个文件的包）。让我们为book.go模型添加一个测试文件：

```go
$ ginkgo generate book
Generating ginkgo test for Book in:
  book_test.go
  
$ cat book_test.go
package books_test

import (
	. "github.com/onsi/ginkgo"
	. "github.com/onsi/gomega"

	"books"
)

var _ = Describe("Book", func() {

})
```
分析：

 - ginkgo和gomega包导入顶级命名空间
 - 我们导入books包，因为我们使用特殊的books_test包来将我们的测试与我们的代码隔离开来。
- 方便起见，我们将books包导入命名空间
- 使用Ginkgo的Describe(text string, body func()) bool函数添加了顶级Describe容器。
- var _ = ... 技巧允许我们在顶级评估Describe，而不必将其包装在func init() {}


Describe 中的功能将包含我们的Specs。现在让我们添加一些来测试从JSON中加载books：

books_test.go
```go
package books_test

import (
	. "github.com/onsi/ginkgo"
	. "github.com/onsi/gomega"

	//"books"
)

type Book struct {
    Title  string
    Author string
    Pages  int
}

func (b *Book) CategoryByLength() string {

    if b.Pages >= 300 {
        return "NOVEL"
    }

    return "SHORT STORY"
}



var _ = Describe("Book", func() {
    var (
        longBook  Book
        shortBook Book
    )

    BeforeEach(func() {
        longBook = Book{
            Title:  "Les Miserables",
            Author: "Victor Hugo",
            Pages:  1488,
        }

        shortBook = Book{
            Title:  "Fox In Socks",
            Author: "Dr. Seuss",
            Pages:  24,
        }
    })

    Describe("Categorizing book length", func() {
        Context("With more than 300 pages", func() {
            It("should be a novel", func() {
                Expect(longBook.CategoryByLength()).To(Equal("NOVEL"))
            })
        })

        Context("With fewer than 300 pages", func() {
            It("should be a short story", func() {
                Expect(shortBook.CategoryByLength()).To(Equal("SHORT STORY"))
            })
        })
    })
})

```
分析	：

 - Ginkgo广泛使用闭包（⚠️闭包不是私有，闭的意思不是“封闭内部状态”，而是“封闭外部状态”！），允许您构建描述性测试套件。
- 您应该使用Describe和Context容器来表达性地组织代码的行为。
- 您应该使用Describe和Context容器来表达性地组织代码的行为。
- 为了在BeforeEach和It之间共享状态，您使用闭包变量，通常在最相关的Describe或Context容器的顶部定义。
- 我们使用Gomega的Expect语法来对CategoryByLength()方法产生期望值。


```go
$ ginkgo 
Running Suite: Books Suite
==========================
Random Seed: 1595926303
Will run 2 of 2 specs

••
Ran 2 of 2 Specs in 0.001 seconds
SUCCESS! -- 2 Passed | 0 Failed | 0 Pending | 0 Skipped
PASS

Ginkgo ran 1 suite in 1.108253073s
Test Suite Passed
```


