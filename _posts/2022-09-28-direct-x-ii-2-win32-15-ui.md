--- layout: post title: 'Direct X로 간단한 게임 만들기 - III. 게임엔진 : UI' date: '2022-09-28T05:08:00.003-07:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2023-01-24T22:49:56.125-08:00' thumbnail: https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgfNl5Dnmq1pw4Tmaq2nd1-VqalyKWioGOj5SANqMbsgJ6ik02EDRykDYLyMz2PMDMu8gdS9kVeBxT9RE4bVEbKBI4dkezmyRyn6nzEtXEhWim4K2n\_F91JHmmr7mJiCm\_hyoDi6gJKwxeV9vEUW9spfC7JsEDtivCRuLkmSkLZDHEf78KFz-DmdcyxsQ/s72-w640-c-h508/Ui%20%ED%85%8C%EC%8A%A4%ED%8A%B8.gif blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-3897041464073651395 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/09/direct-x-ii-2-win32-15-ui.html ---

    > 목차
    [1. GObject 상속 구조 다시 잡기](#user-content-1-gobject-상속-구조-다시-잡기)
    [2. Ui 클래스](#user-content-2-ui-클래스)
    [3. UiManager 클래스](#user-content-3-uimanager-클래스)
    [4. PanelUi 클래스](#user-content-4-panelui-클래스)
    [5. ButtonUi 클래스](#user-content-5-buttonui-클래스)
    [6. ImageUi 클래스](#user-content-6-imageui-클래스)
    [7. 실행화면](#user-content-7-실행화면)
    
    
    ### 1. GObject 상속 구조 다시 잡기
    
    GObject -> UI/RealObject
    RealObject -> Player/Missile/Monster
    
    RealObject: FinalUpdate와 ComponentRender를 최상위 부모가 맡아서 함. 자식은 오버라이딩x
    UI : FinalUpdate를 자식들이 오버라이딩함. 클릭 체크해야 하기 때문. 또한 Update와 Render, FinalUpdate 모두 부모(Ui)의 것을 먼저 실행시켜야 함. 
    
    \- RealObject로 FinalUpdate, ComponentRender 옮겨오고, 여기서 fin
    al 선언
    \- ui에서 필요하지 않은 멤버들(collider, animator, direction)도 RealObject로 옮겨옴.
    
    **[Ui 설계]**
    \- 모든 UI는 Ui 클래스를 상속함. Ui 클래스는 단독으로 생성될 수 없음(순수 가상함수 포함)
    \- 모든 UI는 부모 UI와 자식 UI들의 리스트를 가짐.
    \- 루트 UI만 씬의 오브젝트로 등록되고 자식 UI들은 부모의 UI의 자식으로만 등록됨.
    \- 모든 UI는 매 프레임 마우스가 자신 위에 있는지만 체크해서 저장하고 있음.
    \- UiManager 클래스는 매 프레임 모든 UI와 그 자식들을 탐색하면서 (1) MOUSE_ON 되어있고 (2) 가장 위에 그려져있는 UI를 찾아서 target으로 지정하고, MOUSE_HOLD 상태를 검사하고 설정함.
    \- 또한 이전 프레임의 target을 저장하고 있음. 
    \- 이전 프레임과 현재 프레임의 타깃 정보, MOUSE_HOLD 상태를 검사해 타깃의 MOUSE_ON, MOUSE_DOWN, MOUSE_UP, MOUSE_CLICK 함수를 실행시킴.
    
    
    ### 2. Ui 클래스
    모든 Ui의 부모 클래스.
    \- 부모 Ui와 자식 Ui 리스트를 가짐
    \- 현재 프레임에 마우스가 위에 있는지 체크한 bool값
    \- 이 ui를 타깃으로 클릭이 눌려있는지 체크한 bool값(외부에서 설정해줌)
    \- 최종 위치(부모를 기준으로 계산한 world 좌표)
    \- 위치가 static인가(화면상에서 고정돼있는가) bool값
    \- Update, FinalUpdate에서 공통 작업 정의, 자식 Ui들 실행
    \- Update: 현재 프레임에 마우스가 본인 위에 있는지 체크
    \- FinalUpdate: 부모 위치를 기준으로 계산한 최종 위치 저장.
    
    \*Ui를 상속하는 모든 클래스는 Ui의 Update, FinalUpdate를 먼저 실행하고 자신의 것을 실행함.
    
    ```c++
    //Ui.h
    #pragma once
    #include "GObject.h"
    class Ui : public GObject
    {
    public:
    	Ui(bool is_static_pos);
    	~Ui();
    private:
    	bool is_static_pos_; // 카메라에 영향받지 않는 좌표계를 쓰는가(일반적인 Ui처럼 위치 고정)
    	Vector2 final_pos_;
    	Ui* parent_; // 이 ui가 자식으로 있는 ui. root ui일 경우 nullptr.
    	std::vector<Ui*> children_;	// 이 ui 위에 있는 child ui. 
    
    
    	bool mouse_on_check_; // 위에 다른 ui가 있든 말든 단순 좌표만으로 체크함
    
    	bool lbutton_hold_; // 가장 위에서 실제로 이벤트를 받았을 때에만 체크됨.
    
    
    
    public:
    	virtual void Update() override;
    	virtual void FinalUpdate() override;
    	virtual void Render(HDC hdc) = 0 ;
    protected:
    	virtual void ChildrenUpdate() final;
    	virtual void ChildrenFinalUpdate() final;
    	virtual void ChildrenRender(HDC hdc) final;
    
    
    
    public:
    	virtual void MouseOn() {};
    	virtual void LbuttonClick() {};
    	virtual void LbuttonDown() {};
    	virtual void LbuttonUp() {};
    
    
    	inline void AddChild(Ui* child) { children_.push_back(child); };
    	inline const std::vector<Ui*>& get_children() { return children_; };
    	inline bool	is_static_pos() { return is_static_pos_; };
    	inline const Vector2& get_final_pos() { return final_pos_; };
    	inline Ui* get_parent() { return parent_; };
    	inline void set_parent(Ui* parent) { parent_ = parent; };
    	inline bool get_mouse_on() { return mouse_on_check_; };
    	inline bool get_lbutton_hold() { return lbutton_hold_; };
    	inline void set_lbutton_hold(bool b) { lbutton_hold_ = b; };
    };
    
    
    ```
    
    ```c++
    //Ui.cpp
    #include "Ui.h"
    #include "Core.h"
    Ui::Ui(bool is_static_pos)
    	: is_static_pos_(is_static_pos)
    	, mouse_on_check_(false)
    	, lbutton_hold_(false)
    	, parent_(nullptr)
    {
    }
    
    Ui::~Ui()
    {
    	SafeDeleteVector<Ui*>(children_);
    }
    
    
    
    void Ui::Update()
    {
    	
    	POINT pt;
    	GetCursorPos(&pt);
    	ScreenToClient(Core::GetInstance()->get_main_hwnd(), &pt);
    	Vector2 mouse_pos{ pt };
    	Vector2 pos = get_final_pos();
    	Vector2 scale = get_scale();
    	//화면 안에서 마우스 이벤트가 일어났을 경우만
    	if (Vector2{ 0, 0 } <= mouse_pos && mouse_pos <= Core::GetInstance()->get_resolution()) {
    		//mouse 좌표는 항상 화면을 기준으로 한 render 좌표이므로 world 좌표로 변환해줌.
    		mouse_pos = RenderToWorldPos(mouse_pos);
    		//pos는 이 ui가 static pos를 사용하고 있을 경우에만 world 좌표로 변환해줌.
    		if (is_static_pos_) pos = RenderToWorldPos(pos);
    
    		mouse_on_check_ = (pos <= mouse_pos && mouse_pos <= pos + scale);
    
    	}
    	else {
    		mouse_on_check_ = false;
    	}
    
    	ChildrenUpdate();
    }
    
    void Ui::FinalUpdate()
    {
    	final_pos_ = get_pos();
    	//자식일 경우: 부모의 0,0을 기준으로 한 상대좌표임.
    	if (get_parent() != nullptr) {
    		final_pos_ += get_parent()->get_final_pos();
    	}
    
    	ChildrenFinalUpdate();
    }
    
    
    //=============================================================
    //
    // 자식 Ui 클래스에서 Update, FinalUpdate, Render 후에
    // 실행시켜줘야 하는 함수들.
    //
    //=============================================================
    void Ui::ChildrenUpdate()
    {
    	for (auto iter = children_.begin(); iter != children_.end(); iter++) {
    		(*iter)->Update();
    	}
    }
    
    void Ui::ChildrenFinalUpdate()
    {
    	for (auto iter = children_.begin(); iter != children_.end(); iter++) {
    		(*iter)->FinalUpdate();
    	}
    }
    
    
    void Ui::ChildrenRender(HDC hdc)
    {
    	for (auto iter = children_.begin(); iter != children_.end(); iter++) {
    		(*iter)->Render(hdc);
    	}
    }
    ```
    
    ### 3. UiManager 클래스
    현재 씬의 모든 Ui를 순회하며 타깃 Ui를 찾아내고 해당 Ui에 대해 이벤트 발생시킴.
    반드시 씬의 Update 후에 UiManager의 Update가 이루어져야 함.
    
    ```c++
    //UiManager.h
    #pragma once
    #include "global.h"
    class Ui;
    //매프레임 모든 Ui를 순회하며 MOUSE_CLICK, MOUSE_ON, MOUSE_UP, MOUSE_DOWN를 체크하고 콜백을 실행시켜주는 매니저
    class UiManager
    {
    	SINGLETON(UiManager);
    private:
    	Ui* prev_downed_target; //최근에 DOWN된 Ui를 저장하고 있다가 UP 시점에 일치하면 CLICK 이벤트로 간주.
    public:
    	void FinalUpdate();
    
    };
    
    
    ```
    
    ```c++
    //UiManager.cpp
    #include "UiManager.h"
    #include "SceneManager.h"
    #include "Ui.h"
    
    UiManager::UiManager() {
    
    }
    
    UiManager::~UiManager() {
    
    }
    
    void UiManager::FinalUpdate()
    {
    	//[1] 타깃 ui의 루트 ui를 맨 위로 올리기
    	//[2] 타깃 ui에 대하여 마우스 이벤트 발생
    	//[3] 타깃 ui 제외한 모든 ui에 대하여 초기화해줄 항목 초기화(상태 변수 등)
    
    	//1. 마우스 위치의 맨 위에 있는 ui를 찾기(=마우스가 올려져 있으면서 벡터의 가장 뒤에 있는 ui)
    	// - 현재 씬의 모든 Ui에서 가져옴.
    
    	const std::vector<GObject*>& uis = SceneManager::GetInstance()->get_current_scene()->GetGroupObjects(GROUP_TYPE::UI);
    	std::vector<GObject*> ui_heap = uis; // bfs 탐색용 힙(모든 자식들 계층 순서(너비 우선)대로 탐색)
    	Ui* target_ui = nullptr;
    	for (int i = 0; i < ui_heap.size(); i++) {
    		Ui* ui = static_cast<Ui*>(ui_heap[i]);
    		
    		for (int j = 0; j < ui->get_children().size(); j++) {
    			ui_heap.push_back(ui->get_children()[j]);
    		}
    		//이번 프레임에 target_ui가 확실히 아닌 것은 전부 lbutton_hold를 false로 설정해줌. 
    		if (ui->get_mouse_on()) {
    			if (target_ui != nullptr) {
    				target_ui -> set_lbutton_hold(false);
    			}
    			target_ui = ui;
    		}
    		else {
    			ui->set_lbutton_hold(false);
    		}
    	}
    
    
    	Ui* target_ui_root = target_ui;
    	if (target_ui_root != nullptr) {
    		while (target_ui_root->get_parent() != nullptr) {
    			target_ui_root = target_ui_root->get_parent();
    		}
    	}
    
    	// - target_ui는 확정적으로 MOUSE_ON인 상태
    	// - prev_downed_target은 이전에 클릭한 ui가 저장돼있음.
    	//2. MOUSE_ON된 target_ui에 대하여 모든 마우스 이벤트 검사(ON 제외 아무 이벤트도 발생하지 않았을 수도 있음)
    	if (target_ui != nullptr) {
    		target_ui->MouseOn();
    		// 이번 프레임의 마우스 상태가 DOWN임 = MOUSE_DOWN
    		if (KEY_DOWN(KEY::LBUTTON)) {
    			target_ui->LbuttonDown();
    			prev_downed_target = target_ui;
    			SceneManager::GetInstance()->get_current_scene()->ObjectToTop(GROUP_TYPE::UI, dynamic_cast<GObject*>(target_ui_root));
    		}
    		// 이번 프레임의 마우스 상태가 UP임 = MOUSE_UP
    		else if (KEY_UP(KEY::LBUTTON)) {
    			target_ui->LbuttonUp();
    			if (prev_downed_target == target_ui) {
    				target_ui->LbuttonClick();
    			}
    			prev_downed_target = nullptr;
    		}
    
    		target_ui->set_lbutton_hold(KEY_HOLD(KEY::LBUTTON));
    		
    	}
    	// ui가 아닌 곳에서 마우스가 떼어졌을 경우 -> prev_downed_target 초기화
    	else if (KEY_UP(KEY::LBUTTON)) {
    		prev_downed_target = nullptr;
    	}
    	
    	
    }
    
    
    
    ```
    
    ### 4. PanelUi 클래스
    드래그해서 위치를 이동할 수 있는 PanelUi.
    
    ```c++
    //PanelUi.h
    #pragma once
    #include "global.h"
    #include "Ui.h"
    class PanelUi : public Ui
    {
    public:
    	PanelUi();
    	~PanelUi();
    private:
    
    	Vector2 prev_drag_pos_;
    	bool dragging_;
    public:
    	virtual void Update() override;
    	virtual void FinalUpdate() override;
    	virtual void Render(HDC hdc) override;
    
    	virtual void MouseOn() override;
    	virtual void LbuttonClick() override;
    	virtual void LbuttonDown() override;
    	virtual void LbuttonUp() override;
    };
    
    
    ```
    
    ```c++
    //PanelUi.cpp
    #include "PanelUi.h"
    
    PanelUi::PanelUi()
    	: Ui(true)
    	, prev_drag_pos_{}
    	, dragging_(false)
    {
    }
    
    PanelUi::~PanelUi()
    {
    }
    
    
    
    void PanelUi::Update()
    {
    	Ui::Update();
    
    }
    
    void PanelUi::FinalUpdate()
    {
    	Ui::FinalUpdate();
    
    	//마우스 LBUTTON이 떨어졌다면 dragging도 끝난 것.
    	if (!KEY_HOLD(KEY::LBUTTON)) {
    		dragging_ = false;
    	}
    
    	//드래그에 따라 위치 이동
    	if (dragging_) {
    		Vector2 diff = GET_MOUSE_POS() - prev_drag_pos_;
    		set_pos(get_pos() + diff);
    		prev_drag_pos_ = GET_MOUSE_POS();
    	}
    
    }
    
    void PanelUi::Render(HDC hdc)
    {
    	Vector2 pos = get_final_pos();
    	Vector2 scale = get_scale();
    
    	if (!is_static_pos()) pos = WorldToRenderPos(pos);
    
    	if (dragging_) {
    		SelectGdi _(hdc, PEN_TYPE::GREEN);
    		Rectangle(hdc, static_cast<int>(pos.x), static_cast<int>(pos.y), static_cast<int>(pos.x + scale.x), static_cast<int>(pos.y + scale.y));
    
    	}
    	else {
    		Rectangle(hdc, static_cast<int>(pos.x), static_cast<int>(pos.y), static_cast<int>(pos.x + scale.x), static_cast<int>(pos.y + scale.y));
    	}
    
    	ChildrenRender(hdc);
    
    }
    
    void PanelUi::MouseOn()
    {
    }
    
    void PanelUi::LbuttonClick()
    {
    }
    
    void PanelUi::LbuttonDown()
    {
    	//패널 안에서 마우스 누르면 드래그 시작
    	prev_drag_pos_ = GET_MOUSE_POS();
    	dragging_ = true;
    
    }
    
    void PanelUi::LbuttonUp()
    {
    }
    
    ```
    
    ### 5. ButtonUi 클래스
    클릭 이벤트를 등록할 수 있는 ButtonUi.
    
    ```c++
    //ButtonUi.h
    #pragma once
    //ButtonUi는 외부에서 클릭 이벤트 핸들러 등록 가능.
    #include "global.h"
    #include "Ui.h"
    typedef void (*OnClickHandler)(DWORD_PTR param1, DWORD_PTR param2);
    struct OnClickHandlerParams {
    	OnClickHandler on_click;
    	DWORD_PTR param1;
    	DWORD_PTR param2;
    };
    class ButtonUi : public Ui
    {
    public:
    	ButtonUi(bool is_static_pos);
    	~ButtonUi();
    private:
    	// OnClick 핸들러 함수들
    	std::vector<OnClickHandlerParams> on_click_handler;
    
    
    public:
    	virtual void Render(HDC hdc) override;
    	virtual void MouseOn() override;
    	virtual void LbuttonClick() override;
    	virtual void LbuttonDown() override;
    	virtual void LbuttonUp() override;
    	void AddOnClickHandler(OnClickHandler on_click, DWORD_PTR param1, DWORD_PTR param2);
    
    };
    
    
    ```
    
    ```c++
    //ButtonUi.cpp
    #include "ButtonUi.h"
    
    ButtonUi::ButtonUi(bool is_static_pos)
    	: Ui(is_static_pos)
    	, on_click_handler{}
    {
    }
    
    ButtonUi::~ButtonUi()
    {
    }
    
    void ButtonUi::Render(HDC hdc)
    {
    	Vector2 pos = get_final_pos();
    	Vector2 scale = get_scale();
    
    	if (!is_static_pos()) pos = WorldToRenderPos(pos);
    
    	if (get_lbutton_hold()) {
    		SelectGdi _(hdc, PEN_TYPE::RED);
    		Rectangle(hdc, static_cast<int>(pos.x), static_cast<int>(pos.y), static_cast<int>(pos.x + scale.x), static_cast<int>(pos.y + scale.y));
    	}
    	else {
    
    		Rectangle(hdc, static_cast<int>(pos.x), static_cast<int>(pos.y), static_cast<int>(pos.x + scale.x), static_cast<int>(pos.y + scale.y));
    	}
    
    	ChildrenRender(hdc);
    }
    
    void ButtonUi::MouseOn()
    {
    }
    
    void ButtonUi::LbuttonClick()
    {
    	for (int i = 0; i < on_click_handler.size(); i++) {
    		OnClickHandlerParams params = (on_click_handler[i]);
    		params.on_click(params.param1, params.param2);
    	}
    }
    
    void ButtonUi::LbuttonDown()
    {
    
    }
    
    void ButtonUi::LbuttonUp()
    {
    }
    
    void ButtonUi::AddOnClickHandler(OnClickHandler on_click, DWORD_PTR param1, DWORD_PTR param2)
    {
    	OnClickHandlerParams params{ on_click, param1, param2 };
    	on_click_handler.push_back(params);
    }
    
    ```
    
    ### 6. ImageUi 클래스
    배경 이미지를 설정할 수 있는 ImageUi.
    
    ```c++
    //ImageUi.h
    #pragma once
    #include "Ui.h"
    class Texture;
    
    class ImageUi : public Ui
    {
    public:
    	ImageUi(bool is_static_pos);
    	~ImageUi();
    
    private:
    	Texture* texture_;
    public:
    	virtual void Render(HDC hdc) override;
    	inline void set_texture(Texture* texture) { texture_ = texture; };
    	inline Texture* get_texture() { return texture_; };
    };
    
    
    ```
    
    ```c++
    //ImageUi.cpp
    #include "ImageUi.h"
    #include "Texture.h"
    
    ImageUi::ImageUi(bool is_static_pos)
    	: Ui(is_static_pos)
    	, texture_(nullptr)
    {
    }
    
    ImageUi::~ImageUi()
    {
    }
    
    void ImageUi::Render(HDC hdc)
    {
    	Vector2 pos = get_final_pos();
    	Vector2 scale = get_scale();
    
    	if (!is_static_pos()) pos = WorldToRenderPos(pos);
    
    	if (texture_ != nullptr) {
    		TransparentBlt(
    			hdc //목적지 dc
    			, static_cast<int>(pos.x) //left 좌표
    			, static_cast<int>(pos.y) //top 좌표
    			, static_cast<int>(scale.x) //가로 길이
    			, static_cast<int>(scale.y) //세로 길이
    			, texture_->get_hdc() //소스 dc(=비트맵이 선택된 메모리dc)
    			, static_cast<int>(0) //비트맵의 left 좌표
    			, static_cast<int>(0) //비트맵의 top 좌표
    			, static_cast<int>(texture_->get_width())	//비트맵의 가로 길이
    			, static_cast<int>(texture_->get_height())	//비트맵의 세로 길이
    			, RGB(255, 0, 255)); //무시할 픽셀 색상
    	}
    }
    
    ```
    
    ### 7. 실행화면
    ```c++
    //Scene_Title.cpp
    
    GObject* gobj = new Player();
    	gobj->set_pos(Vector2{ 500, 500 });
    	gobj->set_scale(Vector2{ 50, 50 });
    	gobj->set_group_type(GROUP_TYPE::PLAYER);
    	CreateGObject(gobj, GROUP_TYPE::PLAYER);
    
    	Ui* ui1 = new PanelUi();
    	ui1->set_pos(Vector2{ 0, 0 });
    	ui1->set_scale(Vector2{ 200, 400 });
    	ui1->set_group_type(GROUP_TYPE::UI);
    	ui1->set_name(_T("UI1"));
    	CreateGObject(ui1, GROUP_TYPE::UI);
    
    	Ui* ui2 = new PanelUi();
    	ui2->set_pos(Vector2{ 0, 0 });
    	ui2->set_scale(Vector2{ 200, 400 });
    	ui2->set_group_type(GROUP_TYPE::UI);
    	ui2->set_name(_T("UI2"));
    	CreateGObject(ui2, GROUP_TYPE::UI);
    
    	Ui* child_panel1 = new PanelUi();
    	child_panel1->set_pos(Vector2{ 0, 0 });
    	child_panel1->set_scale(Vector2{ 150, 50 });
    	child_panel1->set_group_type(GROUP_TYPE::UI);
    	child_panel1->set_parent(ui1);
    	child_panel1->set_name(_T("child_ui1"));
    	ui1->AddChild(child_panel1);
    
    	ButtonUi* button_ui = new ButtonUi(true);
    	button_ui->set_pos(Vector2{ 0, 0 });
    	button_ui->set_scale(Vector2{ 40, 20 });
    	button_ui->set_group_type(GROUP_TYPE::UI);
    	button_ui->set_parent(child_panel1);
    	button_ui->set_name(_T("button_ui"));
    	child_panel1->AddChild(button_ui);
    	button_ui->AddOnClickHandler([](DWORD_PTR param1, DWORD_PTR param2) {
    		SetWindowText(Core::GetInstance()->get_main_hwnd(), _T("클릭함!"));
    	}, 0, 0);
    
    	Ui* child_panel2 = new PanelUi();
    	child_panel2->set_pos(Vector2{ 150, 150 });
    	child_panel2->set_scale(Vector2{ 50, 50 });
    	child_panel2->set_group_type(GROUP_TYPE::UI);
    	child_panel2->set_parent(ui1);
    	child_panel2->set_name(_T("child_ui2"));
    	ui1->AddChild(child_panel2);
    
    	ImageUi* image_ui = new ImageUi(true);
    	image_ui->set_pos(Vector2{ 50, 300 });
    	image_ui->set_scale(Vector2{ 100, 100 });
    	image_ui->set_group_type(GROUP_TYPE::UI);
    	image_ui->set_parent(ui1);
    	image_ui->set_name(_T("image_ui"));
    	Texture* texture = ResManager::GetInstance()->LoadTexture(_T("Image Background"), _T("texture\\sample.bmp"));
    	image_ui->set_texture(texture);
    	ui1->AddChild(image_ui);
    ```

움
[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgfNl5Dnmq1pw4Tmaq2nd1-VqalyKWioGOj5SANqMbsgJ6ik02EDRykDYLyMz2PMDMu8gdS9kVeBxT9RE4bVEbKBI4dkezmyRyn6nzEtXEhWim4K2n_F91JHmmr7mJiCm_hyoDi6gJKwxeV9vEUW9spfC7JsEDtivCRuLkmSkLZDHEf78KFz-DmdcyxsQ/w640-h508/Ui%20%ED%85%8C%EC%8A%A4%ED%8A%B8.gif)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgfNl5Dnmq1pw4Tmaq2nd1-VqalyKWioGOj5SANqMbsgJ6ik02EDRykDYLyMz2PMDMu8gdS9kVeBxT9RE4bVEbKBI4dkezmyRyn6nzEtXEhWim4K2n_F91JHmmr7mJiCm_hyoDi6gJKwxeV9vEUW9spfC7JsEDtivCRuLkmSkLZDHEf78KFz-DmdcyxsQ/s1016/Ui%20%ED%85%8C%EC%8A%A4%ED%8A%B8.gif)
  
짤
