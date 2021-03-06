# 关联容器
关联容器有八种，是三种不同特性的二选一组合：map、set，有序、无序，不可重复、可重复：
map，set，multimap，multiset，unordered_map，unordered_set，unordered_map，unordered_map，
## 11.1 使用关联容器
## 11.2 关联容器概述
关联容器的操作：普通的容器操作，不支持顺序容器中基于未知的操作如push_front等
关联容器的特有操作：操作关键字的操作，类型别名，无序容器提供调整哈希性能的操作
关联容器的迭代器都是双向的
### 11.2.1 定义关联容器
```
map<string, size_t> word_count;
set<string> exclude = {"the", "but", "and", "or", "an", "a", "The", "But", "And", "Or", "An", "A"};
map<string, string> autors = {{"Zhang", "San"}, {"Li", "Si"}, {"Wang", "Wu"}};
```
- 初始化multimap或multiset

`multiset<int> miset(ivec.begin(), ivec.end())`
### 11.2.2 关键字类型的要求
有序容器的关键字必须定义元素比较的方法
- 有序容器的关键字类型
必须有严格意义的小于，具体包含下面三个规则：
1. 两个关键字不能同时小于对方
2. 传递性：k1小于k2，k2小于k3，k1小于k3
3. 如果两个关键字同时不小于对方，则两个关键字等价，等价也有传递性
- 使用关键字类型的比较函数
`multiset<Sales_data, decltype(compareIsbn) *>`
### 11.2.3 pair类型
在<utility>中
map的元素是pair类型
`make_pair(v1, v2)`
  
## 11.3 关联容器操作
key_type 关键词类型
value_type 对于set与key_type相同，对于map为pair<const key_type, mapped_type>
mapped_type 只适用于map，为关键字所关联到的类型，即pair.second的类型
### 11.3.1 关联容器迭代器
- set的迭代器是const的，map的key是const的
- 遍历关联容器
```
auto map_it = word_count.cbegin();
while (map_it != word_count.cend())
{
  ++map_it;
}
```
- 关联容器和算法
我们一般不对关联容器使用泛型算法。有写操作的的泛型算法无法用于关联容器，因为set的元素是const，而map的元素中pair.first为const。
### 11.3.2 添加元素
insert操作，由于set和map没有重复元素，向其中添加已存在的元素是没有效果的
- 向map添加元素
要插入一个pair对象
```
word_count.insert({word, 1});
word_count.insert(make_pair(word, 1));
```
- 检测insert的返回值

insert返回值为一个pair，其中first是一个迭代器，second是bool，表明是否成功，如果插入的元素已经存在，则插入失败，不进行插入操作，返回false
- 展开递增语句
- 向multiset或multimap添加元素

对map来说如果一个key对应多个元素可以用multimap
- 删除元素

关联容器提供了一个特别的迭代器，接收关键字参数，删除与该关键字关联的所有元素，返回值是删除元素的数量。对于无重复版本，返回值只可能是0和1。
### 11.3.4 map的下标操作
只有map与unordered_map有下标操作，相对应的还有一个at函数。下标操作接收一个关键字参数，如果下标存在的话，返回它关联的值，如果不存在，则创建一个，如果创建时没有给值，将执行值初始化。下标操作不适用于const的容器。
- 使用下标操作的返回值
下标操作的返回值是mapped_type，而迭代器解引用返回的是value_type。
### 11.3.5 访问元素
c.find(k)返回一个迭代器，指向第一个关键字为k的元素, c.count(k)返回值的个数
- 在multimap或multiset中查找元素

由于关键字相同的项都是保存在相邻的位置，同时使用count和find，find返回第一个，遍历count次，就可以获取所有的k关联元素
- 一种不同的，面向迭代器的解决方法

lower_bound返回第一个匹配的元素位置与upper_bound返回最后一个匹配元素后面的位置。如果元素不存在则两个返回相等的迭代器。元素不存在时，两者返回的都是第一个安全插入点。
- equal_range函数

返回一个lower_bound, upper_bound组合的pair
### 11.3.6 一个单词转换的map
## 11.4 无序容器
无序容器使用场景：关键字没有明显的顺序关系，维护顺序开销很大。
- 使用无序容器
无序容器提供了与有序容器相同的操作。
- 管理桶
- 无序容器对关键字类型的要求
标准库为内置类型、string、智能指针提供了hash模板，所以可以直接定义这些类型关键字的无序容器。如果是我们自定义的关键字类型，则需要我们提供自己的hash模板，以及==运算符。
