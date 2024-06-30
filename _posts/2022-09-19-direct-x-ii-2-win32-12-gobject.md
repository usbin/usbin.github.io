--- layout: post title: 'Direct X로 간단한 게임 만들기 - III. 게임엔진 : GObject 충돌 함수 구현, 프레임 동기화(지연처리)' date: '2022-09-19T04:18:00.009-07:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2023-01-24T22:49:36.032-08:00' thumbnail: https://blogger.googleusercontent.com/img/a/AVvXsEhmyH6bUT0QcaXFRz2rLRARWknUPG-jmFihrD8cct15XrJSmHOfNTbfP3EKmtfLKTTwV6rZiBpwCi\_YLBZc4ZAEjV-lQZlsmspR0ofDkrfMEVyGXzPbrc6KpLLVab0qCSsDN9SyYoMrlLwOyCDUky2MSqlbkqZB9n2JwVWPdEyBZRinRI7-CYU6JpnnGA=s72-w640-c-h400 blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-6811995844312705186 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/09/direct-x-ii-2-win32-12-gobject.html ---

    > 목차
    [1. GObject 충돌 함수 구현](#user-content-1-gobject-충돌-함수-구현)
    [2. 프레임 동기화 I](#user-content-2-프레임-동기화-i)
    [3. 이벤트 매니저](#user-content-3-이벤트-매니저)
    [4. 디버깅 중 DT가 무한정 늘어나는 문제](#user-content-4-디버깅-중-dt가-무한정-늘어나는-문제)
    [5. 프레임 동기화 II](#user-content-5-프레임-동기화-ii)### 1. GObject 충돌 함수 구현
    
    충돌함수가 Collider에 구현돼있는데, 여러 오브젝트가 각각 다른 충돌 처리를 구현한다면 또 Collider를 상속해서 PlayerCollider, MonsterCollider 등등으로 나눠줘야한다.
    이건 너무 번거롭고 프로젝트만 복잡해지기 때문에 GObject 자체에 충돌 함수를 가상함수로 선언해놓고 각 오브젝트가 알아서 구현하게 했다.
    그리고 Collider의 충돌 함수는 owner의 충돌 함수를 실행하도록 한다.
    
    ```c++
    //Collider.cpp
    void Collider::OnCollisionEnter(const Collider& collider)
    {
    	collision_count_++;
    	owner_->OnCollisionEnter(collider);
    }
    
    void Collider::OnCollisionStay(const Collider& collider)
    {
    	owner_->OnCollisionStay(collider);
    }
    
    void Collider::OnCollisionExit(const Collider& collider)
    {
    	collision_count_--;
    	owner_->OnCollisionExit(collider);
    }
    ```
    ```c++
    //GObject.h
    	virtual void OnCollisionEnter(const Collider& collider);
    	virtual void OnCollisionStay(const Collider& collider);
    	virtual void OnCollisionExit(const Collider& collider);
    ```
    이제 GObject를 상속한 오브젝트들이 각각 충돌함수를 구현하면 해당 함수가 실행된다.
    
    
    ### 2. 프레임 동기화 I
    
    **[예상되는 문제들]**
    \- 오브젝트 삭제 오브젝트를 가리키고 있는 포인터는 해당 오브젝트가 삭제되면 예상치 못한 문제가 발생함.
    \- 충돌 중에 삭제되면 Exit가 영영 실행되지 않음.
    \- 업데이트() -> 씬 체인지 -> 렌더() 시 씬 체인지 전후 씬의 오브젝트 리스트가 변경되며 생길 수 있는 문제들...
    
    
    **[프레임 동기화]**
    :지연처리 -> 해당 프레임에서 발생하자마자 바로 적용해선 안 될 사건들을 모든 처리가 끝난 후 적용해서, 다음 프레임에서 모두에게 일괄 반영되도록 함.
    
    **[프레임 동기화해야 할 일]**
    \- 오브젝트 생성, 삭제
    \- 씬 변경
    
    이 세가지 작업을 enum class로 선언해둔다.
    
    ```c++
    enum class EVENT_TYPE {
    	CREATE_OBJECT,
    	DELETE_OBJECT,
    	CHANGE_SCENE,
    	END
    };
    ```
    
    
    ### 3. 이벤트 매니저
    
    메인 루틴에 이벤트 지연 처리를 추가한다.
    
    (1) 이벤트 구조체와 이벤트 매니저 클래스
    이벤트 종류와 파라미터 2개를 갖는 구조체를 EventManager와 같은 파일에 선언한다.
    
    ```c++
    //EventManager.h
    #pragma once
    
    struct Event {
    	EVENT_TYPE type;
    	DWORD_PTR param1;
    	DWORD_PTR param2;
    };
    
    
    class EventManager
    {
    	SINGLETON(EventManager);
    private:
    	std::vector<Event> events_;
    	std::vector<GObject*> dead_objects_;
    	void ExecuteEvent(Event _event);
    public:
    	void Update();
    
    	void AddEvent(Event _event);
    };
    
    ```
    이벤트 매니저는 이벤트를 추가하고 실행하는 기능을 갖는다.
    그리고 메인 루틴의 가장 마지막에 실행될 Update 함수를 갖는다.
    
    \* DWORD와 DWORD_PTR의 차이점: DWORD는 고정 4바이트고, DWORD_PTR은 32비트 환경에선 4바이트, 64비트 환경에선 8바이트다.
    	파라미터로 들어갈 값은 포인터가 될 수도 있으므로 그냥 DWORD가 아니라 DWORD_PTR로 선언한다.
        
    
    이때 오브젝트를 생성, 삭제할 때 간편하게 하기 위해 전역함수를 정의해둔다.
    function.h와 function.cpp로 구성된 전역함수를 만들고,
    
    ```c++
    //funcion.h
    #pragma once
    
    class GObject;
    
    void CreateGObject(GObject* object, GROUP_TYPE type);
    void DeleteGObject(GObject* object, GROUP_TYPE type);
    
    
    //function.cpp
    #include "pch.h"
    #include "function.h"
    #include "GObject.h"
    #include "EventManager.h"
    
    void CreateGObject(GObject* object, GROUP_TYPE type) {
    	Event eve = {};
    	eve.type = EVENT_TYPE::CREATE_OBJECT;
    	eve.param1 = (DWORD_PTR)object;
    	eve.param2 = (DWORD_PTR)type;
    
    	EventManager::GetInstance()->AddEvent(eve);
    }
    
    void DeleteGObject(GObject* object, GROUP_TYPE type) {
    	Event eve = {};
    	eve.type = EVENT_TYPE::DELETE_OBJECT;
    	eve.param1 = (DWORD_PTR)object;
    	eve.param2 = (DWORD_PTR)type;
    
    	EventManager::GetInstance()->AddEvent(eve);
    }
    ```
    
    생성, 삭제 작업을 작성해둔다.
    
    EventManager의 AddEvent 코드:
    ```c++
    void EventManager::AddEvent(Event _event)
    {
    	events_.push_back(_event);
    }
    ```
    
    (2) 이벤트 매니저 클래스
    	- 다음 프레임에 적용할 이벤트 리스트를 벡터로 가짐.
        - 생성 이벤트: 오브젝트를 현재 씬에 추가
        - 삭제 이벤트: 오브젝트를 데드 상태로 변경 -> 다음 프레임: 데드 상태인 오브젝트를 삭제
    
    ```c++
    //EventManager.cpp
    void EventManager::Update()
    {
    	//삭제 작업2: 데드 오브젝트 삭제하기
    	for (GObject* p_obj : dead_objects_) {
    		delete p_obj;
    	}
    	dead_objects_.clear();
    
    
    
    	//이벤트 일괄 실행
    	for (const Event& eve : events_) {
    		ExecuteEvent(eve);
    	}
    	events_.clear();
    }
    
    void EventManager::ExecuteEvent(Event _event)
    {
    	//이벤트 타입에 따라서
    	switch (_event.type) {
    	case EVENT_TYPE::CREATE_OBJECT:
    	{
    		//param1: 오브젝트 포인터
    		//param2: 오브젝트 타입
    		//생성 작업: 현재 씬에 추가
    		SceneManager::GetInstance()->get_current_scene()->AddGObject((GObject*)_event.param1, (GROUP_TYPE)_event.param2);
    	}
    	break;
    	case EVENT_TYPE::DELETE_OBJECT:
    	{
    		//param1: 오브젝트 포인터
    		//param2: 오브젝트 타입
    		//삭제 작업1: 데드 상태로 만들기
    		GObject* p_obj = (GObject*)_event.param1;
    		p_obj->SetDead();
    		dead_objects_.push_back(p_obj);
    
    	}
    	break;
    	case EVENT_TYPE::CHANGE_SCENE:
    	{
    
    	}
    	break;
    	}
    }
    
    ```
    이벤트 타입에 따라 switch문으로 처리를 나눈다.
    
    (3) 오브젝트 생성 이벤트
    오브젝트 생성은 이벤트가 지연 처리되는 시점에 바로 생성해서 추가한다.
     
    (4) 오브젝트 삭제 이벤트
    
    > 저번에 오브젝트 삭제를 지연처리하도록 했는데, 그것만으론 제대로 처리가 안 된 것 같다.
    단순히 씬 안에서 Update가 끝난 후에 삭제 큐에 넣어져있는 것을 일괄 삭제하도록 했는데, 이번에 충돌 시스템을 추가하면서 충돌 처리로 인한 문제가 또 생겨버린다.
    그래서 오브젝트 삭제를 3단계로 나눠 삭제 요청을 받은 프레임의 가장 마지막 작업으로 "삭제예정" 상태로 만들고, 다음 프레임에서 "삭제예정"오브젝트를 참조하는 곳에서 각자 예외상황을 처리한 후, 그 프레임의 마지막 작업으로 실제 삭제를 진행한다.
    
    
    이벤트 삭제는 이벤트가 지연 처리되는 시점(EventManager의 ExecuteEvent 함수)에선 일단 오브젝트를 "Dead"상태로 만들고 dead_objects 벡터에 넣는다.
    그리고 EventManager의 Update 함수에선 dead_objects 벡터의 모든 오브젝트를 실제로 소멸시킨다.
    
    GObject에 is_dead 변수를 추가해주고, 마음대로 설정할 수 없도록 set 함수는 private, get 함수는 public으로 구현한다.
    그리고 EventManager만 접근할 수 있게 friend 선언을 해준다.
    
        
    (5) Core의 마지막 처리로 이벤트 지연 처리
    	- 이벤트 매니저가 이벤트 일괄 실행
    
    
    **[처리 로직]**
    메인 루틴은 메인 Update -> 이벤트 매니저 Update 순서로 실행된다.
    ==== 1번 프레임 ====
    (1)메인 Update 안에서 삭제 이벤트가 요청되고, 이렇게 쌓인 요청들은 이벤트 매니저 Update 함수에서 실행된다.
    (2)Execute 함수가 처음 실행될 땐 벡터가 비어있으므로 건너뛰고, 쌓인 삭제 이벤트를 일괄 실행한다. 이 과정에서 삭제할 오브젝트들은 "Dead" 상태가 된다.
    ==== 2번 프레임 ====
    (3)다시 메인 Update가 실행된다. 메인 Update에서는 Scene 클래스에서 각 GObject의 Update와 FinalUpdate, Render 등이 일어난다.
    먼저 오브젝트의 Update 함수는 Dead가 아닐 때에만 실행되도록 한다.
    FinalUpdate는 Dead 오브젝트의 자연스러운 Exit처리를 위해 Dead여도 실행된다.
    (4)그리고 Render 과정에서 Dead 오브젝트의 참조를 제거한다. (현재 씬은 gobejcts_라는 멤버변수로 모든 오브젝트를 가지고 있는데, 이 안에서 삭제해준다.)
    (5)그러고나면 이벤트 메니저의 Update가 실행된다. 이때는 벡터에 Dead 오브젝트들이 들어가있다. 이 오브젝트들을 일괄 소멸시킨다.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhmyH6bUT0QcaXFRz2rLRARWknUPG-jmFihrD8cct15XrJSmHOfNTbfP3EKmtfLKTTwV6rZiBpwCi_YLBZc4ZAEjV-lQZlsmspR0ofDkrfMEVyGXzPbrc6KpLLVab0qCSsDN9SyYoMrlLwOyCDUky2MSqlbkqZB9n2JwVWPdEyBZRinRI7-CYU6JpnnGA=w640-h400)](https://blogger.googleusercontent.com/img/a/AVvXsEhmyH6bUT0QcaXFRz2rLRARWknUPG-jmFihrD8cct15XrJSmHOfNTbfP3EKmtfLKTTwV6rZiBpwCi_YLBZc4ZAEjV-lQZlsmspR0ofDkrfMEVyGXzPbrc6KpLLVab0qCSsDN9SyYoMrlLwOyCDUky2MSqlbkqZB9n2JwVWPdEyBZRinRI7-CYU6JpnnGA)
  

  

  

    ### 4. 디버깅 중 DT가 무한정 늘어나는 문제
    
    디버깅 중엔 프레임이 넘어가지 않지만 시간은 계속 세어지므로 dt(한 프레임을 넘기는 데 걸리는 시간)이 무한정 늘어난다.
    따라서 디버깅 모드에서만 DT의 최댓값을 1/60으로 잡도록 한다.
    ```c++
    //Time.cpp
    bool Time::Update() {
    	LARGE_INTEGER current_count;
    	QueryPerformanceCounter(&current_count);
    	dt_ = static_cast<double>(current_count.QuadPart - prev_tick_count_.QuadPart) * 1000 / query_performance_frequency_.QuadPart;
    #ifdef _DEBUG
    	dt_ = min(dt_, 1000. / 60.); //ms단위
    #endif
    	acc_dt_ += dt_;
    	fps_++;
    	//dt_ = 1000 / fps_;
    	//acc_dt_ += dt_;
    	if (acc_dt_ >= 1000.f) {
    		TCHAR buffer[100];
    		_stprintf_s(buffer, _T("현재 FPS: %d, DT: %.10f"), fps_, dt_);
    		SetWindowTextW(hwnd_, buffer);
    		acc_dt_ = 0.f;
    		fps_ = 0;
    	}
    	prev_tick_count_.QuadPart = current_count.QuadPart;
    
    	return TRUE;
    }
    ```
    
    ### 5. 프레임 동기화 II
    
    이번엔 씬 변경을 동기화해줄 것이다.
    씬 이벤트의 틀은 위에서 만들어뒀으므로 이벤트 처리 부분만 추가하면 된다.
    
    **[씬이 변경됐을 때 예상되는 문제들]**
    \- Update()와 Render() 사이에 씬이 변경되면 안 됨. Update는 이전 씬에서 하고 Render는 다음 씬에서 하게 되면 문제 발생.
    \- 씬이 변경되면 이전 씬의 모든 오브젝트는 소멸돼야함.
    
    **[해결방법]**
    \- 이벤트 지연 처리로 씬 변경 시점 해결
    \- 부모 씬 클래스에 모든 오브젝트 삭제 함수 구현해놓고, 자식 씬의 Exit 함수에서 호출
    
    ```c++
    //전역함수 function.cpp
    void ChangeScene(SCENE_TYPE scene_type)
    {
    	Event eve = {};
    	eve.type = EVENT_TYPE::CHANGE_SCENE;
    	eve.param1 = (DWORD_PTR)scene_type;
    
    	EventManager::GetInstance()->AddEvent(eve);
    }
    
    ```
    ```c++
    //EventManager.cpp
    case EVENT_TYPE::CHANGE_SCENE:
    	{
    		//param1: 다음 씬 타입
    		//씬 변경 작업: 다음 씬으로 변경
    		SceneManager::GetInstance()->ChangeScene((SCENE_TYPE)_event.param1);
    
    	}
    	break;
    	}
    ```
    
    ```c++
    //Scene.cpp
    void Scene::DeleteGroupObjects(GROUP_TYPE type)
    {
    	SafeDeleteSTL<std::set<GObject*, GObjectPtCompare>>(gobjects_[static_cast<UINT>(type)]);
    
    }
    void Scene::DeleteAllObjects()
    {
    	for (UINT i = 0; i < static_cast<UINT>(GROUP_TYPE::END); i++) {
    		DeleteGroupObjects(static_cast<GROUP_TYPE>(i));
    	}
    }
    
    ```
    
    ```c++
    //Scene_Title.cpp
    bool Scene_Title::Exit()
    {
    	Scene::DeleteAllObjects();
    	CollisionManager::GetInstance()->ResetGroupBitmap();
    	return TRUE;
    }
    ```

