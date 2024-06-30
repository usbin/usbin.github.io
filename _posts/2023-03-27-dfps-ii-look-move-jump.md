--- layout: post title: "[DFPS] II. 플레이어 조작과 공격 : 모드에 따라 다른 구현" date: '2023-03-27T08:09:00.008-07:00' author: 가도 tags: - 포트폴리오\_Unity modified\_time: '2023-03-28T03:14:47.368-07:00' thumbnail: https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh7e4GE9ym3xPUcmpmY74XfesNGKHUNHoqelSV8IDhB1LW3Epsc\_LHQy0OPRRjNHIEXFpATpiJTfo60TtkS9unQA0ZC21JGYIReSF7f9VPA8yAGoyRUY1NcRCGNHgSBnn8oSa3cp-49ByQ5e\_xdSWeyDyXbhVcR\_\_oTFoTGkYMe8N9aeI9DYbQF\_nt9ew/s72-w640-c-h538/KakaoTalk\_20230327\_234624226.jpg blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-4068731320506068336 blogger\_orig\_url: https://sealbinary.blogspot.com/2023/03/dfps-ii-look-move-jump.html --- 먼저 1인칭 뷰와 탑다운 뷰의 구현상의 차이는 크게 세가지다.

  

1. Look() 함수의 차이

1인칭 뷰는 마우스가 움직인 양만큼 시야를 이동시켜서 1인칭 느낌을 내지만,

탑다운 뷰는 마우스를 움직이면 캐릭터가 스크린상의 마우스 방향을 곧바로 쳐다보도록 해야 자연스럽다.

  

2. Move() 함수의 차이

1인칭 뷰는 상하좌우키로 캐릭터를 움직일 때 현재 캐릭터가 바라보고 있는 방향을 기준으로 이동하지만,

탑다운 뷰는 스크린에서 바라보고 있는 화면을 기준으로 이동해야 한다.

만약 탑다운 뷰에서도 캐릭터가 바라보는 방향을 기준으로 움직이면 캐릭터가 정면을 바라보지 않을 땐 플레이어가 조작을 헷갈려할 것.

  

3. Attack() 함수의 차이

1인칭 뷰는 총을 사용할 경우 화면의 가운데에 조준점이 있고, 적이 이 조준점 안에 들어왔을 때 총을 쏘면 타격 판정이 된다.

탑다운 뷰는 총을 사용할 경우 플레이어의 총구에서부터 마우스 커서까지가 발사 방향이 되고 타격 판정의 기준이 된다.

  

  

먼저, Attack 함수는 공격의 시작점과 공격 방향을 정의해서 파라미터로 넘겨줌으로써 두 모드의 동작을 통일시킬 수 있다.

  

그러니까 Look()과 Move()만 다르게 구현하면 된다.
  

  

플레이어의 움직임은 PlayerController가 담당하고,  
무기 장착 및 공격은 WeaponController가 담당하게 할 것이다.
  

커플링을 줄이기 위해 컨트롤러의 동작은 동일하되 내부에 컨트롤을 각각 갖고 있도록 한다.

  

  

#### 움직임 담당

PlayerController

ㄴPlayerControl\_FirstPerson //1인칭일 때 컨트롤 담당

ㄴPlayerControl\_TopdownView //탑다운일 때 컨트롤 담당

  

  

두 컨트롤은 PlayerControl이라는 추상클래스로 묶어준다.

  

PlayerController는 currentControl 멤버변수에 현재 뷰 모드에서 쓰이는 컨트롤을 담고, 특정 동작에 대해 공통된 함수를 실행시킬 것이다.

  

  

#### 공격 담당

WeaponController  

ㄴ 장착한 무기 : Weapon
ㄴ void EquipWeapon(Weapon)

ㄴ void 공격()

Weapon은 추상클래스로, Gun이나 Sword 등 구체 클래스를 상속해서 만들어 사용한다.

  

WeaponController는 1인칭일 땐 화면 중앙을 향해 공격이 뻗어나간다고 정의하고, 탑다운일 땐 마우스 방향을 향해 공격이 뻗어나간다고 정의한다.

  

#### 이벤트 구독

또한 PlayerController와 WeaponController는 "뷰모드 변경" 이벤트를 구독하고, 변경될 때의 동작을 정의해준다.

  

"뷰모드 변경" 이벤트는 Director라는 별도의 클래스에서 발행할 것이다.

  

  

  

대략 이런 형태.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh7e4GE9ym3xPUcmpmY74XfesNGKHUNHoqelSV8IDhB1LW3Epsc_LHQy0OPRRjNHIEXFpATpiJTfo60TtkS9unQA0ZC21JGYIReSF7f9VPA8yAGoyRUY1NcRCGNHgSBnn8oSa3cp-49ByQ5e_xdSWeyDyXbhVcR__oTFoTGkYMe8N9aeI9DYbQF_nt9ew/w640-h538/KakaoTalk_20230327_234624226.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh7e4GE9ym3xPUcmpmY74XfesNGKHUNHoqelSV8IDhB1LW3Epsc_LHQy0OPRRjNHIEXFpATpiJTfo60TtkS9unQA0ZC21JGYIReSF7f9VPA8yAGoyRUY1NcRCGNHgSBnn8oSa3cp-49ByQ5e_xdSWeyDyXbhVcR__oTFoTGkYMe8N9aeI9DYbQF_nt9ew/s1200/KakaoTalk_20230327_234624226.jpg)
  

  

  

  

아래는 소스코드.

  

#### Player

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

using&nbsp;UnityEngine.InputSystem;

using&nbsp;UnityEngine.InputSystem.UI;

using&nbsp;UnityEngine.Events;

&nbsp;

public&nbsp;class&nbsp;Player&nbsp;:&nbsp;LivingEntity

{

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Camera&nbsp;ViewCamera&nbsp;{&nbsp;get&nbsp;=\>&nbsp;\_viewCamera;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;Camera&nbsp;\_viewCamera;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;enum&nbsp;ViewMode&nbsp;{&nbsp;FIRST\_PERSON,&nbsp;TOPDOWN&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;override&nbsp;void&nbsp;Start()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;base.Start();

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;Update()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;OnViewModeChanged(Director.ViewModeChangedArgs&nbsp;args)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_viewCamera&nbsp;=&nbsp;args.ViewCamera;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

}

&nbsp;

[Colored by Color Scripter](https://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

#### LivingEntity

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

&nbsp;

public&nbsp;class&nbsp;LivingEntity&nbsp;:&nbsp;MonoBehaviour,&nbsp;IDamagable,&nbsp;ICombatable

{

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;float&nbsp;MaxHp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;float&nbsp;Speed;

&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;float&nbsp;hp;

&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;bool&nbsp;dead&nbsp;=&nbsp;false;

&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;\_atk;

&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;\_def;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;float&nbsp;Atk&nbsp;=\>&nbsp;\_atk;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;float&nbsp;Def&nbsp;=\>&nbsp;\_def;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;virtual&nbsp;void&nbsp;Start()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;hp&nbsp;=&nbsp;MaxHp;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;virtual&nbsp;void&nbsp;Die()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;dead&nbsp;=&nbsp;true;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Destroy(gameObject);

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;TakeHit(float&nbsp;damage)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;hp&nbsp;-=&nbsp;damage;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(hp&nbsp;\<=&nbsp;0&nbsp;&&&nbsp;!dead)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Die();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;}

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

  

#### ICombatable

| 

1

2

3

4

5

6

7

8

9

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

&nbsp;

public&nbsp;interface&nbsp;ICombatable

{

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;abstract&nbsp;float&nbsp;Atk&nbsp;{&nbsp;get;&nbsp;}

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

  

#### IDamagable

| 

1

2

3

4

5

6

7

8

9

10

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

&nbsp;

public&nbsp;interface&nbsp;IDamagable

{

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;abstract&nbsp;float&nbsp;Def&nbsp;{&nbsp;get;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;TakeHit(float&nbsp;damage);

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

  

#### PlayerController

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

using&nbsp;UnityEngine.InputSystem;

&nbsp;

[RequireComponent(typeof(LivingEntity))]

[RequireComponent(typeof(PlayerInput))]

[RequireComponent(typeof(Rigidbody))]

public&nbsp;class&nbsp;PlayerController&nbsp;:&nbsp;MonoBehaviour

{

&nbsp;&nbsp;&nbsp;&nbsp;Player&nbsp;\_player;

&nbsp;&nbsp;&nbsp;&nbsp;PlayerInput&nbsp;\_input;

&nbsp;&nbsp;&nbsp;&nbsp;Rigidbody&nbsp;\_playerRigidbody;

&nbsp;&nbsp;&nbsp;&nbsp;PlayerControl[]&nbsp;\_controls&nbsp;=&nbsp;new&nbsp;PlayerControl[2];

&nbsp;&nbsp;&nbsp;&nbsp;PlayerControl&nbsp;\_currentControl;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;bool&nbsp;JumpCooldown;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;Awake()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_player&nbsp;=&nbsp;GetComponent\<Player\>();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_input&nbsp;=&nbsp;GetComponent\<PlayerInput\>();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody&nbsp;=&nbsp;GetComponent\<Rigidbody\>();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_controls[(int)Player.ViewMode.FIRST\_PERSON]&nbsp;=&nbsp;new&nbsp;PlayerControl\_FirstPerson(\_playerRigidbody);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_controls[(int)Player.ViewMode.TOPDOWN]&nbsp;=&nbsp;new&nbsp;PlayerControl\_Topdown(\_playerRigidbody);

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;void&nbsp;Start()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;Update()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ControlArgs&nbsp;args;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.Input&nbsp;=&nbsp;\_input;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.Player&nbsp;=&nbsp;\_player;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.PlayerController&nbsp;=&nbsp;this;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_currentControl.Update(args);

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;FixedUpdate()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_currentControl.ApplyChange();

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;OnCollisionEnter(Collision&nbsp;collision)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(collision.collider.CompareTag("Ground"))&nbsp;JumpCooldown&nbsp;=&nbsp;true;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;OnViewModeChanged(Director.ViewModeChangedArgs&nbsp;args)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_currentControl&nbsp;=&nbsp;\_controls[(int)args.ViewMode];

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;struct&nbsp;ControlArgs

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;PlayerInput&nbsp;Input;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Player&nbsp;Player;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;PlayerController&nbsp;PlayerController;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

  

#### PlayerControl\_FirstPerson

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89

90

91

92

93

94

95

96

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

using&nbsp;UnityEngine.InputSystem;

public&nbsp;class&nbsp;PlayerControl\_FirstPerson&nbsp;:&nbsp;PlayerControl

{

&nbsp;&nbsp;&nbsp;&nbsp;Rigidbody&nbsp;\_playerRigidbody;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;//다음&nbsp;프레임에&nbsp;적용할&nbsp;값들

&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;\_movement;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;이동

&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;\_deltaLookDegree;&nbsp;&nbsp;&nbsp;//&nbsp;시야

&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;\_jumpForce;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;점프

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;PlayerControl\_FirstPerson(Rigidbody&nbsp;playerRigidbody)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody&nbsp;=&nbsp;playerRigidbody;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;override&nbsp;void&nbsp;Update(PlayerController.ControlArgs&nbsp;args)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PlayerInput&nbsp;input&nbsp;=&nbsp;args.Input;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Player&nbsp;player&nbsp;=&nbsp;args.Player;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PlayerController&nbsp;controller&nbsp;=&nbsp;args.PlayerController;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;이동

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;moveInput&nbsp;=&nbsp;input.actions["Move"].ReadValue\<Vector2\>();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;moveDirection&nbsp;=&nbsp;new&nbsp;Vector3(moveInput.x,&nbsp;0,&nbsp;moveInput.y).normalized;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Quaternion&nbsp;rotation&nbsp;=&nbsp;Quaternion.Euler(0,&nbsp;player.transform.rotation.eulerAngles.y,&nbsp;0);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;movement&nbsp;=&nbsp;(rotation&nbsp;\*&nbsp;moveDirection)&nbsp;\*&nbsp;Time.deltaTime&nbsp;\*&nbsp;(1&nbsp;+&nbsp;player.Speed)&nbsp;\*&nbsp;5f;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Move(movement);

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;시야

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;lookInputDelta&nbsp;=&nbsp;input.actions["Look"].ReadValue\<Vector2\>();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;lookDeltaDegree&nbsp;=&nbsp;new&nbsp;Vector3(-lookInputDelta.y,&nbsp;lookInputDelta.x,&nbsp;0)&nbsp;\*&nbsp;0.05f;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Rotate(lookDeltaDegree);

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;점프

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(input.actions["Jump"].ReadValue\<float\>()&nbsp;\>&nbsp;0

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&&&nbsp;controller.JumpCooldown)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Jump();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;controller.JumpCooldown&nbsp;=&nbsp;false;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;void&nbsp;Rotate(Vector3&nbsp;deltaDegree)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_deltaLookDegree&nbsp;+=&nbsp;deltaDegree&nbsp;\*&nbsp;15f;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;void&nbsp;Move(Vector3&nbsp;movement)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_movement&nbsp;+=&nbsp;movement;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;void&nbsp;Jump()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_jumpForce&nbsp;+=&nbsp;Mathf.Sqrt(2&nbsp;\*&nbsp;\_playerRigidbody.mass&nbsp;\*&nbsp;Physics.gravity.magnitude&nbsp;\*&nbsp;1);

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;override&nbsp;void&nbsp;ApplyChange()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;이동

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody.MovePosition

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(\_playerRigidbody.position&nbsp;+&nbsp;\_movement);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_movement&nbsp;=&nbsp;Vector3.zero;

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;시야

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//가용범위:&nbsp;-90도(=270도)\<-\>70도

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//위쪽&nbsp;90도까지만&nbsp;올릴&nbsp;수&nbsp;있고

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//아래쪽&nbsp;70도까지만&nbsp;숙일&nbsp;수&nbsp;있음.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;newDegree&nbsp;=&nbsp;\_playerRigidbody.transform.rotation.eulerAngles&nbsp;+&nbsp;\_deltaLookDegree;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(newDegree.x&nbsp;\<&nbsp;270&nbsp;&&&nbsp;newDegree.x&nbsp;\>&nbsp;70)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//가용범위가&nbsp;아닐&nbsp;때

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(\_deltaLookDegree.x&nbsp;\<&nbsp;0)&nbsp;newDegree.x&nbsp;=&nbsp;270;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else&nbsp;newDegree.x&nbsp;=&nbsp;70;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody.transform.eulerAngles&nbsp;=&nbsp;newDegree;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_deltaLookDegree&nbsp;=&nbsp;Vector3.zero;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;점프

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody.AddForce(Vector3.up&nbsp;\*&nbsp;\_jumpForce,&nbsp;ForceMode.Impulse);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_jumpForce&nbsp;=&nbsp;0;

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

#### PlayerControl\_Topdown

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89

90

91

92

93

94

95

96

97

98

99

100

101

102

103

104

105

106

107

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

using&nbsp;UnityEngine.InputSystem;

&nbsp;

public&nbsp;class&nbsp;PlayerControl\_Topdown&nbsp;:&nbsp;PlayerControl

{

&nbsp;&nbsp;&nbsp;&nbsp;Rigidbody&nbsp;\_playerRigidbody;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;//다음&nbsp;프레임에&nbsp;적용할&nbsp;값들

&nbsp;&nbsp;&nbsp;&nbsp;bool&nbsp;\_lookAtDirty&nbsp;=&nbsp;false;

&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;\_movement;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;이동

&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;\_lookAtPos;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;시야

&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;\_jumpForce;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;점프

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;PlayerControl\_Topdown(Rigidbody&nbsp;playerRigidbody)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody&nbsp;=&nbsp;playerRigidbody;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;override&nbsp;void&nbsp;Update(PlayerController.ControlArgs&nbsp;args)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PlayerInput&nbsp;input&nbsp;=&nbsp;args.Input;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Player&nbsp;player&nbsp;=&nbsp;args.Player;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PlayerController&nbsp;controller&nbsp;=&nbsp;args.PlayerController;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;이동&nbsp;:&nbsp;카메라에서&nbsp;본&nbsp;방향으로&nbsp;수정후&nbsp;호출

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;moveInput&nbsp;=&nbsp;input.actions["Move"].ReadValue\<Vector2\>();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;moveDirection&nbsp;=&nbsp;new&nbsp;Vector3(moveInput.x,&nbsp;0,&nbsp;moveInput.y).normalized;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Quaternion&nbsp;rotation&nbsp;=&nbsp;Quaternion.Euler(0,&nbsp;player.ViewCamera.transform.rotation.eulerAngles.y,&nbsp;0);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;movement&nbsp;=&nbsp;(rotation&nbsp;\*&nbsp;moveDirection)&nbsp;\*&nbsp;Time.deltaTime&nbsp;\*&nbsp;(1&nbsp;+&nbsp;player.Speed)&nbsp;\*&nbsp;5f;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Move(movement);

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;시야&nbsp;:&nbsp;마우스&nbsp;방향&nbsp;바라보기(입력이&nbsp;들어올&nbsp;때만)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;InputActionMap&nbsp;playerActionMap&nbsp;=&nbsp;input.actions.FindActionMap("Player");

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if(input.currentActionMap&nbsp;==&nbsp;playerActionMap)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;mousePos&nbsp;=&nbsp;Mouse.current.position.ReadValue();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ray&nbsp;ray&nbsp;=&nbsp;player.ViewCamera.ScreenPointToRay(mousePos);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Plane&nbsp;plane&nbsp;=&nbsp;new&nbsp;Plane(Vector3.up,&nbsp;Vector3.zero);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;distance;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(plane.Raycast(ray,&nbsp;out&nbsp;distance))

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;point&nbsp;=&nbsp;ray.GetPoint(distance);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;point.y&nbsp;=&nbsp;player.transform.position.y;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LookAt(point);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;점프

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(input.actions["Jump"].ReadValue\<float\>()&nbsp;\>&nbsp;0

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&&&nbsp;controller.JumpCooldown)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Jump();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;controller.JumpCooldown&nbsp;=&nbsp;false;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;LookAt(Vector3&nbsp;position)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_lookAtPos&nbsp;=&nbsp;position;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_lookAtDirty&nbsp;=&nbsp;true;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;Move(Vector3&nbsp;movement)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_movement&nbsp;+=&nbsp;movement;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;Jump()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_jumpForce&nbsp;+=&nbsp;Mathf.Sqrt(2&nbsp;\*&nbsp;\_playerRigidbody.mass&nbsp;\*&nbsp;Physics.gravity.magnitude&nbsp;\*&nbsp;1);

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;override&nbsp;void&nbsp;ApplyChange()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;이동

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody.MovePosition

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(\_playerRigidbody.position&nbsp;+&nbsp;\_movement);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_movement&nbsp;=&nbsp;Vector3.zero;

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;시야

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(\_lookAtDirty)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody.transform.LookAt(\_lookAtPos);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_lookAtDirty&nbsp;=&nbsp;false;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;점프

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//=====================

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_playerRigidbody.AddForce(Vector3.up&nbsp;\*&nbsp;\_jumpForce,&nbsp;ForceMode.Impulse);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_jumpForce&nbsp;=&nbsp;0;

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

  

#### WeaponController

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89

90

91

92

93

94

95

96

97

98

99

100

101

102

103

104

105

106

107

108

109

110

111

112

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

using&nbsp;UnityEngine.InputSystem;

&nbsp;

&nbsp;

[RequireComponent(typeof(Player))]

[RequireComponent(typeof(PlayerInput))]

public&nbsp;class&nbsp;WeaponController&nbsp;:&nbsp;MonoBehaviour

{

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Transform&nbsp;GunHold;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Weapon&nbsp;InitialWeapon;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;Weapon&nbsp;equippedWeapon;

&nbsp;&nbsp;&nbsp;&nbsp;Player&nbsp;\_player;

&nbsp;&nbsp;&nbsp;&nbsp;PlayerInput&nbsp;\_input;

&nbsp;&nbsp;&nbsp;&nbsp;Player.ViewMode&nbsp;\_viewMode;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;Awake()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_player&nbsp;=&nbsp;GetComponent\<Player\>();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_input&nbsp;=&nbsp;GetComponent\<PlayerInput\>();

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;virtual&nbsp;void&nbsp;Start()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(InitialWeapon)&nbsp;EquipWeapon(InitialWeapon);

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;Update()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;switch&nbsp;(\_viewMode)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;Player.ViewMode.FIRST\_PERSON:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;일반&nbsp;공격

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(\_input.actions["NormalAttack"].ReadValue\<float\>()&nbsp;\>&nbsp;0

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&&&nbsp;equippedWeapon)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//1인칭일&nbsp;때

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//:&nbsp;카메라의&nbsp;마우스&nbsp;위치로&nbsp;검출.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ray&nbsp;ray&nbsp;=&nbsp;\_player.ViewCamera.ViewportPointToRay(new&nbsp;Vector3(0.5f,&nbsp;0.5f,&nbsp;0));

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AttackArgs&nbsp;args;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.Attacker&nbsp;=&nbsp;\_player;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.Origin&nbsp;=&nbsp;ray.origin;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.Direction&nbsp;=&nbsp;ray.direction;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;equippedWeapon.NormalAttack(args);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;Player.ViewMode.TOPDOWN:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;일반&nbsp;공격

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(\_input.actions["NormalAttack"].ReadValue\<float\>()&nbsp;\>&nbsp;0

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&&&nbsp;equippedWeapon)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//마우스&nbsp;포지션으로&nbsp;쏘기

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ray&nbsp;ray&nbsp;=&nbsp;\_player.ViewCamera.ScreenPointToRay(Mouse.current.position.ReadValue());

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Plane&nbsp;plane&nbsp;=&nbsp;new&nbsp;Plane(Vector3.up,&nbsp;Vector3.zero);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;distance;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(plane.Raycast(ray,&nbsp;out&nbsp;distance))

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;point&nbsp;=&nbsp;ray.GetPoint(distance);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;direction&nbsp;=&nbsp;point&nbsp;-&nbsp;ray.origin;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;direction.y&nbsp;=&nbsp;0;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AttackArgs&nbsp;args;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.Attacker&nbsp;=&nbsp;\_player;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.Origin&nbsp;=&nbsp;\_player.transform.position;&nbsp;&nbsp;&nbsp;//플레이어&nbsp;위치를&nbsp;시작점으로

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args.Direction&nbsp;=&nbsp;direction;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;equippedWeapon.NormalAttack(args);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;default:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;EquipWeapon(Weapon&nbsp;weapon)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(equippedWeapon)&nbsp;Destroy(equippedWeapon.gameObject);

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//weapon의&nbsp;타입에&nbsp;따라&nbsp;장착&nbsp;위치가&nbsp;다름.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;switch&nbsp;(weapon.Type)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;Weapon.WeaponType.Gun:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;equippedWeapon&nbsp;=&nbsp;Instantiate(weapon,&nbsp;GunHold.position,&nbsp;GunHold.rotation);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;equippedWeapon.gameObject.layer&nbsp;=&nbsp;GunHold.gameObject.layer;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;equippedWeapon.transform.parent&nbsp;=&nbsp;GunHold;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;default:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;OnViewModeChanged(Director.ViewModeChangedArgs&nbsp;args)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_viewMode&nbsp;=&nbsp;args.ViewMode;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;총구&nbsp;안&nbsp;보이게

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;struct&nbsp;AttackArgs

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;ICombatable&nbsp;Attacker;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;총의&nbsp;경우

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Vector3&nbsp;Origin;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Vector3&nbsp;Direction;

&nbsp;&nbsp;&nbsp;&nbsp;}

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

#### Weapon

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

&nbsp;

public&nbsp;abstract&nbsp;class&nbsp;Weapon&nbsp;:&nbsp;MonoBehaviour

{

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;enum&nbsp;WeaponType

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Gun

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;WeaponType&nbsp;Type;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;float&nbsp;Atk;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;abstract&nbsp;void&nbsp;NormalAttack(WeaponController.AttackArgs&nbsp;args);

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

#### Gun

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

 | 

using&nbsp;System.Collections;

using&nbsp;System.Collections.Generic;

using&nbsp;UnityEngine;

&nbsp;

public&nbsp;class&nbsp;Gun&nbsp;:&nbsp;Weapon

{

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Transform&nbsp;Muzzle;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;float&nbsp;ShootTerm;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;LayerMask&nbsp;Layermask;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;float&nbsp;Distance;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;\_remainShootTerm;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;Update()

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_remainShootTerm&nbsp;-=&nbsp;Time.deltaTime;

&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;override&nbsp;void&nbsp;NormalAttack(WeaponController.AttackArgs&nbsp;args)

&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(\_remainShootTerm&nbsp;\<=&nbsp;0)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ray&nbsp;ray&nbsp;=&nbsp;new&nbsp;Ray(args.Origin,&nbsp;args.Direction);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;RaycastHit&nbsp;hit;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(Physics.Raycast(ray,&nbsp;out&nbsp;hit,&nbsp;Distance,&nbsp;Layermask,&nbsp;QueryTriggerInteraction.Collide))

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector3&nbsp;point&nbsp;=&nbsp;hit.point;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Debug.DrawRay(ray.origin,&nbsp;point&nbsp;-&nbsp;ray.origin);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IDamagable&nbsp;damagable&nbsp;=&nbsp;hit.collider.GetComponent\<IDamagable\>();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//attacker의&nbsp;스탯과&nbsp;무기&nbsp;damage를&nbsp;계산

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;totalDamage&nbsp;=&nbsp;CombatSystem.CalculateInflictedDamage(args.Attacker,&nbsp;this,&nbsp;damagable);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(damagable&nbsp;!=&nbsp;null)&nbsp;damagable.TakeHit(totalDamage);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\_remainShootTerm&nbsp;=&nbsp;ShootTerm;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;}

}

&nbsp;

[Colored by Color Scripter](http://colorscripter.com/info#e)
 | [cs](http://colorscripter.com/info#e) |

  

  

#### 키입력과 적용을 나눈 이유

Update()는 프레임 단위로 실행되고, FixedUpdate()는 정해진 시간 주기로 실행된다.

Update()의 실행 주기는 매 프레임 달라진다.

따라서 Update()가 한 번 실행되는 동안 FixedUpdate()가 여러번 실행될 수 있고, 한 번도 실행되지 않을 수도 있다.

  

문제는, 화면 갱신과 키입력이 Update() 함수의 주기를 따르지만 물리의 적용은 FixedUpdate()에서 이루어진다는 것이다.

입력에 따른 움직임과 물리 적용을 정확하게 받으려면 FixedUpdate()에서 움직여야 한다.

  

만약 Update()가 한 번 실행되는 동안 FixedUpdate()가 여러번 실행된다면,

  

(이전 프레임) ---- (FixedUpdate) ---- (FixedUpdate) ---- (다음 프레임)

  

이런 형태가 될 것이다.

  

이전 프레임에서 받은 키입력이 아직 갱신되지 않았는데 FixedUpdate()이 두 번 실행되어, 한 번 입력된 값을 두 번 읽고 적용하게 된다. 즉, 프레임이 느려질 경우 공격키를 한 번 눌렀는데 여러번 나가진다거나 하는 문제가 발생할 수 있다.

  

반대로 프레임 갱신 주기가 빨라질 경우 FixedUpdate()가 어떤 프레임 사이에는 한 번도 실행되지 않는다면, 키를 눌렀는데 인식이 되지 않는 문제가 발생할 것이다. 움직임도 뚝뚝 끊기게 된다.

  

  

#### 실행영상

<object class="BLOG_video_class" contentid="60b92ede0b6a5b18" height="559" id="BLOG_video-60b92ede0b6a5b18" width="673"></object>
  

  

  

  

  

  
