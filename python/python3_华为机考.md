


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a188afb7781a1b0ca5150e605455f2ed.gif#pic_center)

[华为机试](https://www.nowcoder.com/exam/oj/ta?page=1&tpId=37&type=37)

##  HJ1 字符串最后一个单词的长度
描述：计算字符串最后一个单词的长度，单词以空格隔开，字符串长度小于5000。（注：字符串末尾不以空格为结尾）

输入描述：输入一行，代表要计算的字符串，非空，长度小于5000。

输出描述：输出一个整数，表示输入字符串最后一个单词的长度。

示例1

```bash
输入：hello nowcoder
输出：8
```

说明：最后一个单词为nowcoder，长度为8   

代码：cat 1_last_string.py 

```bash

a=input().split()
print(len(a[-1]) if len(a)>1 else len(a[0]))
```
执行:
```bash
$ python3 1_last_string.py 
hello nowcoder
8
```

##  HJ2 计算某字符出现次数
描述：写出一个程序，接受一个由字母、数字和空格组成的字符串，和一个字符，然后输出输入字符串中该字符的出现次数。（不区分大小写字母）

数据范围： 1 \le n \le 1000 \1≤n≤1000 
输入描述：第一行输入一个由字母和数字以及空格组成的字符串，第二行输入一个字符。

输出描述：输出输入字符串中含有该字符的个数。（不区分大小写字母）

示例1

```c
输入：ABCabc
     A
输出：2
```
代码：2_string_num.py 
```bash
x1 = input()
x2 = input() 
x3 = x1.upper() 
x4 = x2.upper() 
n = 0 
for word in x3: 
    if word == x4: 
        n = n + 1 
print(n)
```
执行：
```bash
$ python3 2_string_num.py
ABCabc
A
2
```

##  HJ3 明明的随机数
描述：明明生成了NN个1到500之间的随机整数。请你删去其中重复的数字，即相同的数字只保留一个，把其余相同的数去掉，然后再把这些数从小到大排序，按照排好的顺序输出。

数据范围： `\1≤n≤1000`  ，输入的数字大小满足  `\1≤val≤500` 
输入描述：：第一行先输入随机整数的个数 N 。 接下来的 N 行每行输入一个整数，代表明明生成的随机数。 具体格式可以参考下面的"示例"。
输出描述：：输出多行，表示输入数据处理后的结果

示例1

```bash
输入：
3
2
2
1

输出：
1
2

说明：
输入解释：
第一个数字是3，也即这个小样例的N=3，说明用计算机生成了3个1到500之间的随机整数，接下来每行一个随机数字，共3行，也即这3个随机数字为：
2
2
1
所以样例的输出为：
1
2       
```
代码1：3_1_range_num_sort.py

```bash
n=int(input())
my_set={int(input()) for _ in range(n)}
 
for num in sorted(my_set):
    print(num)
```
执行：

```bash
$ python3 3_range_num_sort.py
3
2
2
1
1
2
```
代码2：3_2_range_num_sort.py

```bash
n = int(input())
list = []

for i in range(n):
   a = int(input())
   if a not in list:
      list.append(a)
      list.sort()
for i in list:
   print(i)
```
执行：

```bash
$ python3 3_2_range_num_sort.py
4
5
6
4
3
3
4
5
6
```
##  HJ4 字符串分隔
描述：

 - 输入一个字符串，请按长度为8拆分每个输入字符串并进行输出；
 - 长度不是8整数倍的字符串请在后面补数字0，空字符串不处理。

输入描述：
连续输入字符串(每个字符串长度小于等于100)

输出描述：
依次输出所有分割后的长度为8的新字符串

示例1

```bash
输入：abc
输出：abc00000
```
代码1： 4_1_string_split.py

```bash
a = input()
b = []
i = 0
while i + 8 <= len(a):
    b.append(a[i:i+8])
    i += 8
for j in b:
    print(j)
if i + 8 > len(a) and i < len(a):
    print(a[i:].ljust(8,'0'))
```
输出：

```bash
$ python3 4_string_split.py 
helloworld
hellowor
ld000000
```
代码2：4_2_string_split.py

```bash
import sys
for line in sys.stdin:
  line = line.strip()
  if len(line)%8 != 0:
     line = line + "0"*(8-len(line)%8)
  for j in range(int(len(line)/8)):
     print(line[j*8:(j+1)*8])
```
执行：

```bash
$ python3 4_2_string_split.py
hellowolrd
hellowol
rd000000
```
##  HJ5 进制转换
描述：写出一个程序，接受一个十六进制的数，输出该数值的十进制表示。

数据范围：保证结果在  `\1≤n≤2^31 -1`

输入描述：
输入一个十六进制的数值字符串。

输出描述：
输出该数值的十进制字符串。不同组的测试用例用\n隔开。

示例1

```bash
输入：0xAA
输出：170
```
代码：5_1_convert_16.py

```bash
while True:
    try:
        num16 = input()
        print(int(num16,16))
    except:
        break
```
执行：

```bash
$ python3 5_1_convert_16.py
0xAA
170
```

##  HJ6 质数因子
描述
功能:输入一个正整数，按照从小到大的顺序输出它的所有质因子（重复的也要列举）（如180的质因子为2 2 3 3 5 ）


数据范围：1≤n≤2*10^9+14 
输入描述：输入一个整数
输出描述：按照从小到大的顺序输出它的所有质数的因子，以空格隔开。
示例1

```bash
输入：180
输出：2 2 3 3 5
```
代码：6_1_质数因子.py

```bash
def FindPrimeNumber(num):
    lst = []
    i = 2    #从2开始除num
    while num != 1:    #商不等1时
        if num % i == 0:
            lst.append(i)    #如果能整除,记录下这个除数i
            num //= i    #更新num,num = 商
        else:    #如果num除以i除不尽
            if i>int(num**0.5):    #当i大于根号num时，说明num的质因子只有它本身,此时结束循环
                lst.append(num)
                break
            else:
                i+=1    #除数i+1
    for item in lst:
        print(item,end=' ')
if __name__=='__main__':
    x = int(input())
    FindPrimeNumber(x)
```
执行：

```bash
$ python3 6_1_质数因子.py
180
2 2 3 3 5 
```

代码：6_2_质数因子.py

```bash
import math
def main():
    num = int(input())
    lst = []
    i = 2    #从2开始除num
    while num != 1:    #商不等1时
        if num % i == 0:
            lst.append(i)    #如果能整除,记录下这个除数i
            num //= i    #更新num,num = 商
        else:    #如果num除以i除不尽
            if i>math.sqrt(num):    #当i大于根号num时，说明num的质因子只有它本身,此时结束循环
                lst.append(num)
                break
            else:
                i += 1    #除数i+1
    for n in lst:
        print(n,end=(' '))
main()
```
执行：

```bash
$ python3 6_1_质数因子.py
180
2 2 3 3 5 
```

##  HJ7 取近似值
描述：写出一个程序，接受一个正浮点数值，输出该数值的近似整数值。如果小数点后数值大于等于 0.5 ,向上取整；小于 0.5 ，则向下取整。

数据范围：保证输入的数字在 32 位浮点数范围内
输入描述：输入一个正浮点数值

输出描述：输出该数值的近似整数值

示例1

```bash
输入：5.5
输出：6
```

说明：0.5>=0.5，所以5.5需要向上取整为6   

示例2

```bash
输入：2.499
输出：2
```

说明：0.499<0.5，2.499向下取整为2  

代码：7_1_取近似值.py

```bash
a=str(input())
a1=int(a.split('.')[0])
if int(a.split('.')[1][0]) > 4:
    a1+=1
print(a1)
```
执行：

```bash
$ python3 7_1_取近似值.py
3.59999
4
```
代码2：7_2_取近似值.py

```bash
fl = input()
fl = float(fl)
print(round(fl))
```
执行：

```bash
$ python3 7_2_取近似值.py
3.49999
3
```

##  HJ8 合并表记录
描述
数据表记录包含表索引index和数值value（int范围的正整数），请对表索引相同的记录进行合并，即将相同索引的数值进行求和运算，输出按照index值升序进行输出。

提示:

```bash
0 <= index <= 11111111
1 <= value <= 100000
```

输入描述：
先输入键值对的个数n（1 <= n <= 500）
接下来n行每行输入成对的index和value值，以空格隔开

输出描述：
输出合并后的键值对（多行）

示例1

```bash
输入：
4
0 1
0 2
1 2
3 4

输出：
0 3
1 2
3 4
```


示例2

```bash
输入：
3
0 1
0 2
8 9

输出：
0 3
8 9
```
代码：8_1_合并表记录.py

```bash
def function():
  n = int(input())
  d = {}
  for i in range(n):
    index, value = map(int,input().split())
    if index in d.keys():
       d[index]+=value
    else:
       d[index]=value
  for s in sorted(d):
    print(s,d[s])

if __name__ == '__main__':
   function()
```
执行：

```bash
$ python3 8_1_合并表记录.py
3
3 4
3 5
4 7
3 9
4 7
```

##  HJ9 提取不重复的整数
描述：输入一个 int 型整数，按照从右向左的阅读顺序，返回一个不含重复数字的新的整数。保证输入的整数最后一位不是 0 。

数据范围：1≤n≤10^8
  
输入描述：输入一个int型整数

输出描述：按照从右向左的阅读顺序，返回一个不含重复数字的新的整数

示例1

```bash
输入：9876673
输出：37689
```
执行：

```bash
a=input()
a=a[::-1]
y=''
for i in a:
    if i not in y:
        y=y+i
print(y)
```
执行：

```bash
$ python3 9_2_提取不重复的整数.py
45673353
35764
```

##  HJ10 字符个数统计
描述
编写一个函数，计算字符串中含有的不同字符的个数。字符在 ASCII 码范围内( 0~127 ，包括 0 和 127 )，换行表示结束符，不算在字符里。不在范围内的不作统计。多个相同的字符只计算一次
例如，对于字符串 abaca 而言，有 a、b、c 三种不同的字符，因此输出 3 。

数据范围： 1≤n≤500 
输入描述：输入一行没有空格的字符串。

输出描述：
输出 输入字符串 中范围在(0~127，包括0和127)字符的种数。

示例1

```bash
输入：abc
输出：3
```

示例2

```bash
输入：aaa
输出：1
```
代码：10_1_字符个数统计.py

```bash
import sys
a=sys.stdin.readline().strip()
 
words=''
for i in a:
    if i not in words and ord(i)>=0 and ord(i)<=127:
        words+=i
print(len(words))
```
执行：

```bash
$ python3 10_1_字符个数统计.py
aabbccdef
6
```

##  HJ11 数字颠倒
描述：输入一个整数，将这个整数以字符串的形式逆序输出，程序不考虑负数的情况，若数字含有0，则逆序形式也含有0，如输入为100，则输出为001

数据范围： 0≤n≤2^30 −1 
输入描述：输入一个int整数

输出描述：将这个整数以字符串的形式逆序输出

```bash
示例1
输入：1516000
输出：0006151

示例2
输入：0
输出：0
```
代码：

```bash
a =  input()
str(a)
print(a[::-1])
```
执行：

```bash
$ python3 10_1_数字颠倒.py 
1234
4321
```

##  HJ12 字符串反转
描述：接受一个只包含小写字母的字符串，然后输出该字符串反转后的字符串。（字符串长度不超过1000）
输入描述：输入一行，为一个只包含小写字母的字符串。
输出描述：输出该字符串反转后的字符串。

示例1

```bash
输入：Abcd
输出：dcba
```

代码：

```bash
a=list(input())
print("".join(a[::-1]))
```

执行：

```bash
$ python3 12_1_字符串反转.py
abcd
dcba
```

##  HJ13 句子逆序
描述：将一个英文语句以单词为单位逆序排放。例如“I am a boy”，逆序排放后为“boy a am I”
所有单词之间用一个空格隔开，语句中除了英文字母外，不再包含其他字符
数据范围：输入的字符串长度满足 `1≤n≤1000` 

注意本题有多组输入
输入描述：输入一个英文语句，每个单词用空格隔开。保证输入只包含空格和字母。

输出描述：得到逆序的句子

```bash
示例1
输入：I am a boy
输出：boy a am I
示例2
输入：nowcoder
输出：nowcoder
```
代码：

```bash
def func():
    try:
        number_list = input().strip().split(' ')
        number_list.reverse()
        print(' '.join(number_list))
    except:
        pass
 
 
if __name__ == "__main__":
    func()
```
执行:

```bash
$ python3 13_1_句子逆序.py
I am a boy
boy a am I
```

##  HJ14 字符串排序
描述：给定 n 个字符串，请对 n 个字符串按照字典序排列。
数据范围： `1≤n≤1000`  ，字符串长度满足 `1≤len≤100` 
输入描述：输入第一行为一个正整数n(1≤n≤1000),下面n行为n个字符串(字符串长度≤100),字符串中只含有大小写字母。
输出描述：数据输出n行，输出结果为按照字典序排列的字符串。
示例1

```bash
输入：
9
cap
to
cat
card
two
too
up
boat
boot
输出：
boat
boot
cap
card
cat
to
too
two
up
```
代码：14_1_字符排序.py

```bash
import sys
num=int(sys.stdin.readline().strip())
ll=list()
for i in range(num):
   ll.append(sys.stdin.readline().strip())

ll.sort(key=str)
for item in ll:
  print(item)
```
执行：

```bash
$ python3 14_1_字符排序.py
3
cat
hat
moon
cat
hat
moon
```
代码：14_2 _字符排序.py

```bash
while True:
  try:
      n = int(input())
      result = []
      for i in range(n):
        str1 = input()
        result.append(str1)
      result = sorted(result)
      for i in result:
         print(i)
  except:
      break
```
执行：

```bash
$ python3 14_2_字符串排序.py
3
cat
hat
mooon
cat
hat
mooon
```
##  HJ15 求int型正整数在内存中存储时1的个数
描述：输入一个 int 型的正整数，计算出该 int 型数据在内存中存储时 1 的个数。

数据范围：保证在 32 位整型数字范围内
输入描述： 输入一个整数（int类型）
输出描述： 这个数转换成2进制后，输出1的个数

```bash
示例1
输入：5
输出：2
示例2
输入：0
输出：0
```
代码:15_1_int型正整数在内存中存储时1的个数.py

```bash
print(bin(int(input())).count('1'))
```
执行：

```bash
$ python3 15_1_int型正整数在内存中存储时1的个数.py
6
2
$ python3 15_1_int型正整数在内存中存储时1的个数.py
7
3
```
##  HJ16 购物单
描述：王强决定把年终奖用于购物，他把想买的物品分为两类：主件与附件，附件是从属于某个主件的，下表就是一些主件与附件的例子：
|主件|	附件|
|--|--|
|电脑	|打印机，扫描仪|
书柜|	图书|
|书桌|	台灯，文具|
|工作椅|	无|


如果要买归类为附件的物品，必须先买该附件所属的主件，且每件物品只能购买一次。
每个主件可以有 0 个、 1 个或 2 个附件。附件不再有从属于自己的附件。
王强查到了每件物品的价格（都是 10 元的整数倍），而他只有 N 元的预算。除此之外，他给每件物品规定了一个重要度，用整数 1 ~ 5 表示。他希望在花费不超过 N 元的前提下，使自己的满意度达到最大。
满意度是指所购买的每件物品的价格与重要度的乘积的总和，假设设第ii件物品的价格为v[i]v[i]，重要度为w[i]w[i]，共选中了kk件物品，编号依次为j1,j2.......jk
 ，则满意度为：v[j_1]*w[j_1]+v[j_2]*w[j_2]+ … +v[j_k]*w[j_k]v[j 1]∗w[j 
1 ]+v[j 2 ]∗w[j 2 ]+…+v[j k​ ]∗w[j k]（其中 * 为乘号）
请你帮助王强计算可获得的最大的满意度。


输入描述：
输入的第 1 行，为两个正整数N，m，用一个空格隔开：

（其中 N （ N<32000 ）表示总钱数， m （m <60 ）为可购买的物品的个数。）


从第 2 行到第 m+1 行，第 j 行给出了编号为 j-1 的物品的基本数据，每行有 3 个非负整数 v p q


（其中 v 表示该物品的价格（ v<10000 ）， p 表示该物品的重要度（ 1 ~ 5 ）， q 表示该物品是主件还是附件。如果 q=0 ，表示该物品为主件，如果 q>0 ，表示该物品为附件， q 是所属主件的编号）
 



输出描述：
 输出一个正整数，为张强可以获得的最大的满意度。

```bash
示例1
输入：
1000 5
800 2 0
400 5 1
300 5 1
400 3 0
500 2 0

输出：2200

示例2
输入：
50 5
20 3 5
20 3 5
10 3 0
10 2 0
10 1 0
复制
输出：
130
```

说明：由第1行可知总钱数N为50以及希望购买的物品个数m为5；
第2和第3行的q为5，说明它们都是编号为5的物品的附件；
第4~6行的q都为0，说明它们都是主件，它们的编号依次为3~5；
所以物品的价格与重要度乘积的总和的最大值为10*1+20*3+20*3=130 
