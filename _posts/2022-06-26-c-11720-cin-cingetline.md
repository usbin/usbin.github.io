--- layout: post title: "[C++] 백준 알고리즘 11720번 - cin\>\>과 cin.getline()의 혼용에서 발생하는 문제" date: '2022-06-26T08:00:00.003-07:00' author: 가도 tags: - 자료구조/알고리즘 - C-plusplus modified\_time: '2022-08-12T08:11:45.156-07:00' thumbnail: https://blogger.googleusercontent.com/img/a/AVvXsEicPymzTXFZlQUeWeC9rYTFp\_2eikt8hE\_nk7RuKhZMiie8Rm44Es9\_i2w\_e9-GPui6Xg6PJ0e6MIL91gLm9I1MYvAHlchq6jTvAxtvHSjZ-ojS7VwEd8Xx8z83GBP0qCSs3xQAFC2wCRCWr8PMoH28yRdHtCUohs6zYQGyNJTTndXbEAeMOo8JDDsiuA=s72-c blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-8220790358556713832 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/06/c-11720-cin-cingetline.html ---

&nbsp;

[![](https://blogger.googleusercontent.com/img/a/AVvXsEicPymzTXFZlQUeWeC9rYTFp_2eikt8hE_nk7RuKhZMiie8Rm44Es9_i2w_e9-GPui6Xg6PJ0e6MIL91gLm9I1MYvAHlchq6jTvAxtvHSjZ-ojS7VwEd8Xx8z83GBP0qCSs3xQAFC2wCRCWr8PMoH28yRdHtCUohs6zYQGyNJTTndXbEAeMOo8JDDsiuA=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEicPymzTXFZlQUeWeC9rYTFp_2eikt8hE_nk7RuKhZMiie8Rm44Es9_i2w_e9-GPui6Xg6PJ0e6MIL91gLm9I1MYvAHlchq6jTvAxtvHSjZ-ojS7VwEd8Xx8z83GBP0qCSs3xQAFC2wCRCWr8PMoH28yRdHtCUohs6zYQGyNJTTndXbEAeMOo8JDDsiuA)
  
  
  

#include \<iostream\>int main(){ int N; std::cin\>\>N; std::cin.ignore(); int sum = 0;//한 글자씩 읽기for(int i=0; i\<N; i++){ char c; c = std::cin.get(); sum += (c - '0');} std::cout\<\<sum;}

짧은 코드인데 std::cin\>\>과 std::cin.get() 또는 std::cin.getline()을 혼용할 때 생길 수 있는 문제가 있어서 포스팅함.

​

std::cin\>\>N으로 입력 스트림을 읽으면 공백이나 엔터를 만날 때까지 입력 스트림을 읽어들인다. 근데 공백이나 엔터를 처리하진 않고 딱 앞의 것까지만 가져온다. 다음에 읽을 때에도 문자열의 앞에 붙은 공백이나 엔터는 무시하고 제대로 다음 문자열을 읽어들인다.

std::cin.getline(담을 스트링)으로 입력 스트림을 읽으면 엔터를 만날 때까지 입력 스트림을 읽어들인다. 그리고 마지막의 엔터를 스트림에서 처리해서 없애버린다. 따라서 얘는 자기가 읽는 문자열의 앞에는 당연히 엔터가 붙어있지 않을 것이라 예상하고 읽는다.

​

﻿도레미 파솔라 시도

위처럼 입력된 걸 cin\>\>과 cin.getline을 혼용하여 읽는다고 생각해보자.

​

﻿//입력 스트림: "도레미\n파솔라\n시도\n"std::string str std::cin\>\>str//str: "도레미"//입력 스트림: "\n파솔라\n시도\n"std::cin\>\>str//str: "파솔라"//입력 스트림: "\n시도\n"

이 다음에 std::cin.getline()으로 읽어보자.

std::cin.getline(str);//str: "\n"//입력 스트림: "시도\n"

"시도" 대신 엔터가 들어와버렸다. 만약 cin.get()으로 읽었대도 마찬가지였을 거다. '\n'라는 한 글자짜리 char 타입이라는 것만 다를 뿐.

즉, cin\>\>은 엔터를 스트림에서 없애지 않고 남겨둔다.

cin.getline()은 엔터를 스트림에서 없앤다.

둘을 혼용할 때의 문제는 이 차이 때문에 발생한다.

그냥 혼용을 안 하면 좋겠지만 굳이 해야 한다면 해결방법은, cin\>\>을 사용한 다음 cin.ignore()으로 마지막 글자를 없애주는 것이다.

​

> cin.ignore(count, delimeter) : cin 스트림을 count만큼 비워주는데, 중간에 delimeter를 만나면 그것까지만 비워줌.
> 
> cin.ignore(): cin 스트림에서 한 글자를 비운다.

​

