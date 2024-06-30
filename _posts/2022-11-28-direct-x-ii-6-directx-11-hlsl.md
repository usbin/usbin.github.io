--- layout: post title: 'Direct X로 간단한 게임 만들기 - II. 사전지식 학습 : 3. DirectX(4) - HLSL의 압축 규칙' date: '2022-11-28T03:59:00.006-08:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2022-12-27T02:53:24.580-08:00' thumbnail: https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEitMq7B0qiNgQsL9M62jaywcG1f\_2RCwlKIOW2hQ4iKoJojkSCUyj1sbppq-LRb9DxITRJ9p23RjOnJGT4epWjY9dHxqymU0ca7CwhXzmoTPyjhRJTx2yOvlEvJrp3OUGQX295DAn-9KYkI4xNf2Z68YF75OQAoKI1ZIU-YZ2R-K3UYttWJ2JM8MPQ-Fg/s72-w640-c-h272/Untitled-20.jpg blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-6766854769061050235 blogger\_orig\_url: https://sealbinary.blogspot.com/2022/11/direct-x-ii-6-directx-11-hlsl.html ---

    
      
    
    
    Constant Buffer에 값이 자꾸 이상하게 들어가서 며칠동안 삽질을 했다.
    
    알고보니 HLSL은 구조체 데이터를 16byte 단위로 압축한다고 한다. 따라서 struct도 16byte 단위로 맞춰줘야 한다.
    
    > HLSL은 VS 출력 데이터, GS 입력 및 출력 데이터, PS 입력 및 출력 데이터에 대한 압축 규칙을 구현합니다. (IA 단계에서는 데이터 압축을 풀 수 없으므로 VS 입력에 대한 데이터가 압축되지 않습니다.)
    > [[출처: MS 문서 참고]](https://learn.microsoft.com/ko-kr/windows/win32/direct3dhlsl/dx-graphics-hlsl-packing-rules)
    
    Vertex Buffer와 Index Buffer는 IA 단계에서 설정하기 때문에 압축되지 않는다. 이 둘은 16byte 단위로 맞춰주지 않았는데도 잘 돌아갔던 이유가 이것이었다.
    
    Constant Buffer는 ChangeOnResize는 VS 단계에서, ChangeOnRender는 PS 단계에 바인드를 해놨으므로 16byte 압축이 자동으로 되는 것이다.
    문서에는 4개 벡터라고 쓰여있는데, 4개 벡터란 float4나 int4를 의미하므로 4x4byte=16byte 압축이다.
    
    ChangeOnRenderBuffer와 이에 호환시켜 쓰는 HLSL의 구조체는 이렇게 생겼다.
    ```
    //C++ 구조체
    struct ChangeOnRenderBuffer {
    	BOOL use_texture; //BOOL은 재정의된 int로, 4byte임.
    	XMFLOAT4 mesh_color; //float 4개, 즉 16byte임.
    };
    //HLSL Shader 구조체
    cbuffer ChangeOnRenderBuffer
    {
        int use_texture;
        float4 mesh_color : SV_TARGET;
    };
    ```
    이 HLSL 구조체는 16byte 단위로 압축이 되어 내부적으로는
    
    int(4byte), padding(12byte), float4(16byte)
    
    이렇게 되어있다.
    
    이 상태에서 C++의 struct가 들어간다면 어떻게 될까.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEitMq7B0qiNgQsL9M62jaywcG1f_2RCwlKIOW2hQ4iKoJojkSCUyj1sbppq-LRb9DxITRJ9p23RjOnJGT4epWjY9dHxqymU0ca7CwhXzmoTPyjhRJTx2yOvlEvJrp3OUGQX295DAn-9KYkI4xNf2Z68YF75OQAoKI1ZIU-YZ2R-K3UYttWJ2JM8MPQ-Fg/w640-h272/Untitled-20.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEitMq7B0qiNgQsL9M62jaywcG1f_2RCwlKIOW2hQ4iKoJojkSCUyj1sbppq-LRb9DxITRJ9p23RjOnJGT4epWjY9dHxqymU0ca7CwhXzmoTPyjhRJTx2yOvlEvJrp3OUGQX295DAn-9KYkI4xNf2Z68YF75OQAoKI1ZIU-YZ2R-K3UYttWJ2JM8MPQ-Fg/s2163/Untitled-20.jpg)
  
  

    BOOL 4byte는 그대로 int 4byte에 들어가고,
    XMFLOAT4의 앞부분 12byte는 패딩에 들어가서 사라지고,
    XMFLOAT4의 뒷부분 4byte는 mesh_color에 들어간다.
    
    XMFLOAT4는 (R, G, B, A)로 구성되어 있는데 R,G,B 정보는 사라지고, A 정보가 맨 앞인 R 자리로 들어간 (A, 0, 0, 0)이 mesh_color에 들어가는 것이다.
    이것을 테스트해보면 padding을 명시적으로
    
    ```
    struct ChangeOnRenderBuffer {
    	BOOL use_texture; 
        int padding;
    	XMFLOAT4 mesh_color; 
    };
    ```
    4byte만 넣어줬을 경우, mesh_color는 한 칸 밀려서 (B, A, 0, 0)이 들어가는 걸 확인할 수 있다.
    
    
    ### **[해결방법 1]**
    
    16byte 크기에 맞추려면 3byte 패딩을 넣어주면 된다.
    ```
    struct ChangeOnRenderBuffer {
    	BOOL use_texture; 
        XMINT3 padding;
    	XMFLOAT4 mesh_color; 
    };
    ```
    
    ### **[해결방법 2]**
    
    혹은 순서를 바꾸는 것으로도 해결 가능하다면 순서만 바꿔줘도 된다.
    
    이 경우 element가 두 개 뿐이고 하나가 이미 16byte를 채우고 있으므로 두 개의 순서만 바꿔줘도 해결된다.
    
    물론 이 경우 HLSL 구조체의 순서도 바꿔줘야 한다.
    ```
    struct ChangeOnRenderBuffer {
    	XMFLOAT4 mesh_color; 
    	BOOL use_texture; 
    };
    ```
    
    ```
    cbuffer ChangeOnRenderBuffer
    {
        float4 mesh_color : SV_TARGET;
        int use_texture;
    };
    ```

