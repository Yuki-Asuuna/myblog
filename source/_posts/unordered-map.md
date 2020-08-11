---
title: 浅谈C++11中的unordered_map
date: 2020-08-03 17:19:58
tags: [C++,unordered map]
---

# C++11中的unordered_map

## 1.前言

​	C++11标准加入了以`unordered_map`为代表的`unordered`系列容器，看了一下官方文档，虽然和map在功能上相差不大，但感觉有些功能和特性还挺有意思的。这里做一个简单的比较和整理。



## 2.实现方式的比较

​	unordered_map采用的是hash的方式实现键-值对查询，在绝大多数情况下，hash做单值等值查询是非常快的，其效率优于红黑树（常数级复杂度）。所以如果你仅仅想要实现**“等值查询“**的目的，并且对“有序性“没有要求的话，可以直接使用unordered_map。

​	我们都知道，如果对map进行遍历，其得出的结果是有序的（相当于对一颗平衡树做中序遍历）。而对unordered_map进行遍历，其结果则不一定有序，所以这就是为什么它被称为unordered（无序的）。

​	正因为map的底层实现是红黑树，所以程序员可以在上高效地做一些类似lower_bound、upper_bound、equal_range这样的非等值查询；而unordered_map的底层是hash，显然刚才提到的3个操作是无法高效完成的。



## 3.unordered_map中的特殊方法

​	unordered_map中的大多数方法和运算符（[]、size、erase等）与map并无二异，效果也完全相同。不过也有一些特殊方法：

### 	1）Buckets相关

- bucket_count()

  返回当前容器中桶（bucket）的个数，桶的个数会随着元素的不断加入而动态变化。

  ```cpp
  #include <bits/stdc++.h>
  using namespace std;
  char *randstr(char *str, const int len)
  {
      int i;
      for (i = 0; i < len; ++i)
      {
          switch ((rand() % 3))
          {
          case 1:
              str[i] = 'A' + rand() % 26;
              break;
          case 2:
              str[i] = 'a' + rand() % 26;
              break;
          default:
              str[i] = '0' + rand() % 10;
              break;
          }
      }
      str[++i] = '\0';
      return str;
  }
  int main(){
  	srand(time(NULL));
  	unordered_map <string,int> mm;
  	cout<<mm.bucket_count()<<endl;//output : 11
  	for (int i=0;i<10000;i++){
  		char str[20];
  		randstr(str,8);
  		mm[str]=i;
  	}
  	cout<<mm.bucket_count()<<endl;//output : 15173
  	
  	return 0;
  }
  ```

- max_bucket_count()

  返回桶数量的上限

```cpp
int main(){
	srand(time(NULL));
	unordered_map <string,int> mm;
	cout<<mm.max_bucket_count()<<endl;//output : 576460752303423487
	return 0;
}
```

- bucket_size(n)

  返回n号桶中有多少个元素

  ```cpp
  int main(){
  	srand(time(NULL));
  	unordered_map <string,int> mm;
  	cout<<mm.bucket_count()<<endl;//output : 11
  	for (int i=0;i<10000;i++){
  		char str[20];
  		randstr(str,8);
  		mm[str]=i;
  	}
  	cout<<mm.bucket_size(10)<<endl;//output : 1 （输出不定）
  	return 0;
  }
  ```

- bucket(key)

  定位键值为key的元素在几号桶

  ```cpp
  // unordered_map::bucket
  #include <iostream>
  #include <string>
  #include <unordered_map>
  
  int main ()
  {
    std::unordered_map<std::string,std::string> mymap = {
      {"us","United States"},
      {"uk","United Kingdom"},
      {"fr","France"},
      {"de","Germany"}
    };
  
    for (auto& x: mymap) {
      std::cout << "Element [" << x.first << ":" << x.second << "]";
      std::cout << " is in bucket #" << mymap.bucket (x.first) << std::endl;
    }
  
    return 0;
  }
  ```

  ```
  #可能的输出
  Element [us:United States] is in bucket #1
  Element [de:Germany] is in bucket #2
  Element [fr:France] is in bucket #2
  Element [uk:United Kingdom] is in bucket #4
  ```



### 	2）Hash_policy相关

- load_factor()

  load_factor又被称为“负载因子”，它被定义为元素个数与桶个数的比值。
  $$
  load\_factor=size/bucket\_count
  $$
  load_factor与冲突的概率密切相关，直接影响数据结构的性能。在插入元素时，unordered_map会根据当前的load_factor进行调整，如果load_factor超过一个阈值（max_load_factor），则会增加桶的个数，并且rehash。

  ```cpp
  // unordered_map hash table stats
  #include <iostream>
  #include <unordered_map>
  int main ()
  {
    std::unordered_map<int,int> mymap;
    std::cout << "size = " << mymap.size() << std::endl;
    std::cout << "bucket_count = " << mymap.bucket_count() << std::endl;
    std::cout << "load_factor = " << mymap.load_factor() << std::endl;
    std::cout << "max_load_factor = " << mymap.max_load_factor() << std::endl;
    return 0;
  }
  ```

  ```
  #可能的输出
  size = 0
  bucket_count = 11
  load_factor = 0
  max_load_factor = 1
  ```

- max_load_factor

  刚才我们提到的负载因子可以用这个方法进行获取和设置。

  **默认情况下max_load_factor = 1。**

  > 它有两种重载形式，分别表示获取和设置 load_factor：
  >
  > float max_load_factor()
  >
  > void max_load_factor(float)

  ```cpp
  // unordered_map::max_load_factor
  #include <iostream>
  #include <string>
  #include <unordered_map>
  int main ()
  {
    std::unordered_map<std::string,std::string> mymap = {
      {"Au","gold"},
      {"Ag","Silver"},
      {"Cu","Copper"},
      {"Pt","Platinum"}
    };
  
    std::cout << "current max_load_factor: " << mymap.max_load_factor() << std::endl;
    std::cout << "current size: " << mymap.size() << std::endl;
    std::cout << "current bucket_count: " << mymap.bucket_count() << std::endl;
    std::cout << "current load_factor: " << mymap.load_factor() << std::endl;
  
    float z = mymap.max_load_factor();
    mymap.max_load_factor ( z / 2.0 );
    std::cout << "[max_load_factor halved]" << std::endl;
  
    std::cout << "new max_load_factor: " << mymap.max_load_factor() << std::endl;
    std::cout << "new size: " << mymap.size() << std::endl;
    std::cout << "new bucket_count: " << mymap.bucket_count() << std::endl;
    std::cout << "new load_factor: " << mymap.load_factor() << std::endl;
    return 0;
  }
  ```

- rehash(n)

  将容器中桶的个数设置为n（**或者大于n**），并且rehash。

  但是这里有两种情况：

  1. 如果n>bucket_count()，那么rehash操作会被强制执行。
  2. 如果n<bucket_count()，那么rehash不会执行，也不会影响当前桶的个数。

  rehash的复杂度是线性的（平均情况下），但是在最坏情况下，可能会退化为平方级（元素个数的平方）。

  ```cpp
  // unordered_map::rehash
  #include <iostream>
  #include <string>
  #include <unordered_map>
  
  int main ()
  {
    std::unordered_map<std::string,std::string> mymap;
  
    mymap.rehash(20);
  
    mymap["house"] = "maison";
    mymap["apple"] = "pomme";
    mymap["tree"] = "arbre";
    mymap["book"] = "livre";
    mymap["door"] = "porte";
    mymap["grapefruit"] = "pamplemousse";
  
    std::cout << "current bucket_count: " << mymap.bucket_count() << std::endl;
  
    return 0;
  }
  ```

  

- reserve(n)

  提前指定分配桶的个数，避免rehash带来的开销，和STL vector中的reserve类似，不再赘述。

  

## 4.总结

​	一句话，如果你要实现kv**等值查询**，请你直接抛弃map和hash_map，使用unordered_map即可，除非你对数据有有序的要求。