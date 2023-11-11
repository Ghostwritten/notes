F、X和P，可以用在Describe、Context、It等任何包含测试例的模块

 - F含义Focus，使用后表示只执行该模块包含的测试，当里层和外层都存在Focus时，外层的无效，即下面代码只会执行B测试用例
 - P的含义是Pending，即不执行，用法和F一样，规则的外层的生效
 - X和P的含义一样
 - 还有一个跳过测试例的方式是在代码中加Skip


### pending
您可以将单个Spec或容器标记为待定。这将阻止Spec（或者容器中的Specs）运行。您可以在您的Describe, Context, It 和 Measure前面添加一个P或者一个X来实现这一点：PDescribe("some behavior", func() { ... })

```go
PContext("some scenario", func() { ... })
PIt("some assertion")
PMeasure("some measurement")

XDescribe("some behavior", func() { ... })
XContext("some scenario", func() { ... })
XIt("some assertion")
XMeasure("some measurement")
```
当您标记一个It或者Meature为Pending态时，您不必删掉fun() {...}。 Ginkgo 会自动忽略字符串后面的任何参数。
默认，Ginkgo将会打出每一个处于Pending态的Spec的说明。您可以通过设置--noisyPendings=false标签来关闭它。
默认，Ginkgo不会因为有处于Pending态的规格而导致失败。您可以通过设置--failOnPending标签来改变它。
在编译时，使用P和X将规格标记为Pending态。如果您需要在运行时（可能是由于只能在运行时才知道约束）跳过一个规格。您可以在您的测试中调用Skip：

```go
It("should do something, if it can", func() {
    if !someCondition {
        Skip("special condition wasn't met")
    }

    // assertions go here
})
```
默认地，Ginkgo将会为每一个跳过的规格打印输出一份说明。您可以通过设置--noisySkippings=false标签来关闭它。
注意：Skip(...)导致闭包退出，所以没有必要返回它。
