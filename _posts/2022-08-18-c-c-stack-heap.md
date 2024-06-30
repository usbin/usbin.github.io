--- layout: post title: "[C++] C++에서의 Stack과 heap에 저장되는 변수, 포인터" date: '2022-08-18T00:33:00.006-07:00' author: 가도 tags: - C-plusplus modified\_time: '2022-08-18T00:35:31.987-07:00' thumbnail: https://blogger.googleusercontent.com/img/a/AVvXsEhjolWJ7VkkgcJdV6coPjnJMxU9L1uYAT5TC5Lq8LsLxZnjXu7LddY-BtCcIikaBUa1xl1Bo8-yppVut5h38qY2DYoMdcrxRe-8RdJpCNuQIRKBYwXxw\_VTY4smYXM9xT2gFjPZ\_hUs75iebT9kQMlMQhd7wy2VNYBhTt7n4OD6hZ9Cc3KCzvNaUZ8l7A=s72-c blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-4875916983789468520 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/08/c-c-stack-heap.html ---

    모든 프로세스는 Code, Data, Stack, Heap로 나뉘는 메모리 공간을 갖는다.
    Code에는 프로그램의 소스코드가, Data에는 전역, 정적 변수 및 구조체, 배열이 들어간다.
    
    > 그럼 Stack과 Heap에는 뭐가 들어갈까?
    
    먼저 Stack에는 로컬 변수가 들어간다.
    
    ```c++
    #include <iostream>
    void func(){
      int b = 20;
    }
    int main(){
      func();
      std::cout<<b<<"\n"; //에러!
    }
    ```
    
    위의 코드에서 변수 b는 func() 내에서만 쓸 수 있다.
    main() 함수와 func() 함수에서 사용하는 변수들은 모두 Stack 영역에 들어가는데, 함수가 종료되면 Stack영역에서 해당 함수가 차지하는 공간은 반환되기 때문이다.
    
    정적 변수가 함수가 리턴되어도 사라지지 않고, 전역 변수를 어디에서든 접근할 수 있는 건 Stack 영역이 아니라 Data영역에 저장되기 때문이다.
    또한 정적, 전역 변수 말고도 함수 리턴과 함께 사라지지 않게 만드는 방법이 있다.
    
    동적으로 메모리를 할당받을 경우, 이 메모리는 Heap 영역에 들어가게 되고 함수가 리턴되어도 사라지지 않는다.
    
    ```c++
    #include <iostream>
    
    int* p;
    void func() {
    	int* a = new int(10);
    	p = a;
    }
    int main() {
    	func();
    	std::cout << *p;
        delete p; //new로 할당했으면 사용이 끝난 후엔 반드시 delete
    }
    ```
    출력은 다음과 같다.
    |10||
    |--|--|

    예전엔 malloc(size)로 메모리를 할당받고 free(p)로 해제했는데 요즘은 new와 delete를 사용하는 추세인 것 같다.
    new도 내부적으로 malloc을 통해 구현되어 있다고 하니 사실 편의성 때문에 new를 쓰는 거라고 봐도 무방할듯.
    malloc은 Heap의 메모리를 할당받을 뿐이라 프로그래머가 추가적으로 안에 뭔가 값을 넣어줘야 하는데, new의 경우 메모리 할당과 동시에 초기화가 가능해서 좀더 편하다.
    new를 사용하면 생성자가 실행되고 delete를 실행하면 소멸자가 실행된다.
    
    
    참고로 여기서 ```new int(10)```은 10이라는 정수를 할당한 int형 공간을 만들어 반환하는 코드다.
    10칸짜리 int형 공간이 필요할 땐 ```new int[10]```을 사용한다.
    
    
    # <포인터가 로컬 변수를 가리켰을 때의 문제점>
    
    여기서 주의할 점이 있는데, 아래처럼 로컬 변수를 포인터가 가리키게 해선 안 된다.
    
    ```c++
    #include <iostream>
    
    void func(int* p) {
    	int a = 10;
    	p = &a;
    }
    int main() {
    	int* p = NULL;
    	func(p);
    	std::cout << *p;
    }
    ```
    이 경우 func() 함수가 종료되면 a는 스택에서 사라진다.
    따라서 p는 없는 변수를 가리키게 되어 문제가 생긴다.
    
    
    근데 사실 이렇게 사용해도 당장 문제가 생기진 않는다. 계속 메모리를 사용하다가 a가 있었던 공간에 다른 값이 쓰여져야만 예기치 못한 결과가 나타나기 때문이다.
    
    당장에 문제가 나타나지 않는 이유는 스택공간을 사용하다가 해제할 때마다 굳이 0으로 초기화를 시켜주지 않기 때문이다.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhjolWJ7VkkgcJdV6coPjnJMxU9L1uYAT5TC5Lq8LsLxZnjXu7LddY-BtCcIikaBUa1xl1Bo8-yppVut5h38qY2DYoMdcrxRe-8RdJpCNuQIRKBYwXxw_VTY4smYXM9xT2gFjPZ_hUs75iebT9kQMlMQhd7wy2VNYBhTt7n4OD6hZ9Cc3KCzvNaUZ8l7A)](https://blogger.googleusercontent.com/img/a/AVvXsEhjolWJ7VkkgcJdV6coPjnJMxU9L1uYAT5TC5Lq8LsLxZnjXu7LddY-BtCcIikaBUa1xl1Bo8-yppVut5h38qY2DYoMdcrxRe-8RdJpCNuQIRKBYwXxw_VTY4smYXM9xT2gFjPZ_hUs75iebT9kQMlMQhd7wy2VNYBhTt7n4OD6hZ9Cc3KCzvNaUZ8l7A)
  

    위의 그림처럼, 스택은 새 함수가 실행되면 늘어나고 함수가 리턴되면 해당 함수의 공간을 반환하면서 줄어들고 하는데,
    사용하지 않는 공간이 된다고 0으로 초기화되진 않는다.
    
    그래서 변수 a가 로컬 변수라 함수의 끝과 함께 stack에서 사라졌지만 그 공간의 10이라는 정수는 언젠가 스택이 다시 올라와 덮어쓸 때까지 남아있는 것이다.
    
    아무튼 당장에 문제가 생기는 게 아니라서 실수로라도 이렇게 써버리면 나중에 문제를 찾기가 힘들어지니 주의해야한다.
    
    (휴지통에서 삭제한 파일도 바로 복구하면 복구가 가능한 것도 이때문)
    
    
    # <동적 메모리 할당의 위험성과 스마트 포인터>
    
    포인터는 C언어에서 여러 용도로 쓰일 수 있는데, 일반적으로 동적 메모리 할당과 함께 쓰이기 때문에 해제를 제때 해주지 않으면 메모리 누수가 생긴다는 위험성이 있다.
    
    자바나 C#에선 **가비지 컬렉션** 이 사용하지 않는 메모리를 알아서 해제해주지만 C나 C++는 그런 거 없다. 프로그래머가 일일이 해제해야 한다.
    근데 다행히 일일이 해제하지 않아도 **스마트 포인터** 라는 걸 사용하면 Heap의 어떤 메모리 공간을 가리키는 포인터의 개수가 0이 되었을 때마다 알아서 삭제해준다고 한다.
    
    **가비지 컬렉션** 은 일정 주기로 한 번에 메모리를 쭉 돌면서 삭제하기 때문에 자바로 만든 프로그램은 가끔씩 버벅임이 있다고 한다.
    **스마트 포인터** 를 사용하면 주기적으로 쭉 도는 게 아니라 메모리를 해제할 타임에만 해제하기 때문에 오버헤드가 분산되는 장점이 있다.
    
    그래서 다음 포스팅은 스마트 포인터를 다뤄보려 한다.

