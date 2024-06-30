--- layout: post title: 'Direct X로 간단한 게임 만들기 - II. 사전지식 학습 : 3. DirectX(5) - DX11의 이미지 렌더링' date: '2022-11-30T05:38:00.006-08:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2022-12-27T02:54:17.951-08:00' blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-2886848277195114843 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/11/direct-x-ii-7-directx-11.html ---

    # **1. DX11에서의 이미지 렌더링**
    
    DX에서 이미지 렌더링은 이미지를 가상의 공간에 그리고, 이 가상의 공간을 모니터 속 특정 구역 안에 맵핑시켜 그리는 순서로 일어난다.
    
    ### **1. 파일 이미지를 그리는 경우**
    
    이미 존재하는 파일 이미지를 불러와서 그리는 경우이다.
    
    먼저 렌더링 파이프라인에 Sampler를 등록해줘야 한다. Sampler는 모니터 속 공간과 그리려는 이미지의 픽셀을 맵핑하는 역할을 한다.```
    	D3D11_SAMPLER_DESC sample_desc;
    	ZeroMemory(&sample_desc, sizeof(sample_desc));
    	sample_desc.Filter = D3D11_FILTER_MIN_MAG_MIP_LINEAR;
    	sample_desc.AddressU = D3D11_TEXTURE_ADDRESS_WRAP;
    	sample_desc.AddressV = D3D11_TEXTURE_ADDRESS_WRAP;
    	sample_desc.AddressW = D3D11_TEXTURE_ADDRESS_WRAP;
    	sample_desc.ComparisonFunc = D3D11_COMPARISON_NEVER;
    	sample_desc.MinLOD = 0;
    	sample_desc.MaxLOD = D3D11_FLOAT32_MAX;
    	if (FAILED(p_d3d_device_->CreateSamplerState(&sample_desc, &p_sampler_linear_))) {
    		return E_FAIL;
    	};
    	p_immediate_context_->PSSetSamplers(0, 1, &p_sampler_linear_);
        
    ```
    
    이 Sampler는 나중에 Shader의 글로벌 변수 **SamplerState samLinear** 에 들어간다.
    
    
    그리고 블렌딩 설정을 해준다. 블렌딩 설정은 알파값을 적용시키기 위해 하는데, 투명한 픽셀일 경우 어떻게 처리할지를 설정한다.
    
    
    ```
    	ID3D11BlendState* p_blend_state;
    	D3D11_BLEND_DESC blendStateDesc{};
    	blendStateDesc.AlphaToCoverageEnable = FALSE;
    	blendStateDesc.IndependentBlendEnable = FALSE;
    	blendStateDesc.RenderTarget[0].BlendEnable = TRUE;
    	blendStateDesc.RenderTarget[0].SrcBlend = D3D11_BLEND_SRC_ALPHA;
    	blendStateDesc.RenderTarget[0].DestBlend = D3D11_BLEND_INV_SRC_ALPHA;
    	blendStateDesc.RenderTarget[0].BlendOp = D3D11_BLEND_OP_ADD;
    	blendStateDesc.RenderTarget[0].SrcBlendAlpha = D3D11_BLEND_SRC_ALPHA;
    	blendStateDesc.RenderTarget[0].DestBlendAlpha = D3D11_BLEND_DEST_ALPHA;
    	blendStateDesc.RenderTarget[0].BlendOpAlpha = D3D11_BLEND_OP_ADD;
    	blendStateDesc.RenderTarget[0].RenderTargetWriteMask = D3D11_COLOR_WRITE_ENABLE_ALL;
    
    	if (FAILED(p_d3d_device_->CreateBlendState(&blendStateDesc, &p_blend_state)))
    	{
    		return E_FAIL;
    	}
    
    	p_immediate_context_->OMSetBlendState(p_blend_state, nullptr, 0xFFFFFFFF);
    ```
    
    SrcBlend와 DestBlend, BlendOp 설정은 알파블렌딩을 하게 한다.
    우리가 흔히 아는 R, G, B, A 네 개의 값으로 투명한 효과를 주는 블렌딩이 이 알파블렌딩이다.
    
    ||결과 픽셀 = ( Src * SrcBlend ) BlendOp ( Dest * DestBlend )||
    |---|---|---|
    
    이 연산을 거쳐서 결과 픽셀을 만들어낸다고 한다.
    여기서는 SrcBlend가 Src의 알파값으로 설정했고, DestBlend는 Inverse Src의 알파값으로 설정했으므로
    Src는 본인의 투명도만큼 값이 줄어들고 Dest는 1-Src의 투명도 값만큼 값이 줄어들어 적용이 된다.
    OP는 ADD이므로 최종적으로 픽셀 값이 더해진다. 파란색 앞에 노란색 픽셀이 온다면 더해져서 초록색으로 보일 것이다.
    
    
    
    그리고 SrcBlendAlpha, DestBlendAlpha, BlendOpAlpha는 알파값을 계산할 때 쓰인다.
    
    ||결과 알파 = (SrcAlpha * SrcBlendAlpha) BlendOpAlpha (DestAlpha * DestBlendAlpha)||
    |---|---|---|
    
    여기서 SrcBlendAlpha는 Src의 알파값이고, DestBlendAlpha는 Dest의 알파값이므로,
    Src와 Dest의 알파값이 더해진 값이 최종 알파값이 된다.
    
    이 과정은 여러 개의 샘플러를 사용할 게 아니라면 초기화 과정에서 한 번만 셋팅해도 된다.
    
    
    ```
    struct CustomVertex {
    	XMFLOAT3 position;
    	XMFLOAT2 texture_position;
    };
    
    struct ChangeOnRenderBuffer {
    	XMFLOAT4 mesh_color;
    	BOOL use_texture;
    };
    ```
    
    텍스쳐를 사용하려면 텍스쳐 좌표가 필요하다.
    CustomVertex에 원래는 position과 color값이 있었는데, color를 빼고 texture_position을 넣어준다.
    그리고 ChagneOnRenderBuffer라는 구조체를 새로 정의해 use_texture와 color값을 지정하고 Constant Buffer로 사용해준다.
    
    앞으로 렌더링을 할 때 CustomVertex에 좌표와 텍스쳐 좌표를 넣고,
    그림만 그릴 땐 use_texture를 FALSE로, color를 지정해주고
    텍스쳐를 그릴 땐 use_texture를 TRUE로, color는 사용하지 않는다.
    
    
    
    그리고 초기화 구문에서도 Input Layout 부분에서 TEXCOORD라는 쉐이더용 변수를 새로 선언해주고 COLOR는 지워준다.
    
    ```
    	// Input Layout(Vertex Layout) 생성
    	ID3D11InputLayout* p_input_layout;
    	D3D11_INPUT_ELEMENT_DESC layout[] =
    	{
    		{ "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    		{ "TEXCOORD", 0, DXGI_FORMAT_R32G32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 }
    	};
    	UINT num_elements = ARRAYSIZE(layout);
    
    	if (FAILED(p_d3d_device_->CreateInputLayout(layout, num_elements, p_vs_blob->GetBufferPointer(), p_vs_blob->GetBufferSize(), &p_input_layout)))
    	{
    		p_vs_blob->Release();
    		return E_FAIL;
    	}
    	p_vs_blob->Release();
    	p_immediate_context_->IASetInputLayout(p_input_layout);
    ```
    
    
    아래는 Constant Buffer의 생성 코드이다.
    
    ```
    	D3D11_BUFFER_DESC cb_on_render_desc;
    	ZeroMemory(&cb_on_render_desc, sizeof(cb_on_render_desc));
    	cb_on_render_desc.Usage = D3D11_USAGE_DYNAMIC;
    	cb_on_render_desc.ByteWidth = ((sizeof(cb_on_render_desc) + 15) / 16) * 16;
    	cb_on_render_desc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
    	cb_on_render_desc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
    	cb_on_render_desc.MiscFlags = 0;
    	cb_on_render_desc.StructureByteStride = 0;
    	if (FAILED(p_d3d_device_->CreateBuffer(&cb_on_render_desc, NULL, &p_const_buffer_on_render_)))
    	{
    		return E_FAIL;
    	}
    
    	p_immediate_context_->PSSetConstantBuffers(0, 1, &p_const_buffer_on_render_);
    ```
    
    
    
    이제 쉐이더는 DrawTexture 함수에서 설정한 Shader Resource View와 초기화할 때 설정한 Sampler를 가지고 샘플링을 통해 입력된 좌표 위치에 맞는 픽셀을 매칭한다.
    
    ```
    //SimpleShader.hlsl 파일
    
    
    //--------------------------------------------------------------------------------------
    // GLOBAL
    //--------------------------------------------------------------------------------------
    SamplerState samLinear;
    Texture2D txDiffuse;
    cbuffer ChangeOnResizeBuffer
    {
        matrix projection;
    };
    cbuffer ChangeOnRenderBuffer
    {
        float4 mesh_color : SV_TARGET;
        int use_texture;
    };
    //--------------------------------------------------------------------------------------
    // Vertex Input Type
    //--------------------------------------------------------------------------------------
    struct VertexInputType
    {
        float4 position : POSITION;
        float2 texture_position: TEXCOORD;
    };
    
    //--------------------------------------------------------------------------------------
    // Pixel Input Type
    //--------------------------------------------------------------------------------------
    struct PixelInputType
    {
        float4 position : SV_POSITION;
        float2 texture_position: TEXCOORD;
    };
    
    
    //--------------------------------------------------------------------------------------
    // Vertex Shader
    //--------------------------------------------------------------------------------------
    PixelInputType VS( VertexInputType input )
    {
        PixelInputType output;
        output.position = mul(input.position,projection);
        output.position.x = (-1.0 + output.position.x); //(-1, 1)의 x축을 (0, 1)로 변경
        output.position.y = (1.0 - output.position.y); //(1, -1)의 y축을 (0, 1)로 변경(x축 반전)
        output.texture_position = input.texture_position;
        return output;
    }
    
    
    //--------------------------------------------------------------------------------------
    // Pixel Shader
    //--------------------------------------------------------------------------------------
    float4 PS(PixelInputType input) : SV_Target
    {
        if(use_texture){
            return txDiffuse.Sample(samLinear, input.texture_position);
        }
        else{
            return mesh_color;
        }
    }
    
    ```
    
    
    다음으로는 파일 이미지를 읽은 다음 Texture로 만들고, 이 Texture를 다시 Shader Resource View로 만든다.
    
    여기서 Shader Resource View는 "그리려는 리소스"를 담는 틀 같은 것이다. Shader Resource View로 만들어야만 화면상에 그릴 수 있다.
    (파일 이미지) -> Texture -> Shader Resource View 순서로 생성해야 하지만 그냥 (파일 이미지)에서 바로 Shader Resource View를 한 번에 생성하는 함수가 있더라.
    
    Shader Resource View에는 1D, 2D, 3D 다양한 차원의 텍스쳐가 들어갈 수 있는데 보통은 2D 텍스쳐가 쓰인다고 생각하면 될 것 같다.
    굳이 텍스쳐가 아니라도 리소스의 형태로 들어갈 수 있는 것들은 다 들어가는 듯하다.
    
    다만 Shader Resource View는 왼쪽 위가 (0, 0)이고 오른쪽 아래가 (1,1)인 좌표계를 쓰기 때문에 이에 맞추어 좌표를 변환해줘야 한다.
    
    가령 특정 텍스쳐의 (x, y)부터 (width, height)만큼의 이미지를 쓰려고 한다면, 이 값들을 원본의 가로 세로로 나눠주면 된다.
    
    
    ```
    	ID3D11ShaderResourceView *p_resource_view_;//멤버변수
        
    	if (FAILED(D3DX11CreateShaderResourceViewFromFile(DXClass::GetInstance()->get_d3d_device(), texture_path.c_str(), NULL, NULL, &p_resource_view_, NULL))) {
    		return;
    	}
        
        ID3D11Texture2DDesc texture_desc_; //멤버변수
        
        ID3D11Texture2D* p_texture_interface;
    	ID3D11Resource* res;
    	p_resource_view_->GetResource(&res);
    	res->QueryInterface<ID3D11Texture2D>(&p_texture_interface);
    	p_texture_interface->GetDesc(&texture_desc_);
    ```
    texture_desc_는 텍스쳐의 가로 세로 값을 가져오기 위해 필요하다.
    
    
    
    ```
    	inline UINT get_width() { return texture_desc_.Width; };
    	inline UINT get_height() { return texture_desc_.Height; };
    	inline const Vector2& get_size() { return Vector2{ get_width(), get_height() }; };
    ```
    자주 사용하게 될 테니 함수로 만들어놓자.
    
    아무튼 이제 이걸 Pixel Shader에 넣어주고, Pixel Shader는 샘플링 함수를 수행하도록 변경해준다.
    
    ```
    void DrawTexture(ID3D11Device* p_d3d_device, const Vector2& base_pos, const Vector2& scale, const Vector2& texture_base_pos, const Vector2& texture_scale, Texture* texture)
    {
    	ID3D11DeviceContext* p_immediate_context;
    	p_d3d_device->GetImmediateContext(&p_immediate_context);
    
    	ID3D11ShaderResourceView* p_resource_view = texture->get_resource_view();
    	Vector2 r = texture->get_size();
    	Vector2 normalized_texture_base_pos = texture_base_pos / r;
    	Vector2 normalized_texture_scale = texture_scale / r;
    		
    	//점 좌표
    	const int vertice_count = 4;
    	CustomVertex vertices[] =
    	{
    		{ XMFLOAT3(base_pos.x, base_pos.y, 0.f), XMFLOAT2(normalized_texture_base_pos.x, normalized_texture_base_pos.y)},
    		{ XMFLOAT3(base_pos.x + scale.x,	base_pos.y, 0.f), XMFLOAT2(normalized_texture_base_pos.x + normalized_texture_scale.x, normalized_texture_base_pos.y)},
    		{ XMFLOAT3(base_pos.x + scale.x,	base_pos.y + scale.y,	0.f), XMFLOAT2(normalized_texture_base_pos.x + normalized_texture_scale.x, normalized_texture_base_pos.y + normalized_texture_scale.y)},
    		{ XMFLOAT3(base_pos.x, base_pos.y + scale.y,	0.f), XMFLOAT2(normalized_texture_base_pos.x, normalized_texture_base_pos.y + normalized_texture_scale.y)}
    	};
    	//면 인덱스
    	const int plane_indices_count = 6;
    	UINT plane_indices[] = {
    		0, 1, 2, 
    		
    		2, 3, 0
    	};
    	//Vertex Buffer 채우기
    	DXClass::GetInstance()->WriteVertexBuffer(vertices, vertice_count);
    	//Plane Index Buffer 채우기
    	DXClass::GetInstance()->WriteIndexBuffer(plane_indices, plane_indices_count);
    	//Constant Buffer 채우기
    	DXClass::GetInstance()->WriteConstantBufferOnRender(TRUE, XMFLOAT4(0, 0, 0, 0));
    
    	p_immediate_context->PSSetShaderResources(0, 1, &p_resource_view);
    
    	// Set primitive topology
    	p_immediate_context->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
    	p_immediate_context->DrawIndexed(plane_indices_count, 0, 0);
    }
    
    
    void DXClass::WriteVertexBuffer(CustomVertex* vertices, UINT vertex_count)
    {
    	ID3D11Buffer* p_vertex_buffer;
    	UINT stride = sizeof(CustomVertex);
    	UINT offset = 0;
    	p_immediate_context_->IAGetVertexBuffers(0, 1, &p_vertex_buffer, &stride, &offset);
    
    	D3D11_MAPPED_SUBRESOURCE mapped_resource;
    	if (FAILED(p_immediate_context_->Map(p_vertex_buffer, 0, D3D11_MAP_WRITE_DISCARD, 0, &mapped_resource)))
    	{
    		return;
    	}
    	CustomVertex* vertex_data = (CustomVertex*)mapped_resource.pData;
    	for (int i = 0; i < vertex_count; i++) {
    		vertex_data[i] = vertices[i];
    	}
    	p_immediate_context_->Unmap(p_vertex_buffer, 0);
    }
    
    void DXClass::WriteIndexBuffer(UINT* indices, UINT index_count)
    {
    	ID3D11Buffer* p_idx_buffer;
    	DXGI_FORMAT idx_format = idx_format_;
    	p_immediate_context_->IAGetIndexBuffer(&p_idx_buffer, &idx_format, 0);
    	D3D11_MAPPED_SUBRESOURCE idx_mapped_resource;
    
    	if (p_idx_buffer == NULL || FAILED(p_immediate_context_->Map(p_idx_buffer, 0, D3D11_MAP_WRITE_DISCARD, 0, &idx_mapped_resource)))
    	{
    		return;
    	}
    	UINT* idx_data = (UINT*)idx_mapped_resource.pData;
    	for (int i = 0; i < index_count; i++) {
    		idx_data[i] = indices[i];
    	}
    	p_immediate_context_->Unmap(p_idx_buffer, 0);
    }
    void DXClass::WriteConstantBufferOnResize(Vector2 resolution)
    {
    	XMMATRIX projection = XMMatrixOrthographicLH(resolution.x, resolution.y, 0, 1);
    	D3D11_MAPPED_SUBRESOURCE mapped_resource;
    	if (FAILED(p_immediate_context_->Map(p_const_buffer_on_resize_, 0, D3D11_MAP_WRITE_DISCARD, 0, &mapped_resource)))
    	{
    		return;
    	}
    	ChangeOnResizeBuffer* p_vs_const_data;
    	p_vs_const_data = (ChangeOnResizeBuffer*)mapped_resource.pData;
    	p_vs_const_data->projection = projection;
    	p_immediate_context_->Unmap(p_const_buffer_on_resize_, NULL);
    }
    
    void DXClass::WriteConstantBufferOnRender(BOOL use_texture, XMFLOAT4 mesh_color)
    {
    	D3D11_MAPPED_SUBRESOURCE mapped_resource;
    	if (FAILED(p_immediate_context_->Map(p_const_buffer_on_render_, 0, D3D11_MAP_WRITE_DISCARD, 0, &mapped_resource)))
    	{
    		return;
    	}
    	ChangeOnRenderBuffer* p_const_data;
    	p_const_data = (ChangeOnRenderBuffer*)mapped_resource.pData;
    	p_const_data->use_texture = use_texture;
    	p_const_data->mesh_color = mesh_color;
    	p_immediate_context_->Unmap(p_const_buffer_on_render_, NULL);
    }
    
    ```
    
    삼각형을 그리는 함수와 유사한데, CustomVertex에 텍스쳐 좌표가 추가되었고, Constant Buffer를 채우고 ShaderResourceView를 PS 단계에 바인딩한다는 점만 다르다.
    
    ### **2. 이미지를 직접 만드는 경우**
    
    만약 .png, .bmp 등의 이미 존재하는 이미지로부터 Texture를 읽어들이는 게 아니라 직접 만들려 한다면,
    가령 게임에서 미니맵을 그리는 경우가 있을 것이다.
    
    이 경우는 일단 쓸 일이 없으므로 이론적으로만 정리해놓을 것이다.
    
    캐릭터의 머리 위에서 아래를 내려다보는 뷰를 미니맵으로 만든다고 생각해보자.
    
    이 미니맵을 그릴 장소로 먼저 Render Target View를 만들어야 한다.
    (Shader Resource View가 "그리려는 리소스"였다면, Render Target View는 "리소스를 그리려는 공간"이다.)
    
    Render Target View를 만들었다면 이제 Depth Stencil View를 만들어야 한다.
    
    Depth Stencil View란 깊이, 스텐실 정보를 기록하기 위한 공간이다. 깊이는 말 그대로 그려지는 순서를 위한 깊이 값이고, 스텐실은 그리지 않아도 되는 공간을 제외하는 역할을 한다. 미니맵의 경우엔 미니맵을 재렌더링 할 때 미니맵을 제외한 나머지 화면은 다시 덮어쓸 필요가 없으므로 이 정보가 해당된다.
    
    Depth Stencil View를 만드는 코드는 다음과 같다.
    
    
    ```
    	ID3D11DepthStencilView* p_depth_stencil_view;
    	ID3D11Texture2D* p_depth_stencil_buffer;
    	D3D11_TEXTURE2D_DESC depth_stencil_desc;
    
    	depth_stencil_desc.Width = Width;
    	depth_stencil_desc.Height = Height;
    	depth_stencil_desc.MipLevels = 1;
    	depth_stencil_desc.ArraySize = 1;
    	depth_stencil_desc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
    	depth_stencil_desc.SampleDesc.Count = 1;
    	depth_stencil_desc.SampleDesc.Quality = 0;
    	depth_stencil_desc.Usage = D3D11_USAGE_DEFAULT;
    	depth_stencil_desc.BindFlags = D3D11_BIND_DEPTH_STENCIL;
    	depth_stencil_desc.CPUAccessFlags = 0;
    	depth_stencil_desc.MiscFlags = 0;
    
    	//Create the Depth/Stencil View
    	p_d3d_device_->CreateTexture2D(&depth_stencil_desc, NULL, &p_depth_stencil_buffer);
    	p_d3d_device_->CreateDepthStencilView(p_depth_stencil_buffer, NULL, &p_depth_stencil_view);
    ```
    
    그럼 이제 Render Target과 Depth Stencil을 동시에 지정할 수 있다.
    
    ```
    	p_immediate_context_->OMSetRenderTargets(1, &p_render_target_view_, p_depth_stencil_view);
    ```
    
    이제 앞으로 렌더링 파이프라인의 결과물은 Render Target View와 Depth Stencil View에 쓰여지게 된다.
    
    카메라 위치를 플레이어의 머리 위 위치로 지정하고,
    view, projection 행렬을 거친 결과를 렌더링하면 미니맵 Render Target View에 이 결과가 그려진다.
    
    그 다음부터는 파일 이미지를 그릴 때와 같은 과정이다.
    미니맵이 그려져야 하는 공간의 좌표를 셋팅하고, Shader Resource View에 Render Target View의 텍스쳐를 넣어 생성한 다음, Shader에 전달해주면 된다.

