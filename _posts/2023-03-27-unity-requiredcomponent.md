--- layout: post title: "[Unity] 명시적 의존성 표기하기 RequiredComponent" date: '2023-03-27T20:16:00.005-07:00' author: 가도 tags: - Unity modified\_time: '2023-03-27T20:16:31.727-07:00' blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-7598743322595832364 blogger\_orig\_url: https://sealbinary.blogspot.com/2023/03/unity-requiredcomponent.html --- 유니티 클래스간에 의존성이 필요할 때가 있다.

  

가령, Player 클래스가 있고, Player의 움직임을 담당하는 PlayerController가 있다고 하면

Player는 혼자서도 존재할 수 있는 반면(움직이진 못하겠지만) PlayerController는 Player가 같은 오브젝트에 붙어있어야만 동작할 수 있다.

  

  

이럴 때 의존성을 명시적으로 표기해주면, PlayerController를 부착할 때 Player도 자동으로 추가되게 할 수 있다.

만약 의존관계인 클래스가 없으면 에러가 표시돼서 문제를 찾아내기 쉽다.

  

  

| 

1

2

3

4

 | 

[RequipredComponent(typeof(Player))]

class&nbsp;PlayerController&nbsp;:&nbsp;MonoBehaviour{

&nbsp;

}

[Colored by Color Scripter](https://colorscripter.com/info#e)
 | [cs](https://colorscripter.com/info#e) |

  

  
이런 식으로 쓴다.
