---
title: python中的小知识点——备忘录（持续更新）
date: 2020-07-19 18:13:29
tags: [python]
---

## 1. 打印%（百分号）

%号会被误认为是占位符（类比C printf），如果要输出百分号，则要打两个百分号%

```python
print('100%%')	#输出'100%'
```



## 2. is/is not运算符

is和is not是python中的**身份运算符**，其本身（is和not）也是关键字，它用来比较两个对象的存储单元是否一致。

a is b 等价于 id(a) == id(b)。is not同理。

例如：

```python
a=10
b=a
if a is b:
	print('yes!')	#输出'yes'
```



## 3. //运算符

//属于python中的算术运算符，返回商的整除部分（向下取整）。由于python的除法默认是保留小数的，所以就引入了//运算符。

```
>>> 9 // 4 = 2
```



## 4. 逻辑运算符的赋值功能

众所周知，python中的逻辑运算符有and,or,not，它们除了可以用于if作为条件的连接符，同样也可以用于赋值。

比如：

```python
a1 = 1 and 3	#a1等于3

a2 = "alex" or "jack"	#a2等于"alex"

a3 = 0 or 100	#a3等于100

a4 = "" or "jack"	#a4等于"jack"
```

总结：

1. or， 看第一个值，如果是 `真` 则选择第一个值，否则选择第二个值。
2. and，看第一个值，如果是 `假` 则选择第一个值，否则选择第二个值。



注：

1. 什么值被认为是'假'？有0、空字典、空字符串、空元组、空列表。其它均为True。
2. and的优先级高于or



## 5. 多行表示字符串

传统的""（两个双引号），''（两个单引号）可以用来表示字符串，但是字符串只能挤在一行里面，如果我们要用多行去表示一个字符串，该怎么办？（比如sql的查询语句）



这个时候就可以用三个"""去表示多行字符串

比如：

```python
a="""
	select *
	from emp
	where age > 18;
"""
```



## 6. 常见的字符串方法

- endswith()/startswith()

  用于判断前缀/后缀是否是给定字符串

  ```python
  v1 = "jack is the winner!"
  result = v1.startswith("jack")
  print(result)	#输出True
  ```

- isdecimal()

  用于判断字符串是否是十进制数字

- strip()

  返回字符串去除两边空格、制表符、换行的结果

- replace()

  list.replace(str1,str2)

  将字符串中所有的str1替换为str2

- split()

  非常常用的一个函数，不讲了



## 7. 常见的list方法

- insert()

  用法 list.insert(pos,elem)

  比如 list.insert(1,"ddd")，在列表list中的1号位置插入元素'ddd'

- extend()/+=

  将两个list中的元素进行合并

  ```python
  A=['a','b','c']
  B=['1','2']
  A.extend(B)	#用A+=B也可以
  print(A)	#输出['a','b','c','1','2']
  ```

- remove()

  根据值进行删除（从左到右找到第一个元素删除）

  ```python
  user_list = ["王宝强","陈羽凡","Alex","贾乃亮","Alex"]
  user_list.remove("Alex")
  print(user_list)	#输出["王宝强","陈羽凡","贾乃亮","Alex"]
  ```

- pop()

  根据索引删除某个元素，并且返回那个即将被删除的元素的值

  ```python
  user_list = ["王宝强","陈羽凡","Alex","贾乃亮","Alex"]
  ele = user_list.pop() # 在user_list删除最后一个，并讲删除值赋值给ele
  item = user_list.pop(2) # 在user_list中删除索引未2的值，并将删除值赋值给item
  ```

  当然，用del也是一个好的选择

  ```python
  del user_list[0]
  ```

- clear()

  清空列表

  ```python
  user_list = ["王宝强","陈羽凡","Alex","贾乃亮","Alex"]
  user_list.clear()
  print(user_list)	#空list
  ```

- reverse(）

  ```python
  user_list = ["王宝强","陈羽凡","Alex","贾乃亮","Alex"]
  user_list.reverse()
  print(user_list)	#输出["Alex","贾乃亮","Alex","陈羽凡","王宝强"]
  ```

  

## 8. 字符串转数字（带进制转换）

可以直接用python的int()函数，只要多加一个参数base即可

```python
a='101'
print(int(a,base=2))	#输出5
```



## 9. 元组

**建议在元组的最后多加一个逗号，用于标识它是一个元组**

```python
# 面试题
1. 比较值 v1 = (1) 和 v2 = 1 和 v3 = (1,) 有什么区别？
2. 比较值 v1 = ((1),(2),(3)) 和 v2 = ((1,),(2,),(3,),) 有什么区别？
```

```python
#答
1. v1=(1)是int类型，v2也是int类型，v3是一个元组类型
2. v1是一个含有3个int类型的元组，v2是一个含有3个元组类型的元组
```



## 10. 字典

- items()

  取所有的键值对

  ```python
  for key,value in info.items():
   print(key,value) # key代表键，value代表值，将兼职从元组中直接拆分出来了
  ```

  

- update()

  更新键值对

  ```python
  info = {"age":12, "status":True}
  info.update({"age":14,"name":"武沛齐"}) # info中没有的键直接添加；有的键则更新值
  print(info) # 输出：{"age":14, "status":True,"name":"武沛齐"}
  ```

  

- pop()

  移除键值对

  ```python
  info = {"age":12, "status":True,"name":"武沛齐"}
  data = info.pop("age")
  print(info) # {"age":12,"name":"武沛齐"}
  print(data) # 12
  ```

- get()

  根据键获取值

  ```python
  info = {"age":12, "status":True,"name":"武沛齐"}
  data = info.get("name",None) # 根据name为键去info字典中获取对应的值，如果不存在则返回None，存在则返回值。
  print(data) # 输出：武沛齐
  ```

  

## 11. 集合

- 定义

  和dict用的都是大括号{}，但是字典内部是键值对而集合内部是值。

  ```python
  v1 = {1,2,99,18}
  v2 = {"武沛齐","Alex","老妖","Egon"}
  v3 = {1,True,"world",(11,22,33)}
  ```

  *集合内的元素必须是可哈希元素。

  

- 添加/删除元素

  ```
  s.add()
  s.discard()
  ```

- 集合运算

  ```python
  v1 = s1 & s2	#交集
  v2 = s1 | s2	#并集
  v3 = s1 - s2	#差集
  ```

  

## 12. 函数

- 查看函数注释

  ```python
  def eat(food,drink):
      '''
      这里描述这个函数是做什么的.例如这函数eat就是吃
      :param food:  food这个参数是什么意思
      :param drink: drink这个参数是什么意思
      :return:  执行完这个函数想要返回给调用者什么东西
      '''
      print(food,drink)
  eat('麻辣烫','肯德基')
  
  print(eat.__doc__)  #函数名.__doc__
  ```

  ```
  这里描述这个函数是做什么的.例如这函数eat就是吃
  :param food:  food这个参数是什么意思
  :param drink: drink这个参数是什么意思
  :return:  执行完这个函数想要返回给调用者什么东西
  ```

  

- 查看所有全局/局部变量

  ```python
  globals()
  locals()
  ```

  

- 获取对象hash值

  ```python
  print(hash(object))
  ```

  

- 获取函数帮助文档

  ```python
  help(print)
  ```

  

- 查看对象的属性

  ```python
  print(dir(list))
  ```

  

- enumerate()

  ```python
  lst = ['alex','wusir','taibai']
  for i,k in enumerate(lst):
      print('这是序号',i)
      print('这是元素',k)
  ```

  

- *args和**kwargs

  - *arg会把多出来的位置参数转化为`tuple`

  - **kwarg会把关键字参数转化为`dict`

    ```python
    def exmaple2(required_arg, *arg, **kwarg):
        if arg:
            print "arg: ", arg
    
        if kwarg:
            print "kwarg: ", kwarg
    
    exmaple2("Hi", 1, 2, 3, keyword1 = "bar", keyword2 = "foo")
    #keyword1 = 'bar' keyword2='foo'被称为关键字参数
    
    >> arg:  (1, 2, 3)
    >> kwarg:  {'keyword2': 'foo', 'keyword1': 'bar'}
    ```

    ```python
    def test(*arg,**kwargs):  
    print arg   
    print kwargs  
    print "-------------------"   
    if __name__=='__main__':  
        test(1,2,3,4,5)  
        test(a=1,b=2,c=3)  
        test(1,2,3,a=1,b=3,c=5)  
    ```

    ```
    (1, 2, 3, 4, 5)  
    {}  
    -------------------  
    ()  
    {'a': 1, 'c': 3, 'b': 2}  
    -------------------  
    (1, 2, 3)  
    {'a': 1, 'c': 5, 'b': 3}  
    ```

    

