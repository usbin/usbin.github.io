--- layout: post title: 'Direct X로 간단한 게임 만들기 - II. 사전지식 학습 : 3. DirectX(7) DX11에서 텍스트 그리기' date: '2022-12-12T23:17:00.003-08:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2022-12-27T02:54:53.245-08:00' thumbnail: https://blogger.googleusercontent.com/img/a/AVvXsEi6HZHlZDUycs6G-qT7TRPT5MKmGTXUXaFR6sqk3mPHZQAggK3VFAvtyTJ7RSABkkfDiUl1OqvaSFKLDxmzR\_kGgfCg0gSLXhBRAO0v4TrTfolLOdM6LPrxFuMHhmfzntMR6t2bx-pBLRaskWcOU7a4EQSnDclz3eI-4jeRv77nqOcZoMzIIK1Oie7PbA=s72-c blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-8520733060487424367 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/12/direct-x-ii-9-directx-11.html ---

    게임에서도 도스 화면에서 출력하듯이 std::cout<<"abc";나 Console.Write("abc"); 같은 식으로 편하게 텍스트를 출력하면 좋겠지만
    DirectX에서는 이런 식으로 바로 출력할 수 없는 듯하다.
    
    UI도 수제작으로 만들고 있으니 당연한 거겠지만...
    
    그래서 화면에 텍스트를 출력할 방법을 찾아보니 방법이 두가지가 있다고 한다.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEi6HZHlZDUycs6G-qT7TRPT5MKmGTXUXaFR6sqk3mPHZQAggK3VFAvtyTJ7RSABkkfDiUl1OqvaSFKLDxmzR_kGgfCg0gSLXhBRAO0v4TrTfolLOdM6LPrxFuMHhmfzntMR6t2bx-pBLRaskWcOU7a4EQSnDclz3eI-4jeRv77nqOcZoMzIIK1Oie7PbA=s16000)](https://blogger.googleusercontent.com/img/a/AVvXsEi6HZHlZDUycs6G-qT7TRPT5MKmGTXUXaFR6sqk3mPHZQAggK3VFAvtyTJ7RSABkkfDiUl1OqvaSFKLDxmzR_kGgfCg0gSLXhBRAO0v4TrTfolLOdM6LPrxFuMHhmfzntMR6t2bx-pBLRaskWcOU7a4EQSnDclz3eI-4jeRv77nqOcZoMzIIK1Oie7PbA)
  

    ##### *첫번째, 모든 글자가 그려진 이미지 파일을 따로 저장해놓고 이 이미지에서 매번 글자를 읽어와 조합해 쓰는 방법.*
    이 방법은 구현은 단순하지만 이미지 내 글자 위치를 모두 정보로 저장해놔야 하는 번거로움이 있다. 가령, "가"는 (10, 10) 좌표에 있고 가로 20pt, 세로 16pt를 차지한다는 등의 정보가 있어야 읽어올 수 있을 것이다. 알파벳과 숫자, 특문까진 할만하지만 모든 한글 글자의 조합은 만 자가 넘는데, 한글을 지원하려면 너무나 노가다가 된다.
    
    지금은 모르겠지만 옛날 닌텐도 게임이 다 이 방식을 썼다. 그래서 한글화할 때는 알파벳 이미지를 한글 이미지로 치환하는데, 일본어 알파벳 개수만큼만 한글로 치환할 수 있으니 한글로 닉네임을 지을 때마다 종종 없는 글자가 있곤 했다.
    
    ##### *두번째, 벡터 글꼴을 사용해 렌더링하는 방법.*
    Direct 2D와 3D를 섞어 쓰는 방식이고 더 어려워보이지만 기존 오픈타입 글꼴을 바로 사용할 수 있다는 장점이 있다.
    그리고 벡터이기 때문에 글씨를 키우거나 줄이는 게 더 쉽고 깔끔하게 된다.

    Direct2D를 이용해 2D 이미지의 원하는 크기, 위치에 글씨를 쓰고, 이 이미지를 다시 화면에 투영하는 방식인 것 같다.
    
    > 빈 2D 이미지의 원하는 크기, 위치에 글씨를 쓰고 ===> Direct2D 사용
    이 이미지를 다시 화면에 투영 ===> Direct3D 사용(이전 포스팅 참조)
    
    이런 식으로 동작한다.
    
    나는 이 두번째 방법을 사용하기로 했다.
    
    > [참고한 링크들]
    MS의 DirectWrite 공식 자습서 : https://learn.microsoft.com/ko-kr/windows/win32/directwrite/getting-started-with-directwrite
    Direct2D - DirectWrite를 DirectX11의 출력 파이프라인에 연결하는 방법 : https://stackoverflow.com/questions/58818886/how-to-connect-the-backbuffer-in-directx11-using-direct2d-directwrite
    CreateDxgiSurfaceRenderTarget이 실패하는 문제 : https://stackoverflow.com/questions/69330253/what-are-the-correct-arguments-for-d2d1factorycreatedxgisurfacerendertarget
    
    
    
    ### **1. Direct2D 초기화**
    현재 Direct 3D 11을 사용하고 있으므로 헤더엔 d3d11이 포함되어 있다.
    font를 사용하려면 Direct 2D 1도 같이 사용해야 하므로, 헤더에 d2d1도 포함해준다.
    ```
    //헤더파일
    #pragma comment(lib, "D2d1.lib")
    #pragma comment(lib, "Dwrite.lib")
    #include <dwrite>
    #include <d2d1>
    ```
    그리고 이전에 만들어둔 DXClass에 초기화 함수와 폰트 설정 함수, 그리고 텍스트를 그리는 함수를 만들어준다.
    
    ```
    
    //DXClass.h
    private:
    	//텍스트 렌더링
    	IDWriteFactory* p_dwrite_factory_;
    	ID2D1Factory* p_d2d1_factory_;
    	IDXGISurface1* p_d2d_rt_;
    	ID2D1RenderTarget* p_text_render_target_;
    	IDWriteTextFormat* p_text_format_;
    	//말줄임표 설정
    	DWRITE_TRIMMING trim_options_ = {
    		DWRITE_TRIMMING_GRANULARITY_CHARACTER
    	};
    	IDWriteInlineObject* p_trim_sign_;
    public:
    	void InitD2D1();
    	void SetTextFormat(tstring font_name, tstring font_locale, UINT font_size,
    		DWRITE_FONT_STYLE font_style, DWRITE_FONT_WEIGHT font_weight,
    		DWRITE_TEXT_ALIGNMENT text_alighment, DWRITE_PARAGRAPH_ALIGNMENT paragraph_alignment);
    	void RenderText(const TCHAR* text, UINT length, Vector2 pos, Vector2 scale, D2D1::ColorF color);
    ```
    p_dwrite_factory와 p_d2d1_factory는 dwrite와 d2d1 함수를 호출하기 위해 필요한 핸들 같은 것이다.
    p_d2d_rt_는 2D Surface, p_text_render_target_은 2D Render Target이다.
    
    전부 다 InitD2D1 함수에서 초기화해준다.
    
    ```
    void DXClass::InitD2D1()
    {
    
    	//Release로 해제해줘야 하는 리소스임.
    	if (FAILED(D2D1CreateFactory(D2D1_FACTORY_TYPE_SINGLE_THREADED, D2D1_FACTORY_OPTIONS(), &p_d2d1_factory_))) {
    		return;
    	}
    
    	if (FAILED(DWriteCreateFactory(
    		DWRITE_FACTORY_TYPE_SHARED,
    		__uuidof(IDWriteFactory),
    		reinterpret_cast<IUnknown**>(&p_dwrite_factory_)
    	)))
    	{
    		return;
    	}
    
    	if (FAILED(p_swap_chain_->GetBuffer(0, IID_PPV_ARGS(&p_d2d_rt_)))) {
    		return;
    	};
    	D2D1_RENDER_TARGET_PROPERTIES d2d_rt_props = D2D1::RenderTargetProperties(D2D1_RENDER_TARGET_TYPE_DEFAULT, D2D1::PixelFormat(DXGI_FORMAT_UNKNOWN, D2D1_ALPHA_MODE_PREMULTIPLIED), 0, 0);
    	if (FAILED(p_d2d1_factory_->CreateDxgiSurfaceRenderTarget(p_d2d_rt_, &d2d_rt_props, &p_text_render_target_))) {
    		return;
    	}
    
    
    	//폰트 기본값 설정
    	SetTextFormat(_T("둥근모꼴"), _T("ko-kr"), 20.0f, DWRITE_FONT_STYLE_NORMAL, DWRITE_FONT_WEIGHT_REGULAR,
    		DWRITE_TEXT_ALIGNMENT_CENTER, DWRITE_PARAGRAPH_ALIGNMENT_CENTER);
    	
    }
    
    ```
    back buffer를 얻어와서 2D Render Target에 연결시키는 것 같다.
    그럼 2D Render Target이 그래픽 파이프라인에 끼워져서, 여기에 그리는 것이 화면에 그려지는 듯하다.
    
    
    폰트 기본값은 컴퓨터에 설치돼있는 '둥근모꼴'을 사용할 건데, 반드시 윈도우에 설치돼있는 폰트를 사용해야 한다.
    난 커스텀 폰트를 사용하고 싶기 때문에 폰트를 포함해서 배포하는 방법을 고려해야 할 듯.
    
    그리고 화면 크기가 변경됐을 때 swap chain이 재생성되므로, swap chain에서 파생되는 p_d2d_rt와 p_text_render_target도 재생성해줘야 한다.
    전에 구현해둔 해상도 리셋 함수의 아랫부분에 추가해준다.
    ```
    void DXClass::ResetResolution(Vector2 new_resolution)
    {
    	//==============================
    	// 메인 Render Target View 재초기화
    	//===============================
    	p_render_target_view_->Release();
    	p_swap_chain_->ResizeBuffers(1, new_resolution.x, new_resolution.y, DXGI_FORMAT_R8G8B8A8_UNORM, 0);
    
    	// Render Target View 생성
    	ID3D11Texture2D* p_back_buffer;
    	if (FAILED(p_swap_chain_->GetBuffer(0, __uuidof(ID3D11Texture2D), (LPVOID*)&p_back_buffer))) {
    		return;
    	}
    
    	if (FAILED(p_d3d_device_->CreateRenderTargetView(p_back_buffer, NULL, &p_render_target_view_))) {
    		p_back_buffer->Release();
    		return;
    	}
    	p_back_buffer->Release();
    	p_immediate_context_->OMSetRenderTargets(1, &p_render_target_view_, NULL);
    
    
    	// Viewport 초기화
    	D3D11_VIEWPORT viewport;
    	viewport.Width = new_resolution.x;
    	viewport.Height = new_resolution.y;
    	viewport.MinDepth = 0.0f;
    	viewport.MaxDepth = 1.0f;
    	viewport.TopLeftX = 0;
    	viewport.TopLeftY = 0;
    	p_immediate_context_->RSSetViewports(1, &viewport);
    
    	//==========================================
    	// 텍스트 렌더링용 Render Target View 재초기화
    	//==========================================
    	if (p_d2d_rt_) {
    		p_d2d_rt_->Release();
    		p_d2d_rt_ = nullptr;
    	}
    	if (p_text_render_target_) {
    		p_text_render_target_->Release();
    		p_text_render_target_ = nullptr;
    	}
    	if (FAILED(p_swap_chain_->GetBuffer(0, IID_PPV_ARGS(&p_d2d_rt_)))) {
    		return;
    	};
    	D2D1_RENDER_TARGET_PROPERTIES d2d_rt_props = D2D1::RenderTargetProperties(D2D1_RENDER_TARGET_TYPE_DEFAULT, D2D1::PixelFormat(DXGI_FORMAT_UNKNOWN, D2D1_ALPHA_MODE_PREMULTIPLIED), 0, 0);
    	if (FAILED(p_d2d1_factory_->CreateDxgiSurfaceRenderTarget(p_d2d_rt_, &d2d_rt_props, &p_text_render_target_))) {
    		return;
    	}
    
    
    }
    ```
    위 코드는 프로그램의 메인 루틴에서 해상도 변경이 감지될 때마다 호출되는 함수이다.
    
    
    
    
    
    ```
    void DXClass::SetTextFormat(tstring font_name, tstring font_locale, UINT font_size,
    	DWRITE_FONT_STYLE font_style, DWRITE_FONT_WEIGHT font_weight,
    	DWRITE_TEXT_ALIGNMENT text_alighment, DWRITE_PARAGRAPH_ALIGNMENT paragraph_alignment)
    {
    	if (p_text_format_) {
    		p_text_format_->Release();
    	}
    	if (p_trim_sign_) {
    		p_trim_sign_->Release();
    	}
    	//글꼴, 두께, 스트레치, 스타일 및 로캘 지정
    	if (FAILED(p_dwrite_factory_->CreateTextFormat
    	(	font_name.c_str()
    		, NULL
    		, font_weight
    		, font_style
    		, DWRITE_FONT_STRETCH_NORMAL
    		, font_size
    		, font_locale.c_str()
    		, &p_text_format_
    	)
    	)) {
    		assert(0);
    	}
    	//정렬 설정
    	if (FAILED(p_text_format_->SetTextAlignment(DWRITE_TEXT_ALIGNMENT_CENTER))) {
    		assert(0);
    	}
    	if (FAILED(p_text_format_->SetParagraphAlignment(DWRITE_PARAGRAPH_ALIGNMENT_CENTER))) {
    		assert(0);
    	}
    	//지정된 크기가 부족할 때 trim 설정 => 말줄임표
    	p_dwrite_factory_->CreateEllipsisTrimmingSign(p_text_format_, &p_trim_sign_);
    	p_text_format_->SetTrimming(&trim_options_, p_trim_sign_);
    }
    ```
    텍스트의 포맷을 설정하는 함수이다.
    텍스트의 글꼴, 두께, 스타일, 로캘, 가로 정렬 및 세로 정렬 방식, trim을 설정한다.
    
    text_format과 trim_sign은 설정을 바꾸려면 새로 생성해야 하기 때문에 멤버변수가 이미 초기화되어 있다면 Release를 먼저 해줘야 한다.
    
    
    ### **2. 텍스트 렌더링**
    
    아래는 텍스트를 그리는 함수이다.
    
    ```
    
    void DXClass::RenderText(const TCHAR* text, UINT length, Vector2 pos, Vector2 scale, D2D1::ColorF color)
    {
    	D2D1_RECT_F layout_rect;
    
    	ID2D1SolidColorBrush* p_brush;
    	if (FAILED(p_text_render_target_->CreateSolidColorBrush(
    		D2D1::ColorF(color),
    		&p_brush
    	))) {
    		return;
    	}
    	FLOAT dpi_scale_x = 1;
    	FLOAT dpi_scale_y = 1;
    	layout_rect = D2D1::RectF(
    		static_cast<FLOAT>(pos.x - scale.x/2.f) / dpi_scale_x,
    		static_cast<FLOAT>(pos.y - scale.y/2.f) / dpi_scale_y,
    		static_cast<FLOAT>(pos.x+scale.x/2.f) / dpi_scale_x,
    		static_cast<FLOAT>(pos.y+scale.y/2.f) / dpi_scale_y
    	);
    
    	p_text_render_target_->BeginDraw();
    	p_text_render_target_->SetTransform(D2D1::IdentityMatrix());
    	p_text_render_target_->DrawTextW(
    		text,
    		length,
    		p_text_format_,
    		layout_rect,
    		p_brush
    	);
    	p_text_render_target_->EndDraw();
    
    
    	p_brush->Release();
    }
    
    ```
    매 프레임 brush를 생성하고 해제하며 쓰게 되는데,
    자주 쓰는 brush는 미리 초기화해서 배열로 저장해놓고 꺼내쓰면 좋을 것 같다.
    이 부분은 나중에 개선하는 걸로.
    
    
    이제 SetTextFormat() 함수로 텍스트의 폰트, 폰트 크기, 스타일, 두께, 정렬 방식을 지정할 수 있고,
    RenderText 함수로 텍스트가 그려질 위치와 크기, 색깔을 지정하며 텍스트를 그릴 수 있다.
    
    
    ### *[해결된 문제]*
    
    초기화 구문을 보면,
    ```
    	if (FAILED(p_d2d1_factory_->CreateDxgiSurfaceRenderTarget(p_d2d_rt_, &d2d_rt_props, &p_text_render_target_))) {
    		return;
    	}
    ```
    에서 계속 잘못된 인자라며 에러가 났는데, 구글링을 해본 결과 DWrite, D2d1을 Direct 3D와 섞어 쓰려면 Swap Chain에 플래그를 하나 넣어서 생성해줘야 한다고 하더라.
    
    
    ```
    if (FAILED(D3D11CreateDeviceAndSwapChain(NULL, D3D_DRIVER_TYPE_HARDWARE, NULL, D3D11_CREATE_DEVICE_BGRA_SUPPORT, featureLevels, numFeatureLevels,
    		D3D11_SDK_VERSION, &swap_desc, &p_swap_chain_, &p_d3d_device_, NULL, &p_immediate_context_))) {
    		return E_FAIL;
    	}
    ```
    DX를 초기화하는 함수에서 Swap Chain을 생성하는 함수를 고쳐준다.
    D3D11_CREATE_DEVICE_BGRA_SUPPORT 라는 플래그인데,

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhscMuBUczvYX_jIx2Jy-5WSEP_yu-Gom_aE5VLyA7YZqWwKaDafd_obeAYPigmaq8KCi_gyoB-C7T5SzVhe4rOLzIeM5NpH5YkX4bEfg0pVTcr0OFlHnF-UUgwzxRlaJxPmDLc_S5fwXULnh7ryEmm9kKNmzDqQ3Y8eAHfyye1Sw08i_jaoWyT-BcurA=w640-h276)](https://blogger.googleusercontent.com/img/a/AVvXsEhscMuBUczvYX_jIx2Jy-5WSEP_yu-Gom_aE5VLyA7YZqWwKaDafd_obeAYPigmaq8KCi_gyoB-C7T5SzVhe4rOLzIeM5NpH5YkX4bEfg0pVTcr0OFlHnF-UUgwzxRlaJxPmDLc_S5fwXULnh7ryEmm9kKNmzDqQ3Y8eAHfyye1Sw08i_jaoWyT-BcurA)
  

  

  

    라고 한다.
    
    삽질을 꽤 오래 했는데 너무 쉽게 해결됐다.

