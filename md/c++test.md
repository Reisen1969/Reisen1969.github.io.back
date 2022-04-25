+++
author = "rayrain"
title = "C++记录"
date = "2022-04-26"
description = "C++"
toc= true
math= true
tags = [
    "C","CPP"
]

+++

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

