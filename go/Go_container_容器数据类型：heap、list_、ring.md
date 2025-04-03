#  Go container 容器数据类型：heap、list 、ring
![](https://i-blog.csdnimg.cn/blog_migrate/5c524b614031572ae01642a783858320.png)



该包实现了三个复杂的数据结构：堆，链表，环。 这个包就意味着你使用这三个数据结构的时候不需要再费心从头开始写算法了。

这里的堆使用的数据结构是最小二叉树，即根节点比左边子树和右边子树的所有值都小。 go 的堆包只是实现了一个接口，我们看下它的定义：

```bash
type Interface interface {
    sort.Interface
    Push(x interface{}) // add x as element Len()
    Pop() interface{}   // remove and return element Len() - 1.
}
```
 回顾下 sort.Interface，它需要实现三个方法

 - Len() int
 - Less(i, j int) bool
 - Swap(i, j int)

加上堆接口定义的两个方法

 - Push(x interface{})
 - Pop() interface{}

实例1：
```bash
package main
 
import (
    "container/heap"
    "fmt"
)
 
type IntHeap []int
 
//我们自定义一个堆需要实现5个接口
//Len(),Less(),Swap()这是继承自sort.Interface
//Push()和Pop()是堆自已的接口
 
//返回长度
func (h *IntHeap) Len() int {
    return len(*h);
}
 
//比较大小(实现最小堆)
func (h *IntHeap) Less(i, j int) bool {
    return (*h)[i] < (*h)[j];
}
 
//交换值
func (h *IntHeap) Swap(i, j int) {
    (*h)[i], (*h)[j] = (*h)[j], (*h)[i];
}
 
//压入数据
func (h *IntHeap) Push(x interface{}) {
    //将数据追加到h中
    *h = append(*h, x.(int))
}
 
//弹出数据
func (h *IntHeap) Pop() interface{} {
    old := *h;
    n := len(old);
    x := old[n-1];
    //让h指向新的slice
    *h = old[0: n-1];
    //返回最后一个元素
    return x;
}
 
//打印堆
func (h *IntHeap) PrintHeap() {
    //元素的索引号
    i := 0
    //层级的元素个数
    levelCount := 1
    for i+1 <= h.Len() {
        fmt.Println((*h)[i: i+levelCount])
        i += levelCount
        if (i + levelCount*2) <= h.Len() {
            levelCount *= 2
        } else {
            levelCount = h.Len() - i
        }
    }
}
 
func main() {
    a := IntHeap{6, 2, 3, 1, 5, 4};
    //初始化堆
    heap.Init(&a);
    a.PrintHeap();
    //弹出数据，保证每次操作都是规范的堆结构
    fmt.Println(heap.Pop(&a));
    a.PrintHeap();
    fmt.Println(heap.Pop(&a));
    a.PrintHeap();
    heap.Push(&a, 0);
    heap.Push(&a, 8);
    a.PrintHeap();
}
```

```bash
[root@localhost container]# go run test.go 
[1]
[2 3]
[6 5 4]
1
[2]
[4 3]
[6 5]
2
[3]
[4 5]
[6]
[0]
[3 5]
[6 4 8]
```

