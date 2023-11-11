常用模块列表：
| 模块             | 功能描述                                                 |
|----------------|------------------------------------------------------|
| It             | It 包含的代码为一个测试用例                                      |
| Specify        | 和 It 功能一致， 可作为别名在适当位置替换 It                           |
| Describe       | 将一个或多个测试用例归类,描述一種行為或者一個方法                                         |
| Context        | 同 Describe ,增加條件語句，儘可能全地覆蓋各種condition                                          |
| BeforeEach     | 每个测试用例执行前执行该段代码                                      |
| AftreEach      | 每个测试用例执行后执行该段代码                                      |
| JustBeforeEach | 在 BeforeEach 之后，执行测试用例之前执行                           |
| JustAfterEach  | 在执行测试用例之后，AftreEach之前执行                              |
| BeforeSuite    | 在该测试集执行前执行， package\_suite\_test\.go 中 RunSpecs 执行之前 |
| AfterSuite     | 无论是否有任何测试失败，该函数都会在测试集执行后运行                           |
| By             | 打印信息（字符串），只会在测试例失败后打印，一般用于调试和定位问题                    |
| Fail           | 标志该测试例运行结果为失败，并打印里面的Expect
| expect |這個是gomega提供的方法，用來斷言結果 |
Ginkgo可以轻松编写富有表现力的规格，以有条理的方式描述代码的行为。您可以使用Describe 和 Context容器来组织你的It规格，使用BeforeEach和AfterEach来搭建和拆除测试中的常见设置
### 单个规格：It
您可以通过在Describe或Context容器块中设置It块来添加单个规格：

```go
var _ = Describe("Book", func() {
    It("can be loaded from JSON", func() {
        book := NewBookFromJSON(`{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`)

        Expect(book.Title).To(Equal("Les Miserables"))
        Expect(book.Author).To(Equal("Victor Hugo"))
        Expect(book.Pages).To(Equal(1488))
    })
})
```

### 指定别名: Specify
为了确保您的规格阅读自然，Specify，PSpecify，XSpecify和FSpecify块可用作别名，以便在相应的It替代品看起来不像自然语言的情况下使用。
Specify块的行为与It块相同，可以在It块（以及PIt，XIt和FIt块）的地方使用。
Specify替换It的示范如下：

```go
Describe("The foobar service", func() {
  Context("when calling Foo()", func() {
    Context("when no ID is provided", func() {
      Specify("an ErrNoID error is returned", func() {
      })
    })
  })
})
```
### 提取通用步骤：BeforeEach
您可以使用BeforeEach块在多个测试用例中去除重复的步骤以及共享通用的设置：

```go
var _ = Describe("Book", func() {
    var book Book

    BeforeEach(func() {
        book = NewBookFromJSON(`{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`)
    })

    It("can be loaded from JSON", func() {
        Expect(book.Title).To(Equal("Les Miserables"))
        Expect(book.Author).To(Equal("Victor Hugo"))
        Expect(book.Pages).To(Equal(1488))
    })

    It("can extract the author's last name", func() {
        Expect(book.AuthorLastName()).To(Equal("Hugo"))
    })
})
```
BeforeEach在每个规格之前运行，从而确保每个规格都具有状态的原始副本。使用闭包变量共享公共状态（在本例中为var book Book）。您还可以在AfterEach块中执行清理操作。

在BeforeEach和AfterEach块中设置断言也很常见。例如，这些断言，可以断言在为规格准备状态时没有发生错误。
### 使用容器组织规格：Describe 和 Context
Ginkgo允许您使用Describe和Context容器在套件中富有表现力的组织规格：

```go
var _ = Describe("Book", func() {
    var (
        book Book
        err error
    )

    BeforeEach(func() {
        book, err = NewBookFromJSON(`{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`)
    })

    Describe("loading from JSON", func() {
        Context("when the JSON parses succesfully", func() {
            It("should populate the fields correctly", func() {
                Expect(book.Title).To(Equal("Les Miserables"))
                Expect(book.Author).To(Equal("Victor Hugo"))
                Expect(book.Pages).To(Equal(1488))
            })

            It("should not error", func() {
                Expect(err).NotTo(HaveOccurred())
            })
        })

        Context("when the JSON fails to parse", func() {
            BeforeEach(func() {
                book, err = NewBookFromJSON(`{
                    "title":"Les Miserables",
                    "author":"Victor Hugo",
                    "pages":1488oops
                }`)
            })

            It("should return the zero-value for the book", func() {
                Expect(book).To(BeZero())
            })

            It("should error", func() {
                Expect(err).To(HaveOccurred())
            })
        })
    })

    Describe("Extracting the author's last name", func() {
        It("should correctly identify and return the last name", func() {
            Expect(book.AuthorLastName()).To(Equal("Hugo"))
        })
    })
})
```
您可以使用Describe块来描述代码的各个行为，Context块在不同情况下执行这些行为。在此示例中，我们Describe从JSON加载书籍并指定两个Contexts：当JSON成功解析时以及JSON无法解析时。除了语义差异，两种容器类型具有相同的行为。

当嵌套Describe和Context块时，It执行时，围绕It的所有容器节点的BeforeEach块，从最外层到最内层运行。

 - 注意：每个It块都运行BeforeEach和AfterEach块。这确保了每个规格的原始状态。
 - 通常，容器块中的唯一代码应该是It块或BeforeEach / JustBeforeEach / JustAfterEach / AfterEach块或闭包变量声明。在容器块中进行断言通常是错误的。
 - 在容器块中初始化闭包变量也是错误的。如果你的一个It改变了这个变量，后期It将会收到改变后的值。这是一个测试污染的案例，很难追查。

始终在BeforeEach块中初始化变量。
如果您想在运行时获取有关当前测试的信息，您可以在任何It或BeforeEach / JustBeforeEach/JustAfterEach / AfterEach块中使用CurrentGinkgoTestDescription()。
此次调用CurrentGinkgoTestDescription返回包含有关当前运行的测试的各种信息，包括文件名，行号，It块中的文本以及周围容器块中的文本。
### 分离创建和配置：JustBeforeEach
上面的例子说明了BDD风格测试中常见的反模式。我们的顶级BeforeEach使用有效的JSON创建了一个新的book,但是较低级别的Context使用无效的JSON创建的book执行。这使我们重新创建并覆盖原始的book.幸运的是，使用Ginkgo的JustBeforeEach块，这些代码重复是不必要的。

JustBeforeEach 块保证在所有BeforeEach 块运行之后，并且在It块运行之前运行。我们可以使用这个特性来清除Book规格：

```go
var _ = Describe("Book", func() {
    var (
        book Book
        err error
        json string
    )

    BeforeEach(func() {
        json = `{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`
    })

    JustBeforeEach(func() {
        book, err = NewBookFromJSON(json)
    })

    Describe("loading from JSON", func() {
        Context("when the JSON parses succesfully", func() {
            It("should populate the fields correctly", func() {
                Expect(book.Title).To(Equal("Les Miserables"))
                Expect(book.Author).To(Equal("Victor Hugo"))
                Expect(book.Pages).To(Equal(1488))
            })

            It("should not error", func() {
                Expect(err).NotTo(HaveOccurred())
            })
        })

        Context("when the JSON fails to parse", func() {
            BeforeEach(func() {
                json = `{
                    "title":"Les Miserables",
                    "author":"Victor Hugo",
                    "pages":1488oops
                }`
            })

            It("should return the zero-value for the book", func() {
                Expect(book).To(BeZero())
            })

            It("should error", func() {
                Expect(err).To(HaveOccurred())
            })
        })
    })

    Describe("Extracting the author's last name", func() {
        It("should correctly identify and return the last name", func() {
            Expect(book.AuthorLastName()).To(Equal("Hugo"))
        })
    })
})
```
现在，对每一个It，book实际上只创建一次。这个失败的JSON上下文可以简单地将无效的json值分配给BeforeEach中的json变量。
抽象地，JustBeforeEach允许您将创建与配置分离。使用由BeforeEach链指定和修改的配置在JustBeforeEach中进行创建。
您可以在不同的嵌套级别使用多个JustBeforeEach。Ginkgo将首先从外部运行所有的BeforeEach，然后它将从外部运行JustBeforeEach。虽然功能强大，但这可能会导致测试套件混乱 - 因此请谨慎使用嵌套的JustBeforeEach。
**一些建议：JustBeforeEach是一个很容易被滥用的强大工具。好好利用它。**

### 分离诊断收集和销毁：JustAfterEach
在销毁（可能会破坏有用的状态）之前，在每一个It块之后，有时运行一些代码是很有用的。比如，测试失败后，执行一些诊断的操作。我们可以在上面的示例中使用它来检查测试是否失败，如果失败，则输出实际的book：

```go
 JustAfterEach(func() {
        if CurrentGinkgoTestDescription().Failed {
            fmt.Printf("Collecting diags just after failed test in %s\n", CurrentGinkgoTestDescription().TestText)
            fmt.Printf("Actual book was %v\n", book)
        }
    })
```
您可以在不同的嵌套级别使用多个JustAfterEach。Ginkgo将首先从内到外运行所有JustAfterEach，然后它将从内到外运行AfterEach。虽然功能强大，但这会导致测试套件混乱 - 因此合理地使用嵌套的JustAfterEach。
就像JustBeforeEach一样，JustAfterEach是一个很容易被滥用的强大工具。好好利用它。

### 全局设置和销毁：BeforeSuite 和 AfterSuite
时您希望在整个测试之前运行一些设置代码和在整个测试之后运行一些清理代码。例如，您可能需要启动并销毁外部数据库。
Ginkgo提供了BeforeSuite和AfterSuite来实现这一点。通常，您可以在引导程序文件的顶层定义它们。例如，假设您需要设置外部数据库：

```go
package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"

    "your/db"

    "testing"
)

var dbRunner *db.Runner
var dbClient *db.Client

func TestBooks(t *testing.T) {
    RegisterFailHandler(Fail)

    RunSpecs(t, "Books Suite")
}

var _ = BeforeSuite(func() {
    dbRunner = db.NewRunner()
    err := dbRunner.Start()
    Expect(err).NotTo(HaveOccurred())

    dbClient = db.NewClient()
    err = dbClient.Connect(dbRunner.Address())
    Expect(err).NotTo(HaveOccurred())
})

var _ = AfterSuite(func() {
    dbClient.Cleanup()
    dbRunner.Stop()
})
```
BeforeSuite 函数在任何规格运行之前运行。如果BeforeSuite运行失败则没有规格将会运行，测试套件运行结束。
AfterSuite函数在所有的规格运行之后运行，无论是否有任何测试的失败。由于AfterSuite通常有一些代码来清理持久的状态，所以当你使用control+c 打断运行的测试时，Ginkgo也将会运行AfterSuite。要退出AfterSuite的运行，再次输入control+c。
通过传递带有Done参数的函数，可以异步运行BeforeSuite和AfterSuite。
您只能在测试套件中定义一次BeforeSuite和AfterSuite（不需要设置多次！）

最后，当并行运行时，每个并行进程都将运行BeforeSuite和AfterSuite函数。在这里查看有关并行运行测试的更多信息。

### 文档化复杂的It：By
按照规则，您应该记录您的It，BeforEach， 等精炼到位。有时这是不可能的，特别是在集成式测试中测试复杂的工作流时。在这些情况下，您的测试块开始隐藏通过单独查看代码难以收集的叙述。在这些情况下，Ginkgo 通过By来提供帮助。这里有一个很好的例子：

```go
var _ = Describe("Browsing the library", func() {
    BeforeEach(func() {
        By("Fetching a token and logging in")

        authToken, err := authClient.GetToken("gopher", "literati")
        Exepect(err).NotTo(HaveOccurred())

        err := libraryClient.Login(authToken)
        Exepect(err).NotTo(HaveOccurred())
    })

    It("should be a pleasant experience", func() {
        By("Entering an aisle")

        aisle, err := libraryClient.EnterAisle()
        Expect(err).NotTo(HaveOccurred())

        By("Browsing for books")

        books, err := aisle.GetBooks()
        Expect(err).NotTo(HaveOccurred())
        Expect(books).To(HaveLen(7))

        By("Finding a particular book")

        book, err := books.FindByTitle("Les Miserables")
        Expect(err).NotTo(HaveOccurred())
        Expect(book.Title).To(Equal("Les Miserables"))

        By("Check the book out")

        err := libraryClient.CheckOut(book)
        Expect(err).NotTo(HaveOccurred())
        books, err := aisle.GetBooks()
        Expect(books).To(HaveLen(6))
        Expect(books).NotTo(ContainElement(book))
    })
})
```
传递给By的字符串是通过GinkgoWriter发出的。如果测试成功，您将看不到Ginkgo绿点之外的任何输出。但是，如果测试失败，您将看到失败之前的每个步骤的打印输出。使用ginkgo -v总是输出所有步骤打印。

By 采用一个可选的fun()类型函数。当传入这样的一个函数时，By将会立刻调用该函数。这将允许您组织您的多个It到一组步骤，但这纯粹是可选的。在实际应用中，每个By函数是一个单独的回调，这一特性限制了这种方法的可用性。

