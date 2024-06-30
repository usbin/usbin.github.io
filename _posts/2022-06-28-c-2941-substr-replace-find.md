--- layout: post title: "[C++] 백준 알고리즘 2941번 - 문자열 substr, replace, find" date: '2022-06-28T08:00:00.002-07:00' author: 가도 tags: - 자료구조/알고리즘 - C-plusplus modified\_time: '2022-08-12T08:11:45.150-07:00' thumbnail: https://blogger.googleusercontent.com/img/a/AVvXsEgdMwg2-GVAR6rLeHND2Gtslj\_NC6c2r8NUpf\_Idgd8fmx\_aFKa-WF\_3rDkj\_7BoT6ualK6JBLCGwK5EOxB3pBTe3V7qpjaDDwK5WQFRYEQnYHaiUdx\_jrDmxNs6yDAsIWN3DBOq73odeWyGVs91q-\_tv\_Lms3tzoih\_OHIRfiORhgz4heBUCH67ElVyQ=s72-c blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-6593971499442973147 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/06/c-2941-substr-replace-find.html ---

&nbsp;[https://www.acmicpc.net/problem/2941](https://www.acmicpc.net/problem/2941)

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgdMwg2-GVAR6rLeHND2Gtslj_NC6c2r8NUpf_Idgd8fmx_aFKa-WF_3rDkj_7BoT6ualK6JBLCGwK5EOxB3pBTe3V7qpjaDDwK5WQFRYEQnYHaiUdx_jrDmxNs6yDAsIWN3DBOq73odeWyGVs91q-_tv_Lms3tzoih_OHIRfiORhgz4heBUCH67ElVyQ=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEgdMwg2-GVAR6rLeHND2Gtslj_NC6c2r8NUpf_Idgd8fmx_aFKa-WF_3rDkj_7BoT6ualK6JBLCGwK5EOxB3pBTe3V7qpjaDDwK5WQFRYEQnYHaiUdx_jrDmxNs6yDAsIWN3DBOq73odeWyGVs91q-_tv_Lms3tzoih_OHIRfiORhgz4heBUCH67ElVyQ)
  
  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhQTq4OsSSVmIHuf0lWYbdm2GZx1xrMLcJKmjeg0tGpsg88NdiaUQ0K2wFgpKUyszQJmompgXC2a9oQt2LdPsFB4HMcG8nAxxX2sPsAyfURX02cJ6pUEb5depfgwELXSVmfxSSw6rMN2he533ec1_jU12IdzGn7NutYjHK_VqdnwiaYQMQZ-xdbHijyNQ=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEhQTq4OsSSVmIHuf0lWYbdm2GZx1xrMLcJKmjeg0tGpsg88NdiaUQ0K2wFgpKUyszQJmompgXC2a9oQt2LdPsFB4HMcG8nAxxX2sPsAyfURX02cJ6pUEb5depfgwELXSVmfxSSw6rMN2he533ec1_jU12IdzGn7NutYjHK_VqdnwiaYQMQZ-xdbHijyNQ)
  
  

이 문제는 일단 두 가지 방법으로 풀었는데, 첫번째 방법이 영 찜찜해서 한 번 더 풀었다.

근데 둘다 메모리랑 시간은 똑같이 나왔음.

​

아무튼 첫번째 소스코드는 다음과 같다.

#include \<iostream\>#include \<string\>int main() {//1. 단어 입력받기//2. 알파벳 각각에 대해 몇 개가 있는지 찾고 특정 문자로 대체//3. 단어에서 특정 문자 전부 삭제//4. 단어의 길이+앞에서 센 개수 더하기//5. 출력//1. 단어 입력받기 std::string word; std::cin \>\> word;//2. 단어 배열을 만들고 각 단어가 몇 개 있는지 찾아 세고 특정 문자(@)로 대체 std::string wordBook[8] = {"c=", "c-", "dz=", "d-", "lj", "nj", "s=", "z="}; int apbCount = 0;for (int i = 0; i \< (sizeof(wordBook) / sizeof(wordBook[0])); i++) { int pos = 0;while (true) { pos = word.find(wordBook[i], pos);if (pos == -1) break; apbCount++; word.replace(pos, wordBook[i].length(), "@");}}//3. 2번에서 대체한 문자를 전부 삭제. int pos = 0;while (true) { pos = word.find("@", pos);if (pos == -1) break; word.replace(pos, 1, "");}//4. 2번에서 센 개수 + 3번 결과로 나온 단어의 길이를 세어서//5. 출력 std::cout \<\< apbCount+word.length();}

일단 단어가

| 

ljes=njak

 |

이런 식으로 있다면,

> 2번 과정: 단어장에 있는 것만 먼저 찾아 @로 대체한다.

lj, s=, nj가 있다.

단어 개수: 3개.

@로 대체한 결과:

| 

@e@@ak

 |

\*바로 삭제하지 않고 @로 먼저 대체한 다음, 3번 과정에서 일괄적으로 삭제하는 이유가 있다. 먼저 삭제해버리면 앞뒤의 알파벳이 붙게 되어 우연히 또 단어장의 단어를 만들어내는 경우가 생긴다.

가령, ls=jabcdef 라는 단어가 있다면, s=가 단어장에 있으므로 먼저 삭제된다. 그럼 삭제된 s=의 앞뒤에 있던 l과 j가 만나 ljabcdef가 된다. 그런데 lj라는 단어는 단어장에 있으므로 또 하나의 알파벳으로 세어진다.

​

> 3번 과정: @를 전부 삭제한다.

| 

eak

 |

> 4번 과정: 2번에서 나온 개수와 3번 과정 결과물의 문자열 길이를 더한다.

3 + 3 = 6

​

결과는 6.

​

​

* * *

두번째 소스코드는 다음과 같다.

#include \<iostream\>#include \<string\>//﻿단어장의 단어와 일치하는 게 있다면 그 길이를, 없다면 1을 리턴한다.int getNextApbLength(std::string word, int pos){ std::string wordBook[8] = {"c=", "c-", "dz=", "d-", "lj", "nj", "s=", "z="};for(int i=0; i\<8; i++){if(word.substr(pos, wordBook[i].length())==wordBook[i]){return wordBook[i].length();}}return 1;}int main() { std::string word; std::cin \>\> word; int apbCount = 0; int pos=0;//입력받은 단어의 첫번째 위치부터 마지막까지 읽으며, 매번 단어장의 단어와 비교한다.while(pos\<word.length()){ int length = getNextApbLength(word, pos); pos+=length; apbCount++;} std::cout \<\< apbCount;}

보기엔 이 코드가 더 깔끔한데 효율은 비슷한듯.

시간복잡도를 계산해보니 첫번째 코드는 n^2이고 두번째 코드도 n^2이다.

그래도 두번째 코드가 더 나은 것 같다. 첫번째는 좀 바보코딩 같음.

​

