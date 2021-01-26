# 2021_android_video_player
android video player with FFmpeg

안드로이드는 오픈 플랫폼이기 때문에 단말기가 제공하는 동영상 플레이어의 기능과 지원하는 포맷이 제조사마다 다르다.
서비스 제공자 입장에서는 다양한 종류의 단말기에서 동일한 사용자 서비스를 제공해야하기 때문에 추가적인 노력이 필요하고 따라서 안드로이드 단말기는 제공하는 기능이 플레이어 마다 다르며 
이는 지원하는 동영상 포맷도 다르기 때문이다.

그래서 Android 영상 서비스를 위해서 독자적인 영상 플레이어를 갖추는게 좋으므로 대표적인 오픈소스인 FFmpeg(http://ffmpeg.org/)를 이용해 Android 영상 플레이어를 만들었다.

#코덱과 컨테이너 그리고 플레이어
일반적인 사용자 입장에서 동영상(비디오)에는 오디오가 포함되어 있다고 생각하기 쉬우나, 전문적인 입장에서 바라본다면 이는 서로 다른 영역이다.
MP3같은 오디오 압축코덱이 소리를 담당하고, MPEG, MPEG-4 AVC(H.264), WMV9등의 동영상 압축코덱이 있으며 이는 동영상 정보를 압축하는 알고리즘 종류를 지칭하는 것이라고 이해하는 것도 좋다.
동영상이란 정지 영상(동영상 프레임)을 초당 수 개에서 수십 개까지 빠르게 보여주는 것을 말하는 것이고, 압축 동영상이란 최소한의 정지 영상(동영상 프레임)을 초당 수 개에서 수십 개까지 빠르게 보여주는 것이고,
압축 동영상이란 최소한의 정지 영상정보만으로 완전한 동영상 정보를 만들어 내어 이를 재생해낼 수 있도록 하는 것이다.

우리가 흔히 떠올리는 .mp4, wmv등의 파일은 Digital Container Format이라고 부르는 것이고, 이들은 코덱이 아니다. 이들은 동영상/오디오 코덱을 이용하여 데이터를 저장하는 방식과 재생 동기화 정보등의
부가 정보를 담고 있는 파일 형식을 말하며, 영상 플레이어는 이 Didital Container Format을 읽어서 동영상/오디오를 재생하는 프로그램을 뜻한다.

원본 영상 데이터에서 특정 영상 코덱으로 변환하는 것이 인코딩이라 하고, 변환한 데이터를 파일에 담는 것을 먹싱(muxing)이라고 한다. 동영상을 재생하려면 역순으로 해야 하고,
이때 동영상 파일로부터 비트스트림(bit-stream) 데이터를 추출하는 과정을 디먹싱(demuxing)이라고 한다. 이렇게 디먹싱하게 되면 어떤 코덱으로 인코딩되었는지 알 수 있고 
그리하여 적합한 코덱으로 디코딩하면 원본 데이터 영상을 얻을 수 있다.

FFmpeg
FFmpeg은 크로스 플랫폼을 지원하는 오픈소스 멀티미디어 프레임워크이다. FFmpeg을 이용해 인코딩/디코딩, 트랜스코딩, 먹싱과 디먹싱, 스트림(stream)은 물론 '재생'을 포함한
멀티미디어 관련 대부분의 기능을 가지고 있다.

FFmpeg의 라이센스는 GPL과 LGPL이다. FFmpeg에는 다음과 같은 다양한 세부 라이브러리가 있다.

- libavcodec: 오디오/비디오의 인코더/디코더
- libavformat: 오디오/비디오 컨테이너 포맷의 muxer/demuxer
- libavutil: FFmpeg 개발 시 필요한 다양한 유틸리티
- libpostproc: video post-processing
- libswscale: 비디오의 image scaling, color-space, pixel-format 변환
- libavfilter: 인코더와 디코더 사이에서 오디오/비디오를 변경하고 검사
- libswresample: 오디오 리샘플링(audio resampling)

가장 먼저 libavformat을 이용하여 디먹싱을 한다. 이렇게 디먹싱하면 비트스트림 데이터를 얻는데, 이것으로 어떤 코덱을 사용하는지 파악할 수 있다.
그 후 libavcodec을 이용하여 디코딩을 한다. libavcodec은 추출한 코덱 정보를 바탕으로 적합한 디코더로 비트 스트림에서 원본 데이터를 추출한다.

Android NDK
Android NDK(Native Development Kit)는 C/C++과 같은 네이티드 언어로 개발한 라이브러를 Android 앱에서 사용할 수 있게 하는 개발 도구이다.
구성사항은 다음과 같다

- Cross-toolchains(compiler, linker 등)
- Android Platform을 이요하기 위한 헤더 파일과 라이브러리
- 문서와 샘플코드

현재 Android NDK을 지원하는 위한 ARM instruction set은 다음과 같다.

- ARMv5TE 
- ARMv7-A
- X86 instructions

ARMv5TE machine code는 모든 ARM 기반의 Android 기기에서 동작한다. ARMv7-A는 호환되는 CPU가 탑재된 기기에서만 동작한다.
두 instruction set의 주요한 차이점은 ARMv7-A만 H/W FPU, Thumb-2, NEON instructions를 지원한다는 데 있다.
Android NDK로 개발할 때에 두 instruction set 모두를 지원하거나 둘 중 하나만 지원하도록 할 수 있다.
