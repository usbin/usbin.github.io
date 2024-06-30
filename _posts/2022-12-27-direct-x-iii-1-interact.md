--- layout: post title: 'Direct X로 간단한 게임 만들기 - III. 게임 엔진 : Interact 상호작용 구현' date: '2022-12-27T20:51:00.004-08:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2023-01-24T22:50:48.535-08:00' blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-7718453397042151474 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/12/direct-x-iii-1-interact.html ---

    그동안 포스팅을 하며 구현한 부분은 아래와 같다.
    
    1. 윈도우 생성하기
    2. Core 클래스
    3. Time 클래스
    4. 키입력 처리(KeyManager)
    5. 오브젝트 구조(GObject, RealObject, Ui)
    6. 씬 관리(SceneManager, Scene)
    7. 충돌 처리(CollisionManager)
    8. 이벤트(오브젝트 생성, 삭제, 씬 전환) 시스템
    9. DX 클래스
    
    이 외에도
    10. 툴씬 구현(배경이미지 설정, 타일 깔기, 벽 깔기)
    11. ISavable 인터페이스(저장하기, 불러오기)
    등이 있지만 아직은 틀만 갖춰놓은 것이라 포스팅은 하지 않았다.
    
    이제 상호작용 기능을 구현할 것이다.가령, 플레이어 앞에 상자가 있다면 플레이어가 space바를 눌렀을 때 상자의 상호작용 함수가 실행되어야 한다.
    
    일단 상호작용의 기준이 되는 사각형을 정의하고, 상대 오브젝트의 상호작용 사각형과 겹칠 경우 '상호작용할 오브젝트'로 판별하기로 했다.
    상호작용 가능한 오브젝트는 모두 IInteractable 인터페이스의 OnInteract()함수를 구현한다.
    따라서 검출 후엔 IInteractable인지 확인하고 .OnInteract() 함수로 상대 오브젝트의 상호작용 핸들러를 실행할 수 있게 된다.
    
    이걸 위해서는 매 프레임마다 모든 객체의 상호작용 사각형끼리 비교해서 겹쳤는지 확인하는 과정이 필요하다.
    이미 충돌 처리를 할 때 모든 객체의 콜라이더의 겹침 여부를 체크하는 로직이 있으므로 이걸 재활용하기로 했다.
    
    '상호작용 사각형'은 콜라이더로 정의한다.
    콜라이더는 다른 콜라이더와 겹쳤을 경우 owner의 OnCollisionEnter(), OnCollisionStay(), OnCollisionExit() 함수를 실행하게 되어 있다.
    '상호작용 사각형'으로 쓰이는 콜라이더도 owner로 가질 객체가 필요하다는 것이다.
    하지만 owner를 플레이어 등의 일반 객체로 지정해버리면 실제 충돌 처리에 실행하려고 구현해둔 할 함수가 상호작용 사각형이 겹쳤을 때에도 실행돼버리게 된다.
    
    따라서 이 상호작용을 위한 객체로 Interactor라는, RealObject를 상속한 또다른 객체를 하나 만들고, 모든 RealObject는 컴포넌트로 Interactor의 포인터를 가지게 했다.
    
    Interactor는 상호작용을 위한 콜라이더를 가지고 있으며,
    OnCollisionEnter(), OnCollisionStay(), OnCollisionExit() 함수를 오버라이딩해 '현재 자신과 겹쳐있는 Interactor'의 리스트를 관리한다.
    (Enter에서 추가하고 Exit에서 제거.)
    
    RealObject는 컴포넌트로 Interactor를 가지고 있으므로, Interactor.get_interactors()를 호출해 인터렉터가 관리하는 리스트를 가져올 수 있다.
    이제 여기서 Interactor.get_owner()로 상호작용할 객체를 가져올 수 있다.
    GObject에 virtual 함수로 OnInteract를 구현해, 가져온 객체의 OnInteract()를 접근할 수 있게 한다.
    
    
    
    1. 모든 GObject
    ```
    //GObject.h
    #pragma once
    
    #include "global.h"
    #include "ISavable.h"
    class Collider;
    class Animator;
    class Sprite;
    
    class GObject : public ISavable
    {
    
    	//...
    	virtual void OnInteract(const GObject* req_obj) {};
    	//...
    }
    
    ```
    
    2. Interactor 클래스
    ```
    //Interactor.h
    #pragma once
    #include "global.h"
    #include "Collider.h"
    #include "RealObject.h"
    
    
    class Interactor : public RealObject
    {
    public:
    	Interactor();
    	~Interactor();
    
    private:
    	GObject* owner_;
    	std::vector<Interactor*> interactors_;
    	Vector2 pos_offset_;
    
    
    public:
    	void Init(GObject* owner, Vector2 pos_offset, Vector2 scale);
    
    	inline const std::vector<Interactor*>& get_interactors() { return interactors_; }; //접촉한 인터렉터 목록 반환
    
    	// GObject을(를) 통해 상속됨
    public:
    	virtual void Update() override;
    	virtual void Render(ID3D11Device* p_d3d_device) override;
    	virtual void OnCollisionEnter(Collider* collider) override;
    	virtual void OnCollisionExit(Collider* collider) override;
    	void set_scale(Vector2 scale) = delete;
    	Vector2 get_scale() = delete;
    	inline GObject* get_owner() { return owner_; };
    	friend class GObject;
    
    };
    
    
    ```
    
    ```
    //Interactor.cpp
    #include "Interactor.h"
    #include "KeyManager.h"
    #include "Collider.h"
    Interactor::Interactor()
    {
    	set_group_type(GROUP_TYPE::INTERACTOR);
    }
    
    Interactor::~Interactor()
    {
    }
    
    
    void Interactor::Init(GObject* owner, Vector2 pos_offset, Vector2 scale)
    {
    
    	owner_ = owner;
    	pos_offset_ = pos_offset;
    	Collider* collider = new Collider();
    	collider->set_owner(this);
    	collider->set_scale(scale);
    
    	set_collider(collider);
    }
    
    void Interactor::OnCollisionEnter(Collider* collider)
    {
    	if (collider->get_owner() ) {
    		Interactor* interactee = reinterpret_cast<Interactor*>(collider->get_owner());
    		if (interactee) {
    			interactors_.push_back(interactee);
    		}
    	}
    }
    
    void Interactor::OnCollisionExit(Collider* collider)
    {
    	if (collider->get_owner()) {
    		Interactor* interactee = reinterpret_cast<Interactor*>(collider->get_owner());
    		if (interactee) {
    			interactors_.erase(std::remove(interactors_.begin(), interactors_.end(), interactee), interactors_.end());
    		}
    	}
    }
    
    void Interactor::Update()
    {
    	set_pos(owner_->get_pos() + pos_offset_);
    }
    
    void Interactor::Render(ID3D11Device* p_d3d_device)
    {
    
    	Vector2 render_pos = WorldToRenderPos(get_pos());
    
    #ifdef _DEBUG
    
    	DrawRectangle(p_d3d_device, render_pos - get_collider()->get_scale() / 2.f, get_collider()->get_scale(), ARGB(0xFF00FFFF)); //노란색
    #endif
    }
    
    ```
    ```
    //Player.cpp
    
    	if (KEY_DOWN(KEY::SPACE)) {
    		//상호작용
    		const std::vector<Interactor*>& interactors = get_interactor()->get_interactors();
    		if (!interactors.empty()) {
    			GObject* obj = interactors[0]->get_owner();
    			obj->OnInteract(this);
    		}
    	}
    ```
    
    첫번째 항목과 상호작용을 하게 한다.
    비슷한 위치에 여러 항목이 있을 경우 처리가 애매한 문제가 있긴 한데, 일단은 NPC간 거리를 띄우는 것으로 해결할 수 있으니 다음에 다루는 걸로.

