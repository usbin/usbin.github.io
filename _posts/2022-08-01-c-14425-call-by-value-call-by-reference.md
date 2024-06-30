--- layout: post title: "[C++] 백준 14425번 - 벡터의 이진탐색, call by value, call by reference" date: '2022-08-01T01:40:00.000-07:00' author: 가도 tags: - 자료구조/알고리즘 - C-plusplus modified\_time: '2022-08-12T08:11:45.146-07:00' blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-4655169351110467240 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/08/c-14425-call-by-value-call-by-reference.html ---

&nbsp;[https://www.acmicpc.net/problem/14425](https://www.acmicpc.net/problem/14425)

> 1. 접근하기

가장 처음 접근한 방법은 (1) 단순하게 string 벡터를 만들어 집어넣고, 찾는 거였다.

그러나 시간초과로 실패.

문자열의 길이, 개수를 모두 n으로 치환하고 시간복잡도를 계산해보면 무려 O(n^4)라는 끔찍한 시간복잡도를 갖는다.

(2) 이번엔 순서대로 정렬한 다음, 이진 탐색을 통해 찾기로 했다.

역시나 시간초과로 실패.

이 방법의 시간복잡도는 O(n^2 log n)

​

> 시간초과가 나는 이유

_이진탐색이 O(log n)의 시간복잡도를 갖는데 시간초과가 났다면 더이상 탐색 시간을 빠르게 할 방법은 없는 게 아닌가_...하고 알아보니, 엉뚱한 부분에서 시간초과가 났었다.

이진탐색을 binary\_search(벡터, first, last, 찾을 문자열)의 형태로 사용했는데, 벡터를 파라미터로 넘겨줄 때 벡터를 새로 만들어 복사하기 때문에 시간을 너무 많이 잡아먹어서 그런 거였다.

복사가 아니라 참조로 넘겨주기 위해 binary\_search(&벡터, first, last, 찾을 문자열)의 형태로 바꿔주니 성공.

​

+) 문자열간의 사전순 비교는 a.compare(b)를 통해서 할 수 있다. 쉽게 생각해서 a-b라고 보면 될듯.

> 2. 소스코드

소스코드는 다음과 같다.

#include \<iostream\>#include \<vector\>#include \<algorithm\>int binary\_search(std::vector\<std::string\>& S, int first, int last, std::string str) {if (last-first \< 0) return -1; int mid = (last - first + 1) / 2 + first; int diff = S[mid].compare(str);if (diff\>0) {return binary\_search(S, first, mid - 1, str);}else if (diff\<0) {return binary\_search(S, mid + 1, last, str);}else if (diff==0) {return mid;}else return -1;}int main() { int N, M; std::cin \>\> N \>\> M; std::vector\<std::string\> S;for (int i = 0; i \< N; i++) { std::string input; std::cin \>\> input;S.push\_back(input);} std::sort(S.begin(), S.end()); int count = 0;for (int i = 0; i \< M; i++) { std::string input; std::cin \>\> input; int index = binary\_search(S, 0, S.size() - 1, input);if (index != -1) { count++;}} std::cout \<\< count;}

  

​

​

참고로, 이 문제는 굳이 이진탐색을 직접 구현하지 않고 기본 자료구조인 map을 사용해도 해결 할 수 있다.

> std::map\<키, 값\> 자료구조

맵의 탐색, 삽입, 삭제는 O(log n)의 시간복잡도를 갖는다.

즉, 그냥 map.find(키값) 함수를 사용하면 알아서 이진탐색을 해서 이터레이터를 반환해주고, map.insert(값)을 넣으면 알아서 내부적으로 집어넣으며 정렬이 되기 때문에 굉장히 편하다.

#include \<iostream\>#include \<map\>int main() { int N, M; std::cin \>\> N \>\> M; std::map\<std::string, int\> S;for (int i = 0; i \< N; i++) { std::string input; std::cin \>\> input;S.insert(std::pair\<std::string, int\>(input, 1));} int count = 0;for (int i = 0; i \< M; i++) { std::string input; std::cin \>\> input;if(S.find(input) != S.end()) count++;} std::cout \<\< count;}

​

​

​

+)

덧붙여, 이 문제는 문자열간의 비교 연산에서 시간을 많이 잡아먹는 것 같다.

이 시간을 절약하기 위해 해싱을 사용하면 시간복잡도가 O(nlog n)이 되어 베스트겠지만 공부할 게 너무 많아지니 해싱은 다음 기회에...

