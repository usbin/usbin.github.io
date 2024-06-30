--- layout: post title: 'Direct X로 간단한 게임 만들기 - III. 게임 엔진 : Sound 출력하기(FMOD 라이브러리)' date: '2023-01-25T03:46:00.011-08:00' author: 가도 tags: - 포트폴리오\_DirectX modified\_time: '2023-01-25T05:11:45.818-08:00' thumbnail: https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg\_YTLDgAm3i1nYbbkc2DmFNt2X5EDDNWw1bWVEH-kqATd3csDFqDmYFCfYQVvo67fJlLfeWnm0Bh5p8TWsxUkeSzyI3jagOzJjpAfHfStAlIDvEVW4GM1O8pvFG9TiFTLnGd4l4geuBnfIbDY2lBDTl3yv5VTmIUWO043F1s0lVJ6Fi7YQ-WfW0gA7lg/s72-c/2.png blogger\_id: tag:blogger.com,1999:blog-778891705567453961.post-6833893159888479590 blogger\_orig\_url: https://sealbinary.blogspot.com/2023/01/direct-x-iii-sound-fmod.html ---

    win32 프로그램에서 사운드를 출력하는 방법은 두가지가 있다.
    
    1. 내장 라이브러리인 window.h에서 제공하는 playsound 함수를 사용하기.
    2. 외부 라이브러리 사용하기(ex. FMOD)
    
    1번의 방법은 제일 가볍고 쉬운 방법이지만 대신에 .mp3 파일을 지원하지 않고, 한 번에 1개의 사운드만 재생할 수 있다는 단점이 있다.
    게임 프로그램에서 이건 너무 치명적이다... 그냥 쓰지 말라는 거랑 같음.
    
    그래서 많이 쓰는 사운드 라이브러리인 FMOD를 사용해볼 것이다.
    
    인터넷을 뒤져보니 다행히 정보가 아주 많았다.
    
    FMOD 도큐먼트 : https://documentation.help/FMOD-Studio-API/getting_started1.html
    FMOD_MODE 설정값들 : https://documentation.help/FMOD-Studio-API/FMOD_MODE.html# 1. 비주얼 스튜디오 설정하기
    
    일단 (https://www.fmod.com/download#fmodengine)에서 FMOD Engine을 설치한 다음,
    비주얼 스튜디오의 프로그램 속성에서 설정을 해준다.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg_YTLDgAm3i1nYbbkc2DmFNt2X5EDDNWw1bWVEH-kqATd3csDFqDmYFCfYQVvo67fJlLfeWnm0Bh5p8TWsxUkeSzyI3jagOzJjpAfHfStAlIDvEVW4GM1O8pvFG9TiFTLnGd4l4geuBnfIbDY2lBDTl3yv5VTmIUWO043F1s0lVJ6Fi7YQ-WfW0gA7lg/s16000/2.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg_YTLDgAm3i1nYbbkc2DmFNt2X5EDDNWw1bWVEH-kqATd3csDFqDmYFCfYQVvo67fJlLfeWnm0Bh5p8TWsxUkeSzyI3jagOzJjpAfHfStAlIDvEVW4GM1O8pvFG9TiFTLnGd4l4geuBnfIbDY2lBDTl3yv5VTmIUWO043F1s0lVJ6Fi7YQ-WfW0gA7lg/s779/2.png)
  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjFcreB4FBPuDDx5U8z1j0JvC5DvFZ7BYFiSSQSCa5LU3SJsNrhY3W39-1tL9TpHRcc3nF9SZhjsjbpCxr7RO33dUQsb_JXszE5_RR_DEWxiP3jOL5onV63-ATFwjQn8PC3u25z2irg2WQYiEkXhTF7xuE9ikLhbYuNCTEBNcsHk9I-ZqpHGPWnKoIY_Q/s16000/1.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjFcreB4FBPuDDx5U8z1j0JvC5DvFZ7BYFiSSQSCa5LU3SJsNrhY3W39-1tL9TpHRcc3nF9SZhjsjbpCxr7RO33dUQsb_JXszE5_RR_DEWxiP3jOL5onV63-ATFwjQn8PC3u25z2irg2WQYiEkXhTF7xuE9ikLhbYuNCTEBNcsHk9I-ZqpHGPWnKoIY_Q/s779/1.png)
  

    포함 디렉토리에는 C:\Program Files (x86)\FMOD SoundSystem\FMOD Studio API Windows\api\core\inc 를,
    라이브러리 디렉토리에는 C:\Program Files (x86)\FMOD SoundSystem\FMOD Studio API Windows\api\core\lib\x64 를 추가한다.
    
    링커의 입력에도 fmod_vc.lib를 추가해준다.
    
    그리고 비주얼 스튜디오 실행파일(.exe)이 들어있는 폴더에 C:\Program Files (x86)\FMOD SoundSystem\FMOD Studio API Windows\api\core\lib\x64 경로에 있는 fmod.dll을 복사해 넣어준다.
    fmod.dll은 런타임에 실행파일과 같은 경로에 꼭 있어야 한다.
    
    (fmodL.dll, fmodL_vc.lib이라고, L이 붙은 파일이 하나씩 더 있는데 얘넨 디버깅 편하게 로깅을 위한 파일을 추가로 생성한다고 한다. 대신 파일 크기가 크다고 하니 그냥 L 안 붙은 파일을 사용했다.)
    
    
    그럼 사용할 준비가 끝났다.
    
    
    
    # 2. FMOD 라이브러리의 사용법
    
    ##### **(1) 초기화**
    ```
    #include <fmod.hpp>
    
    FMOD_SYSTEM* fmod_system_ = nullptr;
    FMOD_System_Create(&fmod_system_, FMOD_VERSION);// 성공시 FMOD_RESULT::FMOD_OK를 리턴
    FMOD_System_Init(fmod_system_, 32, FMOD_INIT_NORMAL, nullptr);// 성공시 FMOD_RESULT::FMOD_OK를 리턴
    
    ```
    
    ##### **(2) 사운드 생성**
    ```
    FMOD_MODE mode = 0;
    mode |= FMOD_LOOP_OFF;	//무한반복할 거면 FMOD_LOOP_NORMAL
    
    FMOD_SOUND* fmod_sound = nullptr;
    
    FMOD_System_CreateSound(fmod_system_, str.c_str(), mode, nullptr, &fmod_sound);
    
    ```
    FMOD_MODE의 설정값들은 (https://documentation.help/FMOD-Studio-API/FMOD_MODE.html)에서 볼 수 있다.
    DEFAULT는 0이다.
    
    
    
    ##### **(3) 재생할 사운드 설정**
    사운드 재생엔 사운드, 채널 두 개를 넘겨줘야 한다.
    채널은 따로 만드는 게 아니라 그냥 FMOD_CHANNEL 널포인터를 만든 다음 사운드를 재생할 때 사운드와 같이 넘겨주면 한 번에 초기화된다.
    
    ```
    FMOD_System_PlaySound(fmod_system_, 재생할 파일경로, NULL, false, &channel);
    
    ```
    아직까진 소리가 나지 않는다.
    
    
    
    ##### **(4) 사운드 재생**
    
    ```
    FMOD_System_Update(fmod_system_);
    ```
    이 함수를 매 프레임 계속 실행해줘야 사운드가 재생된다.
    
    ```
    FMOD_BOOL is_playing = 0;
    FMOD_Channel_IsPlaying(channel, &is_playing);
    ```
    해당 채널에서 사운드가 현재 재생중인지 체크한다.
    설정된 사운드가 없거나 재생이 끝난 상태일 때 false가 된다.
    
    
    ##### **(5) 리소스 정리**
    
    ```
    FMOD_System_Release(fmod_system_);
    ```
    다 사용한 뒤엔 릴리즈를 해준다.
    
    
    # 3. FmodSound 클래스
    
    직관적으로 사용하려면 생명주기를 Init()함수로 초기화하고, Update()함수를 돌리고, 끝날 때 OnDestroy()를 실행하도록 래핑하는 게 좋겠다.
    그리고 사운드를 재생할 땐 Play()함수를, 정지할 땐 Stop()함수를 호출하도록 하자.
    
    
    그리고 재생할 사운드는 크게 "배경음악"과 "효과음"이 있을 텐데, 배경음악은 동시에 1개만 들려야 하고, 효과음은 동시에 여러 개가 들릴 수 있을 것이다.
    따라서 배경음악용 채널 1개, 효과음용 채널 31개로 총 32개의 채널을 사용했다.
    
    ```
    //FmodSound.h
    #pragma once
    #include "global.h"
    
    class Sound;
    struct ChannelInfo {
    	FMOD_CHANNEL* channel;
    	Sound* sound;
    	bool repeat;
    };
    
    class FmodSound
    {
    	SINGLETON(FmodSound);
    private:
    
    	FMOD_SYSTEM* fmod_system_ =	nullptr;
    
    	const int CHANNEL_BACKGROUND = 0;
    	ChannelInfo channels_[32];
    	FMOD_SOUND* CreateFmodSound(const tstring& absolute_path);
    	
    public:
    	
    	void Init();
    	void Update();
    
    	void PlayBackground(Sound* sound); //항상 0번 채널에서 재생.
    	void Play(int channel, Sound* sound, bool repeat); //특정 채널에서 재생.
    	void StopBackground();
    	void Stop(int channel);
    	int GetChannel(); //비어있는 채널을 리턴. 없으면 nullptr
    
    	void OnDestroy();
    	friend class Sound;
    	
    };
    
    
    ```
    ```
    //FmodSound.cpp
    #include "FmodSound.h"
    #include "Sound.h"
    FmodSound::FmodSound() {};
    FmodSound::~FmodSound() {};
    void FmodSound::Init()
    {
    	memset(channels_, 0, sizeof(channels_));
    
    	if (FMOD_System_Create(&fmod_system_, FMOD_VERSION) != FMOD_RESULT::FMOD_OK) return;
    	if (FMOD_System_Init(fmod_system_, 32, FMOD_INIT_NORMAL, nullptr) != FMOD_RESULT::FMOD_OK) return;
    
    
    
    
    }
    
    void FmodSound::Update()
    {
    	if (!fmod_system_) return;
    
    	FMOD_System_Update(fmod_system_);
    	for (int i = 0; i < 32; i++) {
    		if (channels_[i].repeat) {
    			FMOD_BOOL is_playing = 0;
    			FMOD_Channel_IsPlaying(channels_[i].channel, &is_playing);
    			if (!is_playing) {
    				Play(i, channels_[i].sound, true);
    			}
    		}
    	}
    
    }
    
    FMOD_SOUND* FmodSound::CreateFmodSound(const tstring& absolute_path)
    {
    	FMOD_MODE mode = 0;
    	mode |= FMOD_LOOP_OFF;
    
    	FMOD_SOUND* fmod_sound = nullptr;
    
    	std::string str = TstringToString(absolute_path);
    
    	FMOD_System_CreateSound(fmod_system_, str.c_str(), mode, nullptr, &fmod_sound);
    	return fmod_sound;
    
    }
    
    void FmodSound::PlayBackground(Sound* sound)
    {
    	StopBackground();
    	Play(CHANNEL_BACKGROUND, sound, true);
    }
    
    void FmodSound::Play(int channel, Sound* sound, bool repeat)
    {
    	channels_[channel].sound = sound;
    	channels_[channel].repeat = repeat;
    	FMOD_System_PlaySound(fmod_system_, sound->get_fmod_sound(), NULL, false, &(channels_[channel].channel));
    }
    
    void FmodSound::StopBackground()
    {
    	Stop(CHANNEL_BACKGROUND);
    }
    
    void FmodSound::Stop(int channel)
    {
    	FMOD_Channel_SetPaused((channels_[channel].channel), true);
    }
    
    int FmodSound::GetChannel()
    {
    	for (int i = 2; i < 32; i++) {
    		if (!channels_[i].channel) {
    			return i;
    		}
    		else {
    			FMOD_BOOL is_playing = 0;
    			FMOD_Channel_IsPlaying(channels_[i].channel, &is_playing);
    
    			if (!channels_[i].repeat && !is_playing) {
    				return i;
    			}
    		}
    		
    	}
    	return -1;
    }
    
    void FmodSound::OnDestroy()
    {
    	FMOD_System_Release(fmod_system_);
    	fmod_system_ = nullptr;
    }
    
    ```
    
    
    
    Sound는 이전에 만들어둔 Res 클래스의 자식 클래스로 구현했다.
    ResManager는 LoadSound()함수를 제공한다. ResManager는 한 번 불러왔던 Sound를 key와 경로의 쌍으로 저장하고 있다.
    최초 1회에 한해서는 Sound를 새로 할당해 로드한 다음 저장하고,
    두번째부터는 해당 키로 저장된 Sound를 리턴한다. 
    
    ```
    //ResManager.h
    #pragma once
    #include "global.h"
    
    class Texture;
    class Sound;
    class ResManager
    {
    	SINGLETON(ResManager);
    private:
    	std::map<tstring, Sound*> sounds_;
    
    public:
    	Sound* LoadSound(const tstring& key, const tstring& relative_path);
    	
    };
    
    
    ```
    ```
    //ResManager.cpp
    #include "ResManager.h"
    #include "Texture.h"
    #include "DXClass.h"
    #include "Sound.h"
    ResManager::ResManager() {}
    
    ResManager::~ResManager() {
    
        for (auto pair : sounds_) {
            delete pair.second;
        }
        sounds_.clear();
    }
    
    
    Sound* ResManager::LoadSound(const tstring& key, const tstring& relative_path)
    {
        auto it = sounds_.find(key);
        if (it != sounds_.end()) {
            return it->second;
        }
        Sound* sound = DEBUG_NEW Sound();
        sound->set_key(key);
        sound->set_relative_path(relative_path);
        sound->Load();
        sounds_.insert(std::make_pair(key, sound));
    
        return sound;
    }
    
    ```
    
    ```
    //Res.h
    #pragma once
    #include "global.h"
    
    class Res
    {
    private :
    	tstring key_;
    	tstring relative_path_;
    	inline void set_key(const tstring& key) { key_ = key; };
    	inline void set_relative_path(const tstring& path) { relative_path_ = path; };
    
    public:
    	Res();
    	virtual ~Res();
    	inline const tstring& get_key() { return key_; };
    	inline const tstring& get_relative_path() { return relative_path_; };
    
    	friend class ResManager;
    };
    
    
    ```
    
    ```
    //Sound.h
    #pragma once
    #include "global.h"
    #include "Res.h"
    class Sound : public Res
    {
    private:
    	Sound();
    	~Sound();
    
    	FMOD_SOUND* fmod_sound_;
    	void Load();
    public:
    	FMOD_SOUND* get_fmod_sound() { return fmod_sound_; };
    
    	friend class ResManager;
    };
    
    
    ```
    ```
    //Sound.cpp
    #include "Sound.h"
    #include "FmodSound.h"
    #include "PathManager.h"
    Sound::Sound()
    {
    }
    Sound::~Sound()
    {
    
    }
    void Sound::Load()
    {
    	tstring sound_path = PathManager::GetInstance()->GetContentPath() + get_relative_path();
    
    	fmod_sound_ = FmodSound::GetInstance()->CreateFmodSound(sound_path);
    }
    
    ```
    
    
    
    배경음악을 재생할 때.
    
    ```
    	//사운드 재생
    	Sound* sound = ResManager::GetInstance()->LoadSound(_T("Farm Background"), _T("sound\\test.mp3"));
    	FmodSound::GetInstance()->PlayBackground(sound);
    ```
    
    효과음을 재생할 때.
    ```
    	Sound* sound = ResManager::GetInstance()->LoadSound(_T("Farm Background"), _T("sound\\test.mp3"));
    	FmodSound::GetInstance()->Play(FmodSound::GetInstance()->GetChannel(), sound, true);
    ```

