# [1704. 判断字符串的两半是否相似](https://leetcode.cn/problems/determine-if-string-halves-are-alike/)

```c++
bool halvesAreAlike(string s) {
  string a = s.substr(0, s.size() / 2);//生成新的子字符串,,,起始位置,,,子字符串长度
  string b = s.substr(s.size() / 2);
  string target = {"aeiouAEIOU"};
  int numa = 0, numb = 0;
  for (int i = 0; i < a.size(); i++) {
    //答案这里使用的是find_first_of()
    if (target.find(a[i]) != -1) numa++;//是在目标字符串中一个一个匹配现有的字符,反过来的
  }
  for (int i = 0; i < b.size(); i++) {
    if (target.find(b[i]) != -1) numb++;
  }
  if (numa == numb)
    return true;
  else
    return false;
}
```

关键点:
- 掌握substr(),find(),find_first_of()三个函数的用法
- 想到在target中一个一个查找a和b中的单个字符

## substr()

> 函数的功能是从一个字符串的指定位置指定长度,复制出来一个新的字符串

```c++
substr(size_type _Off = 0,size_type _Count = npos)
```
两个参数:
- 第一个off是复制字符串的起始位置,默认是0,也就是第一个
- 注意第二个参数count是需要**复制的字符数目**,不是下标
- 如果没有指定字符串长度的话,直接从指定位置复制到字符串末端


  
举一个例子吧
string str = "codoncodon";

- 就是整个复制了一遍str string str4 = str.substr();
- 复制str字符串下标5的字符到str的末端 string str5 = str.substr(5)
- 提取前三个字符，可以用 string str1 = str.substr(0,3); 
- 提取下标4-6 string str2 = str.substr(4,3); 
- 然后下标7-9 string str3 = str.substr(7,3);

## find()与find_first_of()

> 这两个括号里面参数一样(要找的字符串,查找的开始位置下标,查找的字符串长度)

str1.find(str2,5,8);

在str1中查找与str2完全对的上的字符串,从下标为5(第六个)开始查找,往后数八个


find("xt")是在this中寻找连续的xt字符返回第一个出现的位置

find_first_of("aeiouAEIOU")是在this中寻找aeiouAEIOU这几个单个的字符,返回其中第一个出现的位置

