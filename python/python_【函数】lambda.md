```c
# Python 中的 lambda 关键字提供了# 用于声明小型和
# 匿名函数的快捷方式：


>>> add =  lambda x, y: x + y
 >>> add( 5 , 3 )
 8


# 你可以用 def 关键字声明相同的 add() # 函数：

>>>  def  add (x, y):
 ...     返回x + y
 >>> add( 5 , 3 )
 8

# 那么有什么大惊小怪的呢？
# Lambda 是*函数表达式*：
>>> ( lambda x, y: x + y)( 5 , 3 )
 8

# • Lambda 函数是单表达式
# 函数，不一定要绑定
到名称（它们可以是匿名的）。

# • Lambda 函数不能使用常规
的# Python 语句，并且始终包含
# 隐式`return` 语句。
```

