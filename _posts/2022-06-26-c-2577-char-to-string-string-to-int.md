--- layout: post title: "[C++] 백준 알고리즘 2577번 - 특정 숫자로 배열 초기화, char to string, string to int" date: '2022-06-26T08:00:00.007-07:00' author: 가도 tags: - 자료구조/알고리즘 - C-plusplus modified\_time: '2022-08-12T08:12:16.502-07:00' thumbnail: https://blogger.googleusercontent.com/img/a/AVvXsEgI8kqT5wJcUbgbx-2iomx7qZCYDI6RLVHbeo6Q-hz6XugLTvRALNfLs8-TbpbNG8E8TXh0FjZxsg9kShK7gEHqJl-MCYCaW6Kkk9xEUkv1Z-uZKg7frsziphChlO-Y4dfSNWG1JvkergA0pNx5jH2sbGh9BzHmUYLtTzAtHnCXJl\_OpdE0ojHm3eQr8A=s72-c blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-7439606083605085222 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/06/c-2577-char-to-string-string-to-int.html ---

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgI8kqT5wJcUbgbx-2iomx7qZCYDI6RLVHbeo6Q-hz6XugLTvRALNfLs8-TbpbNG8E8TXh0FjZxsg9kShK7gEHqJl-MCYCaW6Kkk9xEUkv1Z-uZKg7frsziphChlO-Y4dfSNWG1JvkergA0pNx5jH2sbGh9BzHmUYLtTzAtHnCXJl_OpdE0ojHm3eQr8A=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEgI8kqT5wJcUbgbx-2iomx7qZCYDI6RLVHbeo6Q-hz6XugLTvRALNfLs8-TbpbNG8E8TXh0FjZxsg9kShK7gEHqJl-MCYCaW6Kkk9xEUkv1Z-uZKg7frsziphChlO-Y4dfSNWG1JvkergA0pNx5jH2sbGh9BzHmUYLtTzAtHnCXJl_OpdE0ojHm3eQr8A)
  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEg1Ss7356J_45XzeFNEnix59q8ISm5B0Amz5IJwlSXv-U2YzGkhbmJ3m_Ze-oeXw7mVFA9DQfN69E_rbmPXoSK8aQl2dhtBH8wYbxSX0ZgCaMD_Z9EQPHH96xX25uEAguAkxOetULvK7fix4B5tFQ4LFmegr6gmTZb_MILKtbWp222NIOkdDqJas8QC0A=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEg1Ss7356J_45XzeFNEnix59q8ISm5B0Amz5IJwlSXv-U2YzGkhbmJ3m_Ze-oeXw7mVFA9DQfN69E_rbmPXoSK8aQl2dhtBH8wYbxSX0ZgCaMD_Z9EQPHH96xX25uEAguAkxOetULvK7fix4B5tFQ4LFmegr6gmTZb_MILKtbWp222NIOkdDqJas8QC0A)
  
  

  

슬슬 알고리즘다운 알고리즘이 나오기 시작했다.

숫자의 집합을 다루는 데에는 여러 방법이 있을 텐데, 값을 접근하는 데에는 배열이 가장 좋고 여기가 배열 카테고리 문제기도 해서, 배열을 사용했다.

​

일단 생각한 건

방법1. A\*B\*C 결과값을 %연산을 사용해 1의 자리, 10의 자리, 100의 자리의 숫자를 하나씩 읽으며 세는 방법

방법2. string으로 다루어 접근하는 방법

두 가지 방법이 있겠는데 난 (2)가 더 쉬워보여서 (2)로 했다.

​

(1) 먼저 0부터 9까지의 숫자가 몇 번 쓰였는지 세고 저장하기 위한 10칸짜리 배열을 선언하고 초기화한다.

int countArray[10];std::fill\_n(countArray, 10, 0);

std::fill\_n(배열, 크기, 초기화할 값);

이 코드 자주 쓸 것 같으니 외워두자.

​

(2) A\*B\*C 연산을 한 다음 string으로 저장한다.

int A, B, C;std::cin \>\> A \>\> B \>\> C;int result = A \* B \* C;std::string strNum = std::to\_string(result);

저번에 포스팅했던 int to string 방법으로 std::to\_string(숫자)를 쓴다.

​

(3) 이제 string의 0번째 인덱스부터 length-1번째 인덱스까지 접근하며 해당 값을 세어서 아까 만든 배열에 저장해준다.

for (int i = 0; i \< strNum.length(); i++) { countArray[std::stoi(std::string(1, strNum[i]))]++;}

여기서 std::stoi(스트링)은 string to int로의 변환을 위한 함수이고,

std::string(반복 횟수, 반복할 문자)는 저번에도 포스팅했던 문자 반복 함수이다. 한 글자짜리 char 타입을 string으로 바꾸기 위해 사용했다.

​

​

결과 코드는 아래.

#include \<iostream\>#include \<string\>int main() { int A, B, C; int countArray[10]; std::fill\_n(countArray, 10, 0); std::cin \>\> A \>\> B \>\> C; int result = A \* B \* C; std::string strNum = std::to\_string(result);for (int i = 0; i \< strNum.length(); i++) { countArray[std::stoi(std::string(1, strNum[i]))]++;}for (int i = 0; i \< 10; i++) { std::cout \<\< countArray[i] \<\< "\n";}}

​

  

  

