---
title: 프로그래머스 - 최빈값 구하기
author: Me
date: 2024-07-07 00:00:00 +0900
categories:
- 개발 공부
- 알고리즘
---
<img src="../assets/img/2024-07-07-programmers_most-frequent-value/2024-07-07-22-24-10.png" width="75%" alt=""/>


좀 찝찝하게 풀었다. \
추후에 더 깔끔한 코드를 고민해봐야함.

```c++
#include <string>
#include <vector>
#include <map>

using namespace std;

int solution(vector<int> array) {


    map<int, int> map;

    for(auto it=array.begin(); it!=array.end(); it++){
        if(map.find(*it) == map.end())
            map.insert({*it, 1});
        else
            map[*it]++;
    }

    int max_key = -1;
    int max = -1;
    for(auto it=map.begin(); it!=map.end(); it++){
        if(it->second > max){
            max_key = it->first;
            max = it->second;
        }
    }
    for(auto it=map.begin(); it!=map.end(); it++){
        if(it->first != max_key && it->second == max) return -1;
    }
    return max_key;
}
```