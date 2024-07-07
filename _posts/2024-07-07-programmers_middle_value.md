---
title: 프로그래머스 - 중앙값 구하기
author: Insub
date: 2024-07-07 00:00:00 +0900
categories:
- 프로그래머스
---

# 프로그래머스 - 중앙값 구하기
<img src="../assets/img/2024-07-07-programmers_middle_value/2024-07-07-21-27-10.png" width="75%" />

기초 알고리즘 문제인데, c++을 몇 년만에 다시 공부하면서 quick sort 함수 구현을 해보게 되어서 글로 남긴다.

## 1. 접근

(1) 정렬한다.<br/>
(2) array[len/2]를 리턴한다.

## 2. 소스코드
```c++
void quick_sort(int* array, int array_len, int left, int right) {
    if (left >= right) return;
    int org_left = left;
    int org_right = right;
    int pivot = array[left++];

    //pivot - left ------ right

    while (left <= right) {
        while (left < array_len && array[left] <= pivot) left++;
        while (right >= left && array[right] >= pivot) right--;

        //swap
        if (left < right) {
            int tmp = array[left];
            array[left] = array[right];
            array[right] = tmp;
        }

    }
    int tmp = array[right];
    array[right] = array[org_left];
    array[org_left] = tmp;


    quick_sort(array, array_len, org_left, right - 1);
    quick_sort(array, array_len, right + 1, org_right);
}
void sort(int* array, int array_len) {
    if (array_len <= 0) return;

    quick_sort(array, array_len, 0, array_len - 1);


}

// array_len은 배열 array의 길이입니다.
int solution(int array[], size_t array_len) {
    sort(array, array_len);
    return array[array_len / 2];
}
```


> **<퀵 정렬 알고리즘>**
<br/>첫번째 값을 pivot으로 잡아서, left는 +1씩 증가시켜가며 pivot보다 큰 값을 만날 때까지 이동하고, right는 -1씩 감소시켜가며 pivot보다 작은 값을 만날 때까지 이동한다.
<br/>left와 right가 둘다 정지했다면, left \< right라는 가정 하에 두 자리의 값을 swap한다.
<br/>
<br/>그리고 다시 위의 동작을 반복한다.
<br/>위의 동작은 left가 right를 넘어설 때까지 반복된다.
<br/>
<br/>left가 right를 넘어섰다면 right의 자리와 pivot의 자리의 값을 교체한다.

