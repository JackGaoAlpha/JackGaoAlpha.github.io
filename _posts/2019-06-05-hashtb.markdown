---
layout: post
title: 数据结构和算法（07）：散列表
date: 2019-06-05 01:02:07.000000000 +09:00
---
## 结构

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4htxyonjfj30dh08uq38.jpg)

## 散列冲突

1. 开放寻址法

开放寻址法的实现主要有：线性探测、二次探测、双重散列。

2. 链表法

## 工业级的散列表

1. 散列函数不能太复杂：直接寻址法、平方取中法、折叠法、随机数法等

2. 高效动态扩容：需要扩容情况下，当有新数据要插入时，将新数据插入新散列表中，并且从老的散列表中拿出一个数据放入到新散列表。经过多次插入操作之后，老的散列表中的数据就一点一点全部搬移到新散列表中了。这样没有了集中的一次性数据搬移，插入操作就都变得很快了。

3. 散列冲突的解决方法：基于链表的散列冲突处理方法比较适合存储大对象、大数据量的散列表，而且，比起开放寻址法，它更加灵活，支持更多的优化策略，比如用红黑树代替链表。

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4hty81motj3060060t8p.jpg)

4. Java 中的 HashMap

* 初始大小为 16

* 最大装载因子默认 0.75

* 采用链表法解决冲突， JDK 1.8 版本引入红黑树

* 散列函数

```java
int hash(Object key) {
    int h = key.hashCode()；
    return (h ^ (h >>> 16)) & (capitity -1); //capicity 表示散列表的大小
}

public int hashCode() {
  int var1 = this.hash;
  if(var1 == 0 && this.value.length > 0) {
    char[] var2 = this.value;
    for(int var3 = 0; var3 < this.value.length; ++var3) {
      var1 = 31 * var1 + var2[var3];
    }
    this.hash = var1;
  }
  return var1;
}
```

## 散列表 + 链表： LRU 缓存淘汰

实现缓存的添加、删除、查找的时间复杂度都是 O(1)。

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4htyfyu04j30cd087mxp.jpg)

## 算法题

### LeetCode 49. Group Anagrams

将 Anagrams 分组。

```
Input: ["eat", "tea", "tan", "ate", "nat", "bat"],
Output:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]
```

使用散列表将排序后的字符串分组就好了。

```python
class Solution(object):
    def groupAnagrams(self, strs):
        ans = collections.defaultdict(list)
        for s in strs:
            ans[tuple(sorted(s))].append(s)
        return ans.values()
```



