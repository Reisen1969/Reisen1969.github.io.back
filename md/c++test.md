+++
author = "rayrain"
title = "C++刷题记录"
date = "2022-04-26"
description = "C++"
toc= true
math= true
tags = [
    "C","CPP"
]

+++

### 一些算法 技巧

```
对于算法来说,迭代器的区间都是前闭后开

计算某个区间之和accumulate

从迭代器获得index : i - num.begin()
                distance(num.begin(),i)

操作string
isalnum,判断单个字符是否是 字母或数字
tolower 单个字符转小写

```

### 牛顿法平方根

```java
 public int mySqrt(int x) {

        int start = 0, end = x;
        int res = -1;

        while (start <= end) {
            int mid = start + (end - start)/2;

            if ((long)mid * mid <= x) {
                start = mid + 1;
                res = mid;
            } else {
                end = mid -1;
            }
        }

        return res;
    }
```



## cin读入逗号分隔的数字

```

#include<bits/stdc++.h>
using namespace std;

int main()
{
    vector<int> data;

    string s;
    getline(cin,s);

    istringstream input;

    input.str(s);

    for(string line;getline(input,line,',');) {

    data.push_back(stoi(line));

    }
    int sum = 0;
    for(auto i:data) sum+=i;

    cout<<sum;

    return 0;


}
```

### leetcode版本的链表与使用

```
#include<bits/stdc++.h>

using namespace std;

struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x),next(next) {}

};

int main()
{
    ListNode* head = new ListNode(0);
    ListNode* point = head;


    for(int i = 1; i < 10; i++) {
        point->next = new ListNode(i);
        point = point->next;
    }

    point = head;
    while(1) {
        cout<< point->val << endl;
        if (point->next)
            point = point->next;
        else
            break;
    }


    return 0;
}

```

