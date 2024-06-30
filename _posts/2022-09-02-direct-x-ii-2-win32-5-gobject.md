--- layout: post title: 'Direct X로 간단한 게임 만들기 - III. 게임엔진 : GObject, 더블 버퍼링' date: '2022-09-02T04:57:00.008-07:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2023-01-24T22:48:40.025-08:00' thumbnail: https://blogger.googleusercontent.com/img/a/AVvXsEjpswLVDE618NL4cWS-xKfTOIH-k97VhFmlN4eOl5LQ6JeFXD2uDPokPBhk3cyMcDx3G4pFZRdTXnwIWBN1cWM3tz\_B4TBYXTIcXKNPCg66uMS1DjdOZwhUeWNWqd\_MkqtYjShn3\_AExHQefi\_wCgDCMOPhOnaDIiUO0H1qN3bTMQ7vgA4EEtnE9MnRBQ=s72-w640-c-h400 blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-4097578509137997412 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/09/direct-x-ii-2-win32-5-gobject.html ---

    > 목차
    [1. 더블 버퍼링이란](#user-content-1-더블-버퍼링이란)
    [2. 화면에 네모를 그리는 프로그램(더블버퍼링 전)](#user-content-2-화면에-네모를-그리는-프로그램더블버퍼링-전)
    [3. 더블 버퍼링 적용하기](#user-content-3-더블-버퍼링-적용하기)
    [4. 화면에 네모를 그리는 프로그램(더블버퍼링 적용 후)](#user-content-4-화면에-네모를-그리는-프로그램더블버퍼링-적용-후)### 1. 더블 버퍼링이란
    
    Win32 프로그램에서 화면에 무언가를 그릴 때의 과정은 다음과 같다.
    
    ```c++
    //(1) DC를 생성한다.
    while(TRUE){
    	//(2) DC에 그리기 함수를 사용한다
    }
    //(3) DC를 릴리즈한다.
    ```
    
    (2)번 과정이 바로 **렌더링** 이다.
    
    렌더링은 이전 포스팅에서도 말했듯이 화면의 모든 그래픽 요소를 가장 밑에 깔리는 것부터 가장 위의 것까지 순서대로 그리는 걸 말한다.
    
    문제는 렌더링이 이루어지는동안 화면을 쳐다보고 있는 우리 눈에 이 과정이 띄엄띄엄 목격된다는 것에 있다.
    
    가령, 텅 빈 하얀 방에 테이블이 있고 그 위에 사과가 놓여져있다고 한다면, 렌더링하는 과정은
    - 하얀 배경색을 꽉 채워 칠한다.
    - 테이블을 그린다.
    - 사과를 그린다.
    이 과정을 매 프레임 반복하게 된다.
    
    그런데 우리 눈의 동체시력으로 따라갈 수 있는 속도는 컴퓨터의 처리속도보다 느리기 때문에, 순간순간을 보게 된다.
    그래서 실제로 저걸 눈으로 쳐다보고 있다면 어떤 순간에는 배경만 있고, 어떤 순간에는 배경과 테이블이 있고, 어떤 순간에는 배경과 테이블과 사과가 있을 것이다.
    따라서 우리 눈에는 화면의 모든 그림이 미친듯이 깜빡이는 것처럼 보인다.
    
    예시 프로그램을 하나 만들어보자.
    
    ### 2. 화면에 네모를 그리는 프로그램(더블버퍼링 전)
    
    일단 네모를 그리는 프로그램을 만들 건데, 마우스 커서를 클릭한 순간부터 네모를 그리기 시작해서 놓으면 화면에 그려지게 할 것이다.
    
    (1) 마우스 커서가 눌린 순간 마우스 위치를 시작 위치로 저장
    (2) 마우스 커서가 눌린 상태로 움직이는 동안 시작위치-현재 위치로 네모를 그림
    (3) 마우스 커서가 놓이는 순간 시작 위치-현재 위치를 크기로 갖는 오브젝트를 생성하고 리스트에 추가
    (4) 리스트를 순회하며 모든 오브젝트의 크기와 위치 정보를 바탕으로 화면에 순서대로 그림
    
    위와 같은 로직으로 동작하게 만들 건데, 먼저 화면에 보이는 모든 오브젝트를 담을 GObject 클래스를 만들었다.
    
    ```c++
    //GObject.h
    #pragma once
    //게임 안에 존재하는, 그려져야 하는 모든 오브젝트
    //매 렌더링에서 그려줌.
    class GObject
    {
    private:
    	static unsigned int id_counter_;
    	unsigned int id_;
    	RECT rect_;
    	SHORT priority_ = 0; //렌더링 우선순위. 클수록 나중에 그려짐(set의 뒤쪽에 위치)
    public:
    	GObject(RECT rect);
    	~GObject();
    	SHORT priority() { return priority_; };
    	unsigned long long id() { return id_; };
    
    	bool operator< (const GObject& o) const {
    		if (this->priority_ != o.priority_) return this->priority_ < o.priority_;
    		else return this->id_ < o.id_;
    	};
    	bool operator<= (const GObject& o) const {
    		if (this->priority_ != o.priority_) return this->priority_ < o.priority_;
    		else return this->id_ <= o.id_;
    	};
    	bool operator== (const GObject& o) const {
    		return this->id_ == o.id_;
    	}
    	const RECT rect() { return rect_; };
    };
    
    struct GObjectPtCompare {
    	const bool operator()(const GObject* o1, const GObject* o2) const {
    		return (*o1) < (*o2);
    	}
    };
    ```
    id_counter와 id_는 오브젝트가 새로 생성될 때마다 id값을 갖기 위해 존재한다.
    GObjectPtCompare 함수는 set에 GObject의 포인터가 들어가는데, set은 원소의 삽입 삭제 때마다 자동으로 정렬이 일어난다.
    근데 포인터간의 대소비교가 아니라 GObject의 우선순위, 그리고 만들어진 순서에 따라 정렬되도록 하기 위해
    GObject 자체의 연산자를 오버라이딩해주고 set의 Compare함수를 커스텀해서 넣어줬다.
    
    
    ```c++
    //GObject.cpp
    #include "pch.h"
    #include "GObject.h"
    #include "Core.h"
    
    unsigned int GObject::id_counter_ = 1;
    GObject::GObject(RECT rect) 
    	: rect_(rect)
    	, id_(id_counter_++){
    	Core::GetInstance()->OnCreateGObject(this);
    
    }
    GObject::~GObject(){
    	Core::GetInstance()->OnDestroyGObject(this);
    }
    
    ```
    
    생성될 땐 반드시 RECT로 초기화하도록 했다.
    그리고 생성될 땐 Core의 OnCreateGObject를, 소멸될 땐 OnDestroyGObject를 호출한다.
    
    이 두 함수는 Core 인스턴스의 gobjects_라는 멤버변수에 해당 GObject를 넣고 빼는 동작을 한다.
    
    ```c++
    //GObject.cpp
    bool Core::OnCreateGObject(GObject* object)
    {
    	gobjects_.insert(object);
    	return TRUE;
    }
    
    bool Core::OnDestroyGObject(GObject* object)
    {
    	gobjects_.erase(object);
    	return TRUE;
    }
    ```
    
    Core 클래스에 선언된 gobjects_ 변수.
    ```c++
    std::set<GObject*, GObjectPtCompare> gobjects_;//모든 GObject를 담는 컨테이너<gobject gobjectptcompare="">
    ```
    
    이제 위의 로직을 따라서 Core의 Render 함수에 마우스로 네모를 그리는 코드를 작성했다.
    
    ```c++
    
    RECT g_rect;
    bool prev_pressed = false;
    
    bool Core::Render()
    {
    	Clear();
    	DrawMouse();
    
    	return TRUE;
    }
    
    void Core::DrawMouse()
    {
    	for (auto it : gobjects_) {
    		RECT rect = it-&gt;rect();
    		Rectangle(h_dc_, rect.left, rect.top, rect.right, rect.bottom);
    	}
    	SHORT l_button_state = GetAsyncKeyState(VK_LBUTTON);
    	POINT cur_pos;
    	GetCursorPos(&amp;cur_pos);
    	ScreenToClient(h_wnd_, &amp;cur_pos);
    	RECT client_rect;
    	GetClientRect(h_wnd_, &amp;client_rect);
        bool is_pressed = l_button_state &amp; 0x8000;
    	if (GetFocus() == h_wnd_ //현재 윈도우에 포커스가 있을 때
    		&amp;&amp; PtInRect(&amp;client_rect, cur_pos) //마우스가 작업영역에 있을 때
    		&amp;&amp; ( is_pressed || prev_pressed)) //마우스 좌클릭이 눌렸을 때 혹은 떼어졌을 때
    	{
    		bool begin_drawing = is_pressed &amp;&amp; !prev_pressed;
    		bool is_drawing = is_pressed &amp;&amp; prev_pressed;
    		bool end_drawing = !is_pressed &amp;&amp; prev_pressed;
    		if (begin_drawing) {
    			g_rect.left = cur_pos.x;
    			g_rect.top = cur_pos.y;
    		}
    		if (is_drawing) {
    			g_rect.right = cur_pos.x;
    			g_rect.bottom = cur_pos.y;
    			Rectangle(h_dc_, g_rect.left, g_rect.top, g_rect.right, g_rect.bottom);
    
    		}
    		if (end_drawing) {
    			new GObject{ g_rect };
    		}
    
    		prev_pressed = l_button_state &amp; 0x8000;
    	}
    }
    ```
    Clear() 함수는 그냥 화면을 흰색으로 채우는 함수다.
    
    ```c++
    void Clear() { Rectangle(h_mem_dc_, -1, -1, pt_resolution_.x + 1, pt_resolution_.y + 1); };
    ```
    
    윈도우 메시지는 키가 눌림, 눌린 상태임, 키가 올라옴, 이 세 가지의 상태를 구분해서 메시지로 줬지만
    GetAsyncKeyState 함수는 그냥 키가 눌렸는지 눌리지 않았는지만 알려준다.
    정확히는 0x8000과 0x1 두 비트로 알려주는데 전자는 단순히 키 입력이 들어왔는지, 후자는 직전 프레임 이후~현재 프레임 사이에 키 입력이 들어왔었는지를 알려준다.
    </gobject>

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjpswLVDE618NL4cWS-xKfTOIH-k97VhFmlN4eOl5LQ6JeFXD2uDPokPBhk3cyMcDx3G4pFZRdTXnwIWBN1cWM3tz_B4TBYXTIcXKNPCg66uMS1DjdOZwhUeWNWqd_MkqtYjShn3_AExHQefi_wCgDCMOPhOnaDIiUO0H1qN3bTMQ7vgA4EEtnE9MnRBQ=w640-h400)](https://blogger.googleusercontent.com/img/a/AVvXsEjpswLVDE618NL4cWS-xKfTOIH-k97VhFmlN4eOl5LQ6JeFXD2uDPokPBhk3cyMcDx3G4pFZRdTXnwIWBN1cWM3tz_B4TBYXTIcXKNPCg66uMS1DjdOZwhUeWNWqd_MkqtYjShn3_AExHQefi_wCgDCMOPhOnaDIiUO0H1qN3bTMQ7vgA4EEtnE9MnRBQ)
  

    이 부분은 나중에 키 매니저 클래스를 따로 만들어서 정리해야 하는데 일단은 임시로 이렇게 구현했다.
    키가 이전 프레임에서 눌리지 않은 상태였는데 현재 프레임에서 눌린 상태라면 "키다운됨"
    키가 이전 프레임에서 눌린 상태였고 현재 프레임에서도 눌린 상태라면 "키다운 상태"
    키가 이전 프레임에서 눌린 상태였고 현재 프레임에서 눌리지 않은 상태라면 "키업됨"
    이렇게 상태를 구분했다.
    
    <동작 모습>

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg4u76K8EM5FqodoJNIsh8x8l1tY6D0NygWUNNcxczExv_6cJTFwHIak4f426hVyAIorCXHsghNiKMQ43y3ERhX6wdmGc5eWURW1cmpTbHerFkFQom2zpGLUg9rg0bJ2XQFgUAWE-PsIhDLI16WRo5EP5C9K9n5xljPtjRjU_zOjyAwTjoGpv-oaZek4Q/w640-h466/%EB%8D%94%EB%B8%94%EB%B2%84%ED%8D%BC%EB%A7%81%EC%A0%81%EC%9A%A9%EC%A0%84.gif)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg4u76K8EM5FqodoJNIsh8x8l1tY6D0NygWUNNcxczExv_6cJTFwHIak4f426hVyAIorCXHsghNiKMQ43y3ERhX6wdmGc5eWURW1cmpTbHerFkFQom2zpGLUg9rg0bJ2XQFgUAWE-PsIhDLI16WRo5EP5C9K9n5xljPtjRjU_zOjyAwTjoGpv-oaZek4Q/s626/%EB%8D%94%EB%B8%94%EB%B2%84%ED%8D%BC%EB%A7%81%EC%A0%81%EC%9A%A9%EC%A0%84.gif)
  

    ### 3. 더블 버퍼링 적용하기
    무진장 깜빡거리는 걸 확인할 수 있다.
    이걸 해결하기 위해 더블 버퍼링을 적용했다.
    
    - 메모리DC와 비트맵을 멤버변수로 선언하고 Init함수에서 초기화한다.
    - 네모를 메모리DC에 그리도록 변경한다
    - Render 함수가 끝날 때마다 메모리DC의 비트맵을 윈도우DC로 고속복사한다.
    
    여기서 윈도우DC는 우리 눈에 보이는 저 깜빡이는 화면이고, 메모리DC는 눈에 보이지 않는 메모리 공간에 존재하는 화면이다.
    비트맵은 화면을 구성하는 도화지 같은 건데, (0, 0)부터 해상도 크기까지 넣어주면 된다. 눈에 보이는 크기보다 작게 비트맵을 넣어버리면 비트맵 밖 영역에는 그림이 그려지지 않는다.
    따라서 윈도우의 해상도를 저장할 POINT 멤버변수를 Core 클래스에 추가했다.
    
    그니까 눈에 보이지 않는 화면에 차곡차곡 그려서 완성한 뒤 눈에 보이는 화면에 고속복사해오면 우린 완성된 화면만을 깔끔하게 볼 수 있다.
    고속복사 함수는 win32에서 기본적으로 제공한다.
    
    ```c++
    //Core.h
    class Core{
    //... 생략...
    	POINT pt_resolution_;
    	HDC h_dc_;
    	HDC h_mem_dc_;
    //...생략...
    }
    ```
    
    
    Core의 Init() 함수와 소멸자 함수
    
    ```c++
    //Core.cpp
    int Core::Init(HWND h_wnd, int width, int height) {
    	
    	h_wnd_ = h_wnd;
    	h_dc_ = GetDC(h_wnd_);
    	RECT rect{ 0, 0, width, height };
    	pt_resolution_ = POINT{ width, height };
    	if (!AdjustWindowRect(&rect, WS_OVERLAPPEDWINDOW, TRUE)) {
    		return E_FAIL;
    	}
    	if (!SetWindowPos(h_wnd, nullptr, 100, 100, rect.right - rect.left, rect.bottom - rect.top, 0)) {
    		return E_FAIL;
    	}
    
    	
    	h_bitmap_ = CreateCompatibleBitmap(h_dc_, pt_resolution_.x, pt_resolution_.y);
    	h_mem_dc_ = CreateCompatibleDC(h_dc_);
    	HBITMAP org_bitmap = (HBITMAP) SelectObject(h_mem_dc_, h_bitmap_);
    	DeleteObject(org_bitmap);
    	Time::GetInstance()->Init(h_wnd);
    	return S_OK;
    }
    Core::~Core() {
    	ReleaseDC(h_wnd_, h_dc_);
    	DeleteDC(h_mem_dc_);
    	DeleteObject(h_bitmap_);
    }
    ```
    DrawMouse() 함수에서도 h_dc_에 그리는 부분을 전부 h_mem_dc_에 그리는 것으로 변경하고,
    Render() 마지막에 고속복사 코드를 추가해준다.
    
    
    ```c++
    //Core.cpp
    bool Core::Render()
    {
    	Clear();
    	DrawMouse();
    	BitBlt(h_dc_, 0, 0, pt_resolution_.x, pt_resolution_.y, h_mem_dc_, 0, 0, SRCCOPY);
    	
    	return TRUE;
    }
    
    
    void Core::DrawMouse()
    {
    	for (auto it : gobjects_) {
    		RECT rect = it->rect();
    		Rectangle(h_mem_dc_, rect.left, rect.top, rect.right, rect.bottom);
    	}
    	SHORT l_button_state = GetAsyncKeyState(VK_LBUTTON);
    	POINT cur_pos;
    	GetCursorPos(&cur_pos);
    	ScreenToClient(h_wnd_, &cur_pos);
    	RECT client_rect;
    	GetClientRect(h_wnd_, &client_rect);
    
    
    	bool is_pressed = l_button_state & 0x8000;
    	if (GetFocus() == h_wnd_ //현재 윈도우에 포커스가 있을 때
    		&& PtInRect(&client_rect, cur_pos) //마우스가 작업영역에 있을 때
    		&& (is_pressed || prev_pressed)) //마우스 좌클릭이 눌렸을 때 혹은 떼어졌을 때
    	{
    		bool begin_drawing = is_pressed && !prev_pressed;
    		bool is_drawing = is_pressed && prev_pressed;
    		bool end_drawing = !is_pressed && prev_pressed;
    		if (begin_drawing) {
    			g_rect.left = cur_pos.x;
    			g_rect.top = cur_pos.y;
    		}
    		if (is_drawing) {
    			g_rect.right = cur_pos.x;
    			g_rect.bottom = cur_pos.y;
    			Rectangle(h_mem_dc_, g_rect.left, g_rect.top, g_rect.right, g_rect.bottom);
    
    		}
    		if (end_drawing) {
    			new GObject{ g_rect };
    		}
    
    		prev_pressed = l_button_state & 0x8000;
    	}
    }
    ```
    
    ### 4. 화면에 네모를 그리는 프로그램(더블버퍼링 적용 후)

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiqEFBIOtCSBgFRHSpH5Bfn5eOUfx0WnD5p5oNXyGVPfYX47eC1rmmfiODij3r-YUCgWunkwRaVRtgNuem72BZT80K3cri1sfdKDZhUm7dvXGnzg9lQCA6jcAA8mtc6ibniUhYmkxyHUVIpMQXflzud7Avit1FgUTOzuoGrCDfuRc2etzzg8OzkVWrN5Q/w640-h466/%EB%8D%94%EB%B8%94%EB%B2%84%ED%8D%BC%EB%A7%81%EC%A0%81%EC%9A%A9%ED%9B%84.gif)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiqEFBIOtCSBgFRHSpH5Bfn5eOUfx0WnD5p5oNXyGVPfYX47eC1rmmfiODij3r-YUCgWunkwRaVRtgNuem72BZT80K3cri1sfdKDZhUm7dvXGnzg9lQCA6jcAA8mtc6ibniUhYmkxyHUVIpMQXflzud7Avit1FgUTOzuoGrCDfuRc2etzzg8OzkVWrN5Q/s626/%EB%8D%94%EB%B8%94%EB%B2%84%ED%8D%BC%EB%A7%81%EC%A0%81%EC%9A%A9%ED%9B%84.gif)
  

    깜빡임이 사라진 것을 확인할 수 있다.
    
    
    오늘 하면서 키입력의 키다운, 키다운된 상태, 키업을 구분하는 게 너무 코드가 번잡하단 생각이 들었다.
    이 부분을 클래스로 빼서 깔끔하게 정리해볼 예정이다.
    
    그리고 FPS 출력코드를 없애지 않아서 FPS를 확인 가능한데, 화면에 네모를 그리면 그릴수록 FPS가 뚝뚝 떨어진다.
    이건 CPU로 처리하기 때문이라서 GPU 처리로 변경하면 개선된다고 한다. 그게 Direct X가 들어가는 부분인듯.

