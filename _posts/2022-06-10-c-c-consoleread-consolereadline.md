---
layout: post
title: C#공부 - C#의 엔트리 포인트, Console.Read()와 Console.ReadLine()의 차이점
date: '2022-06-10T08:00:00.001-07:00'
author: 가도
categories: [cs공부]
modified\_time: '2022-08-12T00:53:38.562-07:00'
blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-1011547443174258950
blogger\_orig\_url: https://sealbinary.blogspot.com/2022/06/c-c-consoleread-consolereadline.html
---

&nbsp;요즘 너무 공부를 안 해서 매일 코테 5문제씩 풀기로 했다.

C#도 놓은 지 한참이라 코테 1번 문제부터 초심으로 돌아가 푸는 중이다.

아래는 코테 풀면서 관련된 기초 문법+지식 정리한 것.

​

1. C#의 엔트리 포인트

C# 프로그램은 Main이라는 이름의 static 함수의 첫 줄에서부터 시작한다.

클래스의 이름과 Main 함수의 리턴형, 파라미터에 영향을 받지 않는다.

ex)

​
```c#
class Program{
    public static void Main(){//여기서부터 시작
    }
}
```
​

가장 간소한 프로그램의 형태는 위와 같다.

접근제한자는 기본값이 private인데, 이를 public으로 바꾸어도 영향을 받지 않는다.

```c#
public class Program{
    public static void Main(){//여기서부터 시작
    }
}
```
​

Main 함수의 리턴형과 인자를 마음대로 바꾸어도 영향을 받지 않는다.

```c#
class Program{
    public static int Main(string[] args){//여기서부터 시작
        return 0;
    }
}
```
클래스 이름을 마음대로 바꾸어도 상관없다.

```c#
class Chicken{
    public static void Main(){//여기서부터 시작
    }
}
```
​

중요한 건 Main이라는 함수가 프로그램 내에 단 하나만 존재해야 한다는 것. 그리고 이 Main 함수가 static 함수여야 한다는 것.

​

​

2. Console.Read()와 Console.ReadLine()의 차이점

c#에서 입력을 받으려면 Console.Read()나 Console.ReadLine()을 사용할 수 있다.

​

Console.Read()의 반환값은 int인데, 입력받은 문자의 아스키코드 값을 반환한다. 가령, A를 입력받으면 A의 아스키 값인 65를 반환한다. 한 번에 하나의 문자만 읽는다.

Console.ReadLine()의 반환값은 string이다. 엔터가 나올 때까지 모든 문자를 읽어 string으로 반환한다. 가령, A를 입력받으면 string 타입의 "A"를 반환한다.

​

입력이

1 3

이렇게 들어올 때, 두 숫자를 더해야 하는 문제가 나왔다고 생각해보자.

```c#
int a = Console.Read();
Console.Read();
int b = Console.Read();
Console.WriteLine(a+b):
```
​

단순히 생각해서 이런 코드를 짜면 틀린다.

첫줄에서 a에는 1이 아니라 문자 '1'의 아스키코드 값인 49가 들어간다. b에는 51이 들어갈 것이다.

번거로워도 Console.ReadLine().Split(' ');를 이용하는 게 좋겠다.

