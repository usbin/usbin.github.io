--- layout: post title: "[C++] 백준 알고리즘 1181번(단어 정렬) - 커스텀 sort 함수 주의점(segmentation fault)" date: '2022-07-27T01:38:00.001-07:00' author: 가도 tags: - 자료구조/알고리즘 - C-plusplus modified\_time: '2022-08-12T08:11:45.148-07:00' thumbnail: https://blogger.googleusercontent.com/img/a/AVvXsEjXIlwvYD04iQ2\_FZXHyMRui4C6Y9JzO5sheANQKNCK4c3eMDxvGYLp8PpXTy5YMYWNZEYWRqeGog\_ZNjz3jW\_tThLOjRFnU\_MP5G-sByfUiHoGSFOxh3cahS7Wg16hIBwunqRyY5ANAERkLNg6gvrcK9rBSHd-oiO4zEkvjtS1IZNP4l7HyWplfSro3g=s72-c blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-5111204435971065173 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/07/c-1181-sort-segmentation-fault.html ---

&nbsp;[https://www.acmicpc.net/problem/1181](https://www.acmicpc.net/problem/1181)

저번처럼 그냥 sort 커스텀 정렬을 사용하면 되...는 줄 알았는데

​

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjXIlwvYD04iQ2_FZXHyMRui4C6Y9JzO5sheANQKNCK4c3eMDxvGYLp8PpXTy5YMYWNZEYWRqeGog_ZNjz3jW_tThLOjRFnU_MP5G-sByfUiHoGSFOxh3cahS7Wg16hIBwunqRyY5ANAERkLNg6gvrcK9rBSHd-oiO4zEkvjtS1IZNP4l7HyWplfSro3g=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEjXIlwvYD04iQ2_FZXHyMRui4C6Y9JzO5sheANQKNCK4c3eMDxvGYLp8PpXTy5YMYWNZEYWRqeGog_ZNjz3jW_tThLOjRFnU_MP5G-sByfUiHoGSFOxh3cahS7Wg16hIBwunqRyY5ANAERkLNg6gvrcK9rBSHd-oiO4zEkvjtS1IZNP4l7HyWplfSro3g)
  
  

안 되더라.

​

4시간동안 삽질 끝에 segfault가 뜨는 이유를 알아냈다.

​

"sort 함수는 두 수가 동일할 경우 false를 반환해야 한다"고 한다.

sort 함수는 Strict weak ordering을 만족해야 하는데, 이는 다음과 같다.

1. 모든 x에 대해 f(x, x)는 false를 반환해야 한다.

2. 모든 x,y에 대해 f(x,y)가 true면 f(y,x)는 false를 반환해야 한다.

3. 모든 x, y, z에 대해 f(x, y)와 f(y, z)가 true이면 f(x, z)는 true이어야 한다.

4. 모든 x, y, z에 대해 f(x, y)와 f(y, x)가 false이고 f(y, z)와 f(z, y)가 false이면 f(x, z)와 f(z, x)는 false여야 한다.

여기서 문제가 되는 건 1번이다.

모든 x에 대해 f(x, x)가 false를 반환해야 한다는 건 즉, 같은 수를 정렬함수에 넣었을 때 리턴값은 반드시 false가 나와야 한다는 것이다.

​

위의 코드에선 계속 정렬함수를 이렇게 썼는데

std::sort(strV.begin(), strV.end(), [](std::string str1, std::string str2) {if (str1.length() \< str2.length()) return true;else if (str1.length() == str2.length()) {for (int i = 0; i \< str1.length(); i++) {if (str1[i] - str2[i] \< 0) return true;else if (str1[i] - str2[i] \> 0) return false;}return true;}else return false;});

str1과 str2의 글자수가 다를 땐 글자수가 적은 것이 앞에 오도록,

같을 땐 첫번째 글자부터 비교해서 사전상 앞에 오는 것이 앞에 오도록,

첫 글자부터 마지막 글자까지 동일하다면 그냥 true를 리턴하도록(옳은 순서로 치도록) 했다.

문제는 이 경우 a와 b가 동일할 경우 true를 반환하도록 되어 있다는 것이다.

​

이 정렬함수를 그대로 쓰면서 고치고 싶다면

std::sort(strV.begin(), strV.end(), [](std::string str1, std::string str2) {if (str1.length() \< str2.length()) return true;else if (str1.length() == str2.length()) {for (int i = 0; i \< str1.length(); i++) {if (str1[i] - str2[i] \< 0) return true;else if (str1[i] - str2[i] \> 0) return false;}return false; //true-\>false로 고침}else return false;});

이렇게 해주면 되는데, 이것보다 더 간편한 방법이 있어서 그걸 쓰기로 했다.

​

c++은 문자열간의 사전순 비교를 제공한다고 하더라.

str1 \< str2 을 하면 str1이 str2보다 사전순으로 앞에 와야만 true가 나온다.

즉,

std::sort(strV.begin(), strV.end(), [](std::string str1, std::string str2) {if (str1.length() == str2.length()) {return str1 \< str2;}else {return str1.length() \< str2.length();}});

이렇게 해주면 된다.

str1과 str2의 길이가 같다면 사전순으로 먼저 오는 것이 앞에 오도록 하고,

길이가 다르다면 짧은 것을 앞에 오도록 한다.

​

전체 소스코드는 다음과 같다.

#include \<iostream\>#include \<string\>#include \<algorithm\>#include \<vector\>int main() { int N; std::cin \>\> N; std::vector\<std::string\> strV;for (int i = 0; i \< N; i++) { std::string word; std::cin \>\> word; strV.push\_back(word);} std::sort(strV.begin(), strV.end(), [](std::string str1, std::string str2) {if (str1.length() == str2.length()) {return str1 \< str2;}else {return str1.length() \< str2.length();}}); strV.erase(std::unique(strV.begin(), strV.end()), strV.end());for (int i = 0; i \< strV.size(); i++) { std::cout \<\< strV[i] \<\< "\n";}}

​

- [Keep](https://blog.naver.com/PostView.naver?blogId=rooprete99&logNo=222831552720&categoryNo=46&parentCategoryNo=28&viewDate=&currentPage=2&postListTopCurrentPage=1&from=postList&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=2# "Keep 보내기")
- [메모](https://blog.naver.com/PostView.naver?blogId=rooprete99&logNo=222831552720&categoryNo=46&parentCategoryNo=28&viewDate=&currentPage=2&postListTopCurrentPage=1&from=postList&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=2# "메모 보내기")
- [보내기](https://blog.naver.com/PostView.naver?blogId=rooprete99&logNo=222831552720&categoryNo=46&parentCategoryNo=28&viewDate=&currentPage=2&postListTopCurrentPage=1&from=postList&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=2# "기타 보내기 펼치기")

[수정](https://blog.naver.com/PostView.naver?blogId=rooprete99&logNo=222831552720&categoryNo=46&parentCategoryNo=28&viewDate=&currentPage=2&postListTopCurrentPage=1&from=postList&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=2#)&nbsp;[삭제](https://blog.naver.com/PostView.naver?blogId=rooprete99&logNo=222831552720&categoryNo=46&parentCategoryNo=28&viewDate=&currentPage=2&postListTopCurrentPage=1&from=postList&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=2#)&nbsp;[설정](https://blog.naver.com/PostView.naver?blogId=rooprete99&logNo=222831552720&categoryNo=46&parentCategoryNo=28&viewDate=&currentPage=2&postListTopCurrentPage=1&from=postList&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=2#)

  

