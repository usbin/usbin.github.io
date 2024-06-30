--- layout: post title: 'Direct X로 간단한 게임 만들기 - II. 사전지식 학습 : 3. DirectX(6) - 해상도 변경' date: '2022-11-30T20:31:00.007-08:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2022-12-27T02:54:38.338-08:00' blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-5438988465284665703 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/11/direct-x-ii-8-directx-11.html ---

    창 크기를 지금은 자유자재로 조절할 수 있는데,
    이 부분도 나중에 고정 크기 몇가지 중 설정하는 것으로 변경해야한다.
    
    아무튼 지금 문제가 창 크기를 바꾸면 이미지가 좀 이상해진다. 
    선은 가늘어지거나 굵어지거나 없어지기도 하고, png 이미지는 저화질을 왕창 확대한 것처럼 도트가 깨진다.
    
    
    Constant Buffer를 통해 projection 행렬은 변경된 창 해상도로 만들어 넘겨줬지만
    Back Buffer 크기와 viewport의 크기는 재설정해주지 않아서 생기는 문제 같다.
    
    
    일단은 Constant Buffer를 통해 projection 행렬은 변경된 창 해상도로 만들어 넘겨주는 부분부터.
    
    ```
    
    
    	//Constant Buffer 생성
    	D3D11_BUFFER_DESC cb_on_resize_desc;
    	ZeroMemory(&cb_on_resize_desc, sizeof(cb_on_resize_desc));
    	cb_on_resize_desc.Usage = D3D11_USAGE_DYNAMIC;
    	cb_on_resize_desc.ByteWidth = ((sizeof(cb_on_resize_desc) + 15) / 16) * 16;
    	cb_on_resize_desc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
    	cb_on_resize_desc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
    	cb_on_resize_desc.MiscFlags = 0;
    	cb_on_resize_desc.StructureByteStride = 0;
    	if (FAILED(p_d3d_device_->CreateBuffer(&cb_on_resize_desc, NULL, &p_const_buffer_on_resize_)))
    	{
    		return E_FAIL;
    	}
    ```
    ```
    struct ChangeOnResizeBuffer {
    	XMMATRIX projection;
    };
    ```
    resize 때마다 갱신해줄 Constant Buffer다.
    
    Shader 파일에서도 
    ```
    cbuffer ChangeOnResizeBuffer {
    	matrix projection;
    };
    ```
    의 형태로 정의되어 있다.
    
    
    
    
    그럼 이제 resize가 일어났는지를 체크하는 함수가 필요하다.
    WM_SIZE 메시지는 윈도우의 크기가 변할 때마다 발생하는 메시지이다.
    
    
    ```
    LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
    {
    
        switch (message)
        {
        case WM_COMMAND:
        {
                //...
        }
            break;=
        case WM_SIZE:
        {
            Core::GetInstance()->SyncResolution();
        }
        	break;
        case WM_DESTROY:
            PostQuitMessage(0);
            break;
        default:
            return DefWindowProc(hWnd, message, wParam, lParam);
        }
        return 0;
    }
    ```
    WM_SIZE가 발생했을 때 Core::SyncResolution()을 실행해준다.
    ```
    void Core::SyncResolution()
    {
    	//해상도값과 실제 해상도값 비교(작업영역)
    	RECT window_rect;
    	ZeroMemory(&window_rect, sizeof(window_rect));
    	GetWindowRect(hwnd_, &window_rect);
    	RECT resolution_rect{ 0, 0, pt_resolution_.x, pt_resolution_.y };
    	AdjustWindowRect(&resolution_rect, WS_OVERLAPPEDWINDOW, TRUE);
    
    	Vector2 old_window_size( resolution_rect.right-resolution_rect.left, resolution_rect.bottom-resolution_rect.top );
    	Vector2 new_window_size(window_rect.right - window_rect.left, window_rect.bottom - window_rect.top);
    	Vector2 new_resolution(new_window_size-(old_window_size - pt_resolution_));
    	if (new_resolution.x > 0 && new_resolution.y > 0) {
    		LONG width_diff = new_window_size.x - old_window_size.x;
    		LONG height_diff = new_window_size.y - old_window_size.y;
    		pt_resolution_.x = new_resolution.x;
    		pt_resolution_.y = new_resolution.y;
    
    
    		//파라미터 버퍼에 해상도 입력
    		DXClass::GetInstance()->ResetResolution(Vector2(pt_resolution_));
    		DXClass::GetInstance()->WriteConstantBufferOnResize(pt_resolution_);
    
    	}
    
    }
    ```
    AdjustWindowRect() 함수는 인자로 작업영역의 해상도를 받아, 윈도우의 테두리나 메뉴 크기까지 계산해서 더해진 상태의 해상도로 만들어준다.
    
    가령, 게임에서 1920x1080 해상도를 사용한다는 것은 게임화면이 1920x1080이지 창 테두리까지 포함해서 1920x1080를 사용하려는 건 당연히 아닐 것이다. 따라서 윈도우 프로그램은 "창의 크기"와 "작업영역의 크기"를 따로 다룬다.
    
    pt_resolution_은 멤버변수로, 작업영역의 크기를 저장해놨던 변수다. 
    구조체에 pt_resolution_을 셋팅하고 AdjustWindowRect를 실행하면 resolution_rect에 전체 윈도우 크기가 저장된다.
    
    new_resolution이 0이라는 건 변경된 윈도우와 현재 윈도우의 크기가 똑같다는 뜻이고 
    new_resolution의 값이 하나라도 음수가 되는 건 윈도우가 최소화될 때이다.
    이 경우는 해상도 동기화를 수행하지 않는다.
    
    위의 두 사례에 해당되지 않아 해상도 동기화를 수행하는 경우, 즉 해상도가 변경된 게 확실할 경우, 멤버변수 pt_resolution_(작업영역 크기)를 변경해줘야 하는데, window_rect는 작업영역+@인 윈도우 크기이므로 이 값을 바로 넣으면 안 되고,
    작업영역->윈도우 크기로 변환할 때 증가한 값 만큼을 구해서 이 값을 뺀 다음 적용해준다.
    
    그리고 마지막 두 줄은 Resolution을 재설정하고 아까 만든 Constant Buffer를 설정하는 코드다.
    Reset Resolution 함수는 다음과 같다.
    
    ```
    void DXClass::ResetResolution(Vector2 new_resolution)
    {
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
    }
    ```
    
    기존의 Render Target View를 Release하고
    Swap Chain의 버퍼 크기를 새로 셋팅한 다음(Front 및 Back Buffer의 크기), Back Buffer를 받아와 새 Render Target View를 만든다.
    Back Buffer는 Release 해준다.
    
    (Back Buffer는 이후로도 계속 사용하는데 왜 Release하는지를 찾아봤으나 나오지 않는다... Get과 Release가 쌍으로 동작하는 것 같다. 참조를 받아오고/참조를 해제하고 의 개념으로 생각된다. 버퍼를 해제하는 게 아니고 버퍼의 참조를 해제하는 개념인듯.)
    
    
    
    Viewport는 그릴 영역을 의미한다. 1920x1080이지만 그 안에서 특정 사각형만 그릴 거라면 해상도보다 작게 설정하면 된다는데,
    한 번 테스트해봤더니 해당 해상도를 벗어나면 캐릭터가 사라지더라.
    그리고 좌표계산에 뭔가 오류가 생기는지 클릭 인식은 바뀐 해상도로 되지만 그려지는 해상도는 viewport 해상도로 적용돼서,
    ui를 드래그하면 마우스 좌표랑 점점 어긋나고, 놓여져있는 위치와 클릭해서 클릭되는 위치가 다르다.
    
    아무튼 해상도를 변경할 땐 1.좌표계산을 위한 projection 행렬, 2.swap chain의 buffer 크기, 3. viewport 크기 셋다 설정해줘야 한다는 것.

