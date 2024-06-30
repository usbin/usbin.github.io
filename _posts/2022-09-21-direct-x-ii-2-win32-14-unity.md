--- layout: post title: 'Direct X로 간단한 게임 만들기 - III. 게임엔진 : 카메라, Unity 빌드' date: '2022-09-21T07:00:00.004-07:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2023-01-24T22:49:48.541-08:00' thumbnail: https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhx0AhqvkCuOwd-x1XgGde5Gnq9bcvWe7FtXE9zXLp2y-ZGDD17Wp2oQExyZsqtV41cBNvl55eBTw3fWcIBn22YHeFfQBLqyQvI4jqL5shZWFmGbYLgusHuKDe4bO-odcQ6WzvJOmCTn7YVVxLM2aMqJtmzw0S59Ag2IV2zOSiVRl1gvJDz6qE-lG-t-g/s72-w640-c-h508/%EB%B0%A9%ED%96%A5%ED%82%A4%EB%A1%9C%20%EC%B9%B4%EB%A9%94%EB%9D%BC%20%EC%9D%B4%EB%8F%99.gif blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-4447498553209591951 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/09/direct-x-ii-2-win32-14-unity.html ---

    > 목차
    [1. 카메라](#user-content-1-카메라)
    [2. 카메라 클래스](#user-content-2-카메라-클래스)
    [3. Render 변경](#user-content-3-render-변경)
    [4. 추가기능(마우스 클릭지점으로 천천히 이동)](#user-content-4-추가기능마우스-클릭지점으로-천천히-이동)### 1. 카메라
    
    프로그램에 카메라를 추가했다.
    카메라는 실제로 화면을 그리는 데 사용되는 건 아니고 "카메라 좌표"라는 걸 저장하고 있다가 Render 할 때 실제 그려질 "렌더링 좌표"를 계산하는 역할을 한다.
    
    앞으로 모든 오브젝트와 컴포넌트 등등 화면에 그려질 모든 것들은 렌더링을 할 때마다 본인의 좌표를 카메라의 GetRenderPos를 호출해 렌더링 좌표로 변환한 후에 그곳에 그려줄 것이다.
    
    렌더링 좌표를 얻는 방법은 카메라의 원점에서부터 카메라가 떨어져있는지를 Vector2 값으로 구하고,
    받은 오브젝트의 좌표에서 카메라의 위치를 빼주면 된다.
    카메라의 원점은 처음 (0,0)부터 (x축 해상도, y축 해상도)까지 그려진 화면의 정중앙 좌표이므로 해상도/2 지점이 된다.
    즉, 렌더링 좌표 = 오브젝트 좌표 - (카메라 현재위치 - 카메라 원점) 으로 구할 수 있다.
    
    ### 2. 카메라 클래스
    
    ```c++
    //Camera.h
    #pragma once
    #include "global.h"
    class GObject;
    
    class Camera
    {
    	SINGLETON(Camera);
    
    private:
    	Vector2 look_pos_; //카메라가 보는 위치(중심 좌표)
    	GObject* look_target_;	//타깃이 있으면 따라감
    
    public:
    	void Update();
    	Vector2 GetRenderPos(Vector2 obj_pos);	//카메라 기준 좌표
    	Vector2 GetWorldPos(Vector2 obj_pos);	//전체 월드 기준 좌표
    	void set_look_pos(Vector2 look_pos) { look_pos_ = look_pos; };
    	Vector2 get_look_pos() { return look_pos_; };
    	void set_target(GObject* target) { look_target_ = target; };
    };
    
    
    ```
    
    ```c++
    //Camera.cpp
    #include "Camera.h"
    #include "GObject.h"
    #include "Core.h"
    Camera::Camera()
    	: look_target_(nullptr)
    	, look_pos_(Core::GetInstance()->get_resolution() / 2) {
    
    }
    Camera::~Camera() {
    
    }
    void Camera::Update()
    {
    	if (look_target_ != nullptr) {
    		look_pos_ = look_target_->get_pos();
    	}
    
    	Vector2 direction{};
    	if (KEY_HOLD(KEY::A)) {
    		direction.x = -1;
    	}
    	else if (KEY_HOLD(KEY::D)) {
    		direction.x = 1;
    	}
    	if (KEY_HOLD(KEY::W)) {
    		direction.y = -1;
    	}
    	else if (KEY_HOLD(KEY::S)) {
    		direction.y = 1;
    	}
    	look_pos_ += direction.Normalize()*DtF()*200.f;
    	
    }
    
    Vector2 Camera::GetRenderPos(Vector2 obj_pos)
    {
    	//카메라 원점(== 해상도/2 지점)
    	Vector2 center_pos = Core::GetInstance()->get_resolution() / 2;
    	//원점에서 카메라가 얼마나 이동했는가
    	Vector2 diff = look_pos_ - center_pos;
    	//카메라 이동량만큼 빼줌.
    	return Vector2{ obj_pos - diff};
    }
    
    Vector2 Camera::GetWorldPos(Vector2 obj_pos)
    {
    	//카메라 원점(== 해상도/2 지점)
    	Vector2 center_pos = Core::GetInstance()->get_resolution() / 2;
    	//원점에서 얼마나 이동했는가
    	Vector2 diff = look_pos_ - center_pos;
    	//카메라 이동량만큼 더해줌.
    	return Vector2{ obj_pos + diff};
    }
    
    ```
    
    WASD 키로 카메라를 이동하도록 했다.
    만약 타깃이 있다면 타깃의 위치를 카메라의 좌표로 매 Update마다 설정하도록 해서, 타깃을 따라가게 했다.
    
    그리고 Core의 메인루틴에도 Update 구문을 추가해둔다.
    
    ```c++
    //Core.cpp
    
    bool Core::Progress()
    {
    	//===============
    	//	선행작업
    	//===============
    	SyncResolution();
    	Time::GetInstance()->Update();
    	KeyManager::GetInstance()->Update();
    	Camera::GetInstance()->Update(); //여기 추가함
    
    	//===============
    	//	메인 루틴
    	//===============
    	SceneManager::GetInstance()->Update();
    
    	//===============
    	//	마무리 루틴
    	//===============
    	CollisionManager::GetInstance()->Update();
    
    	//===============
    	//	화면 렌더링
    	//===============
    	SceneManager::GetInstance()->ClearView(hdc_mem_);
    	SceneManager::GetInstance()->Render(hdc_mem_);
    	BitBlt(hdc_, 0, 0, pt_resolution_.x, pt_resolution_.y, hdc_mem_, 0, 0, SRCCOPY);
    
    
    	//===============
    	//	이벤트 지연 처리
    	//===============
    	EventManager::GetInstance()->Update();
    
    	return TRUE;
    }
    ```
    현재 Core 메인 루틴은 위와 같다.
    
    ### 3. Render 변경
    이제 모든 Render 함수에서 자신의 pos가 아니라 렌더링 pos를 사용해서 그리도록 변경해준다.
    Render 함수가 있는 곳은 GObject를 상속하는 클래스들과 Component들이다.
    현재는 Player, Monster, Missile, Collider, Animation 다섯가지 클래스에서 Render를 사용하고 있으므로 전부 변경해주...면 되는데 Player의 Render는 애니메이터가 담당하고 있으므로 Render 함수가 비어있다. 따라서 네 클래스만 변경해주면 된다.
    
    ```c++
    //Monster.cpp
    void Monster::Render(HDC hdc) {
    	SelectGdi _(hdc, BRUSH_TYPE::HOLLOW);
    	Vector2 render_pos = GetRenderPos(get_pos());
    
    	Rectangle(hdc
    		, static_cast<int>(render_pos.x - get_scale().x/2.)
    		, static_cast<int>(render_pos.y - get_scale().y / 2.)
    		, static_cast<int>(render_pos.x + get_scale().x / 2.)
    		, static_cast<int>(render_pos.y + get_scale().y / 2.));
    
    }
    
    ```
    
    ```c++
    //Missile.cpp
    void Missile::Render(HDC hdc)
    {
    	Vector2 render_pos = GetRenderPos(get_pos());
    	Ellipse(hdc
    		, static_cast<int>(render_pos.x - get_scale().x/2.)
    		, static_cast<int>(render_pos.y - get_scale().y / 2.)
    		, static_cast<int>(render_pos.x + get_scale().x / 2.)
    		, static_cast<int>(render_pos.y + get_scale().y / 2.));
    	
    }
    ```
    
    ```c++
    //Collider
    
    void Collider::Render(HDC hdc)
    {
    	//테두리 그리기
    	PEN_TYPE pen_type = PEN_TYPE::GREEN;
    	if (collision_count_) pen_type = PEN_TYPE::RED;
    	SelectGdi _(hdc, pen_type);
    	SelectGdi __(hdc, BRUSH_TYPE::HOLLOW);
    
    	Vector2 render_pos = GetRenderPos(final_pos_);
    	Rectangle(hdc
    		, static_cast<int>(render_pos.x - scale_.x / 2)
    		, static_cast<int>(render_pos.y - scale_.y / 2)
    		, static_cast<int>(render_pos.x + scale_.x / 2)
    		, static_cast<int>(render_pos.y + scale_.y / 2));
    }
    ```
    
    ```c++
    //Animation
    void Animation::Render(HDC hdc)
    {
    
    	//현재 프레임 그리기
    	const Vector2& pos = GetRenderPos(animator_->get_owner()->get_pos() + offset_);
    		
    	const AnimationFrame& frame = frames_[frame_index_];
    	TransparentBlt(
    		hdc //목적지 dc
    		, static_cast<int>(pos.x - frame.img_size.x / 2.) //left 좌표
    		, static_cast<int>(pos.y - frame.img_size.y / 2.) //top 좌표
    		, static_cast<int>(frame.img_size.x) //가로 길이
    		, static_cast<int>(frame.img_size.y) //세로 좌표
    		, texture_->get_hdc() //소스 dc(=비트맵이 선택된 메모리dc)
    		, static_cast<int>(frame.base_pos.x)	//비트맵의 left 좌표
    		, static_cast<int>(frame.base_pos.y)	//비트맵의 top 좌표
    		, static_cast<int>(frame.img_size.x)	//비트맵의 가로 길이
    		, static_cast<int>(frame.img_size.y)	//비트맵의 세로 길이
    		, RGB(255, 0, 255)); //무시할 픽셀 색상
    
    }
    ```
    
    ### 4. 추가기능(마우스 클릭지점으로 천천히 이동)
    
    마우스로 어느 지점을 클릭하면 그 지점으로 카메라가 천천히 이동하는 기능을 추가했다.
    이 기능을 구현하려면 마우스 좌표를 받아오는 기능이 필요하다.
    
    전에 구현한 KeyManager 클래스에서 포커스가 윈도우에 있을 땐 마우스 좌표도 계속해서 저장하도록 변경한다.
    이때 마우스 좌표는 월드 좌표여야 한다.
    GetCursorPos라는 윈도우 함수는 윈도우 화면상의 좌표를 주는데, 이걸 클라이언트 좌표(윈도우 좌표)로 변환하면 곧 렌더링 좌표가 된다.
    이걸 월드좌표로 한 번 더 변환해야 한다.
    
    ```c++
    //KeyManager.cpp
    bool KeyManager::Update()
    {
    	//프로그램이 포커싱됨
    	if (GetFocus() != nullptr) {
    		for (int i = 0; i < static_cast<int>(KEY::LAST); i++) {
    			//키 눌린 상태
    			if (GetAsyncKeyState(g_windows_keys[i]) & 0x8000) {
    				//생략...
    			}
    			//키 눌리지 않은 상태
    			else {
    				//...생략
    			}
    		}
            //========= 이부분에 추가 ================
    		POINT pt_mouse_pos;
    		GetCursorPos(&pt_mouse_pos);
    		ScreenToClient(Core::GetInstance()->get_main_hwnd(), &pt_mouse_pos);
    		
    		mouse_pos_ = GetWorldPos(Vector2{ pt_mouse_pos });
            //========================================
    	}
    	//프로그램이 포커싱되어 있지 않음 -> 자연스럽게 키가 떨어진 것처럼 바꿈
    	else {
    		for (int i = 0; i < static_cast<int>(KEY::LAST); i++) {
    			//...생략
    			
    		}
    	}
    
    
    	
    	
    	return TRUE;
    }
    ```
    그리고 이 mouse_pos_를 받아오는 함수를 만든 다음, inline 함수로도 만들어둔다.(자주 사용할 것 같으므로)
    
    ```c++
    //inline.h
    inline Vector2 GET_MOUSE_POS() {
    	return KeyManager::GetInstance()->GetMousePos();
    }
    ```
    
    이제 카메라의 변수로 남은시간과 목적지 벡터를 추가하고
    
    ```c++
    //Camera.h
    	Vector2 look_pos_dest_;	//천천히 움직여서 도착할 위치
    	float remain_second_;	//도착하기까지 몇 초 남았는지
    ```
    
    Update 함수에서 마우스 클릭이 들어온 순간 목적지와 남은 시간을 설정하도록 하고,
    
    카메라 상태에 따라 구문을 분리해준다.
    (1) 타깃이 설정돼있을 때
    (2) 목적지가 설정돼있을 때(남은 시간이 0보다 클 때)
    (3) 그 외엔 WASD로 움직이도록
    세 경우로 나눈다.
    
    이 중 (2)일 때엔 목적지와의 거리와 남은 시간을 계산해 한 프레임에 이동할 거리를 구하고 그만큼 이동시킨다.
    
    ```c++
    //Camera.cpp
    void Camera::Update()
    {
    	//타깃이 있을 때: 타깃 따라감
    	if (look_target_ != nullptr) {
    		look_pos_ = look_target_->get_pos();
    	}
    	//목적지가 지정되었을 때: 해당 목적지로 천천히 이동 
    	else if(remain_second_>0){
    		Vector2 diff = look_pos_dest_ - look_pos_;
    		// 1초에 가야하는 거리*DtF() = 1프레임에 가야하는 거리
    		// = 남은 거리/시간 * DtF()
    		look_pos_ += (diff / remain_second_) * DtF();
    
    		remain_second_ -= DtF();
    	}
    	//아무 상태도 아닐 때: WASD로 이동
    	else {
    		Vector2 direction{};
    		if (KEY_HOLD(KEY::A)) {
    			direction.x = -1;
    		}
    		else if (KEY_HOLD(KEY::D)) {
    			direction.x = 1;
    		}
    		if (KEY_HOLD(KEY::W)) {
    			direction.y = -1;
    		}
    		else if (KEY_HOLD(KEY::S)) {
    			direction.y = 1;
    		}
    		look_pos_ += direction.Normalize() * DtF() * 200.f;
    
    		
    	}
    	if (KEY_DOWN(KEY::LBUTTON)) {
    		MoveTo(GET_MOUSE_POS(), .5f);
    	}
    	
    	
    }
    
    ```
    
    **[WASD로 움직일 때]**

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhx0AhqvkCuOwd-x1XgGde5Gnq9bcvWe7FtXE9zXLp2y-ZGDD17Wp2oQExyZsqtV41cBNvl55eBTw3fWcIBn22YHeFfQBLqyQvI4jqL5shZWFmGbYLgusHuKDe4bO-odcQ6WzvJOmCTn7YVVxLM2aMqJtmzw0S59Ag2IV2zOSiVRl1gvJDz6qE-lG-t-g/w640-h508/%EB%B0%A9%ED%96%A5%ED%82%A4%EB%A1%9C%20%EC%B9%B4%EB%A9%94%EB%9D%BC%20%EC%9D%B4%EB%8F%99.gif)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhx0AhqvkCuOwd-x1XgGde5Gnq9bcvWe7FtXE9zXLp2y-ZGDD17Wp2oQExyZsqtV41cBNvl55eBTw3fWcIBn22YHeFfQBLqyQvI4jqL5shZWFmGbYLgusHuKDe4bO-odcQ6WzvJOmCTn7YVVxLM2aMqJtmzw0S59Ag2IV2zOSiVRl1gvJDz6qE-lG-t-g/s1016/%EB%B0%A9%ED%96%A5%ED%82%A4%EB%A1%9C%20%EC%B9%B4%EB%A9%94%EB%9D%BC%20%EC%9D%B4%EB%8F%99.gif)
  

    **[마우스 클릭으로 움직일 때]**

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgIV0TC7vr_hyW-_ZUcVEC6yFIjBFVLGitmIuEzugWJDo7ObvvTFQ70sDdHq0u2pczPwlwSO7X4SyYl5Gl5JPm8LiLlhKwyXuWez2oXesdCNqepnb28betC4iRlrDyDXvJa-OZEPsHfFZF81UpOQ3ISGox126JOIS-QtHe5W6mRmaIBIilHpWWntGekyQ/w640-h508/%EB%A7%88%EC%9A%B0%EC%8A%A4%EB%A1%9C%20%EC%B9%B4%EB%A9%94%EB%9D%BC%20%EC%9D%B4%EB%8F%99.gif)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgIV0TC7vr_hyW-_ZUcVEC6yFIjBFVLGitmIuEzugWJDo7ObvvTFQ70sDdHq0u2pczPwlwSO7X4SyYl5Gl5JPm8LiLlhKwyXuWez2oXesdCNqepnb28betC4iRlrDyDXvJa-OZEPsHfFZF81UpOQ3ISGox126JOIS-QtHe5W6mRmaIBIilHpWWntGekyQ/s1016/%EB%A7%88%EC%9A%B0%EC%8A%A4%EB%A1%9C%20%EC%B9%B4%EB%A9%94%EB%9D%BC%20%EC%9D%B4%EB%8F%99.gif)
  

    ### 5. Unity 빌드
    \* 게임엔진 Unity와 관련없음.
    
    파일이 많아지면서 vs 코드를 빌드할 때 너무 시간이 오래 걸리는데, 모든 파일을 링크하는 데 소요되는 시간 때문이라고 한다.
    한 파일에 모든 cpp를 몰아넣으면 cpp 하나에 여러 헤더들만 연결시켜주면 돼서 빨리 빌드할 수 있다고 함.
    이걸 Unity 빌드라고 한다.
    
    vs는 Unity 빌드 기능을 제공한다.
    
    프로젝트 속성-고급-고급옵션-모든 구성에서 Unity Build 사용을 예로 바꾸고, 디렉터리를 설정해준다.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjPZF4PscS3ZjpG7vKmWQoxqePGwTb2BMwXETg3iqMGDlLmz_CGYhRzsZKwlSs74TFyUQjbXTbdJyThBB1bXfoLcoDBordltXyUuSxMSo5eFfg1ea7RwvxB6eNX2qlvWm2yQ0ZdI5JMwvcaJY1FxTWYKuaJ7L6jjMcmLOHsbxAqOsqdXCZDpWi7aN7QhA=w640-h446)](https://blogger.googleusercontent.com/img/a/AVvXsEjPZF4PscS3ZjpG7vKmWQoxqePGwTb2BMwXETg3iqMGDlLmz_CGYhRzsZKwlSs74TFyUQjbXTbdJyThBB1bXfoLcoDBordltXyUuSxMSo5eFfg1ea7RwvxB6eNX2qlvWm2yQ0ZdI5JMwvcaJY1FxTWYKuaJ7L6jjMcmLOHsbxAqOsqdXCZDpWi7aN7QhA)
  

  

  

    대신 Unity 빌드를 사용하면 미리 컴파일된 헤더를 사용할 수 없다.
    미리 컴파일된 헤더도 컴파일 시간을 줄이기 위한 방책이었는데, 헤더로 줄이는 것보다 소스파일을 하나로 합치는 게 훨씬 빠르더라.
    pch.h는 미리 컴파일된 헤더에 보통 사용하는 이름이라 헷갈릴 것 같아 이름을 global.h로 바꿨다.
    귀찮긴 한데 모든 헤더가 global.h를 참조하게 하고, 기존의 pch.h를 참조하는 부분은 일일이 싹다 지웠다.
    
    프로젝트 속성-C/C++-모든 구성에서 미리컴파일된 헤더를 사용 안 함으로 설정한다.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgwL4HSiQGgRW35_7RAUYs2JSG8VdyrZtNPCilrv3CWvccsogApgDR9xf-PNHCDpWUAb_kngKTX2bc9UypPyl6eEpxuX9Crhs2-rbL2ryv8BYkAJX-KtcAiPyxpQ0OlbLa-7eM4e61oBhuTR72P3d-f269tkjhPEMBed_DYQsPUvaXPotFedzb38RoCeQ=w640-h442)](https://blogger.googleusercontent.com/img/a/AVvXsEgwL4HSiQGgRW35_7RAUYs2JSG8VdyrZtNPCilrv3CWvccsogApgDR9xf-PNHCDpWUAb_kngKTX2bc9UypPyl6eEpxuX9Crhs2-rbL2ryv8BYkAJX-KtcAiPyxpQ0OlbLa-7eM4e61oBhuTR72P3d-f269tkjhPEMBed_DYQsPUvaXPotFedzb38RoCeQ)
  

    전에는 컴파일 할 때 한 10~20초 걸렸는데 Unity 빌드로 바꾸니 컴파일 할 때 시간이 거의 2초 내로 줄었다.

