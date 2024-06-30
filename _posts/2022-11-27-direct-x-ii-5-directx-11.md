--- layout: post title: 'Direct X로 간단한 게임 만들기 - II. 사전지식 학습 : 3. DirectX(3) - DX 11의 버퍼 종류: DEFAULT, DYNAMIC' date: '2022-11-27T04:43:00.008-08:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2022-12-27T02:52:53.715-08:00' blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-4650309144936801318 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/11/direct-x-ii-5-directx-11.html ---

    # 1. DEFAULT 버퍼와 DYNAMIC 버퍼
    ```
    	D3D11_BUFFER_DESC cb_on_resize_desc;
    	ZeroMemory(&cb_on_resize_desc, sizeof(cb_on_resize_desc));
    	cb_on_resize_desc.Usage = D3D11_USAGE_DEFAULT;
    	cb_on_resize_desc.ByteWidth = ((sizeof(cb_on_resize_desc)+15)/16)*16;
    	cb_on_resize_desc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
    	cb_on_resize_desc.CPUAccessFlags = 0;
    	cb_on_resize_desc.MiscFlags = 0;
    	cb_on_resize_desc.StructureByteStride = 0;
    ```
    DEFAULT 버퍼는 이런 식으로, 반드시 Usage는 D3D11_USAGE_DEFAULT로 설정하고 CPUAccessFlags는 0으로 설정해야 한다.
    
    
    ```
    	D3D11_BUFFER_DESC cb_on_resize_desc;
    	ZeroMemory(&cb_on_resize_desc, sizeof(cb_on_resize_desc));
    	cb_on_resize_desc.Usage = D3D11_USAGE_DYNAMIC;
    	cb_on_resize_desc.ByteWidth = ((sizeof(cb_on_resize_desc)+15)/16)*16;
    	cb_on_resize_desc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
    	cb_on_resize_desc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
    	cb_on_resize_desc.MiscFlags = 0;
    	cb_on_resize_desc.StructureByteStride = 0;
    ```
    DYNAMIC 버퍼는 반드시 Usage는 D3D11_USAGE_DYNAMIC으로, CPUAccessFlags는 D3D11_CPU_ACCESS_WRITE로 설정해야 한다.
    
    그리고 CONSTANT 버퍼의 경우 16의 배수로 크기를 할당해야 한다.
    마찬가지로 struct도 16byte로 정렬되어 있어야 한다.
    순서를 바꿔서 맞추거나 임의의 padding 요소를 넣어서 맞춰줘야 한다.
    
    
    # 2. GPU에 메모리를 쓰는 두가지 방법
    
    쉐이더에 정점을 전달하기 위해서는 Vertex Buffer,
    인덱스를 전달하기 위해서는 Index Buffer,
    파라미터를 전달하기 위해서는 Constant Buffer를 사용한다.
    
    버퍼를 생성할 땐 Buffer Desc의 Usage 항목을 설정하는데,
    주로 사용하는 것으로 Default, Dynamic 이 있다.
    Default는 버퍼에 값을 변경할 일이 없을 때 쓰고 Dynamic은 값을 변경해야 할 때 쓴다고 하는데...
    Default로 만들어도 값을 변경할 수 있더라.
    
    Default 버퍼는 UpdateSubresource() 함수로 값을 변경한다. 이 경우 바로 GPU로 값을 쓰는 것이기 때문에 버퍼의 값을 읽을 수는 없다.
    Dynamic 버퍼는 Map()과 Unmap() 함수로 값을 변경한다. 이 경우 GPU의 주소를 시스템 메모리에 맵핑한 다음 시스템 메모리상 주소에 접근하는 것이므로, 버퍼의 값을 읽고 쓸 수 있다.
    
    
    ### 1. UpdateSubresource()
    ```c++
    ConstantBufferData cb_data;
    cb_data.color = my_color;
    immediate_context-&gt;UpdateSubresource(p_cb, 0, 0, &amp;cb_data, 0, 0);
    ```
    - 데이터를 쓸 때에만 사용한다.
    - p_cb(ID3D11Buffer*)의 타입이 Default로 생성되었어야 한다.
    
    
    ### 2. Map(), Unmap()
    ```c++
    D3D11_MAPPED_RESOURCE mapped_resource;
    immediate_context->Map(p_cb, 0, D3D11_MAP_WRITE_DISCARD, 0, &mapped_resource);
    ConstantBufferData* p_cb_data = (ConstantBufferData*)mapped_resource.pData;
    p_cb_data->color = my_color;
    immediate_constext->Unmap(p_cb, 0);
    ```
    
    - 데이터를 읽거나 쓸 수 있다.
    - p_cb의 타입이 Dynamic으로 생성되었어야 한다.
    - read를 시도할 경우 시스템 메모리로 값을 복사하는 오버헤드가 발생한다.

Constant Buffer의 경우 주로 Default로 생성해서 UpdateSubresource()를 사용한다고 한다.
