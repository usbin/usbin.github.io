--- layout: post title: "[Unity/C#] for문의 index를 람다식에서 사용시 주의할 점" date: '2022-07-05T08:00:00.004-07:00' author: 가도 tags: - Unity - C# modified\_time: '2022-08-12T08:12:02.308-07:00' blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-2700753875910433183 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/07/unityc-for-index.html ---

for(int i=0; i\<10; i++){ myButtons[i].onClick.AddListener(()=\>{ Debug.Log(i);});}

이 코드를 짜놓고 저 myButtons의 버튼들을 누르면 어떤 버튼을 누르든 10이 출력된다.

i번째 버튼을 누르면 i라는 숫자를 로그로 출력하는 코드를 짰는데 대체 왜?

​

먼저, 이걸 해결하려면 루프문 안에서 로컬변수를 새로 생성하고, 그 로컬변수를 사용해주면 된다.

for(int i=0; i\<10; i++){ int index=i; myButtons[i].onClick.AddListener(()=\>{ Debug.Log(index);});}

이렇게.

​

이게 뭔 의미가 있을까? 왜 이렇게 사용하면 제대로 동작할까?

임시 클래스를 만드는 시점이 바뀌기 때문이다.

> C#의 람다식

| 

C#에서 람다식을 쓰면 람다함수 내에서 사용하는 변수와 코드를 임시 클래스로 만들어서 쓴다고 한다.

​

위의 코드는 루프문 바깥에서 임시클래스를 만들고 멤버변수로 i를 할당한다.

그래서 루프를 다 도는 동안 하나의 임시클래스를 계속 참조하며 마찬가지로 이 임시클래스의 멤버함수로 들어간 람다식을 AddListener에 집어넣는다. 처음부터 마지막 myButton에는 모두 하나의 임시클래스가 들어간 셈이다. 그리고 i는 당연히 매 루프마다 업데이트 되고, 마지막에 업데이트한 상태(10을 담고 있는 상태)로 남아있다가, myButton을 누르는 순간 호출될 것이다.

​

반면 아래 코드는 람다식 안에서 사용하는 변수인 index를 for 루프문 안에서 만들었으므로 다음 루프를 돌 땐 또 새로운 임시클래스를 만든다. 따라서 각각의 myButton에는 다른 임시클래스가 들어간다. 이 경우 i는 업데이트되지 않는다.

 |

> 결론

for문 안에서 람다식을 쓰고, 그 람다식 안에서 for문의 index를 쓴다면, 바로 사용하지 말고 반드시 for문 안에서 로컬변수를 새로 만들고 그 안에 담아서 사용하자. 아래 코드처럼.

for(int i=0; i\<10; i++){ int index=i; myButtons[i].onClick.AddListener(()=\>{ Debug.Log(index);});}

​

​

이 포스팅은 아래의 블로그를 참조했다. 소스코드까지 예를 들며 설명해주고 있으니 이 부분이 이해가 안 될 땐 이걸 보자.

[https://enghqii.tistory.com/49](https://enghqii.tistory.com/49)

