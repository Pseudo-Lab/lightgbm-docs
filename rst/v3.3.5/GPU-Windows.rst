GPU Windows 컴파일
=======================

이 가이드는 MinGW 빌드용입니다.

GPU가 포함된 MSVC(Visual Studio) 빌드는 `설치 가이드 <./Installation-Guide.rst#build-gpu-version>`__ 를 참조하세요.
(훨씬 쉽기 때문에 이 방법을 사용하는 것을 권장합니다).

MinGW/gcc를 사용하여 Windows(CLI/R/Python)에 LightGBM GPU 버전을 설치합니다.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

이것은 사전 컴파일된 라이브러리 없이 소스에서 컴파일하는 전체 단계와 Boost 설치 가이드입니다.

설치 단계 (수행하려는 작업에 따라 다름):

-  적절한 OpenCL SDK 설치

-  MinGW 설치

-  Boost 설치

-  Git 설치

-  CMake 설치 

-  LightGBM 바이너리 생성하기

-  CLI에서 LightGBM 디버깅(GPU가 충돌하거나 기타 다른 충돌이 있는 경우)

Visual Studio C++ 컴파일러와 같은 다른 컴파일러를 사용하려는 경우 필요에 따라 단계를 조정해야 합니다.

이 컴파일 튜토리얼에서는 OpenCL 단계에 AMD SDK를 사용합니다.
그러나 원하는 OpenCL SDK를 자유롭게 사용할 수 있으며, PATH를 올바르게 조정하기만 하면 됩니다.

또한 관리자 권한이 필요합니다. 관리자 권한이 없으면 설치할 수 없습니다.

마지막에 원래 PATH로 복원할 수 있습니다.

--------------

PATH 수정하기(초보자)
----------------------------

PATH를 수정하려면 ``제어판`` 으로 이동한 후 다음 그림과 같이 따라하세요:

.. image:: ./_static/images/screenshot-system.png
   :align: center
   :target: ./_static/images/screenshot-system.png
   :alt: A screenshot of the System option under System and Security of the Control Panel.

그다음 ``Advanced`` > ``Environment Variables...`` 으로 이동하세요:

.. image:: ./_static/images/screenshot-advanced-system-settings.png
   :align: center
   :target: ./_static/images/screenshot-advanced-system-settings.png
   :alt: A screenshot of the System Properties window.

``System variables`` 아래 ``Path`` 변수:

.. image:: ./_static/images/screenshot-environment-variables.png
   :align: center
   :target: ./_static/images/screenshot-environment-variables.png
   :alt: A screenshot of the Environment variables window with variable path selected under the system variables.

--------------

안티바이러스 성능 영향
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

타사 바이러스 백신이나 Windows에 사전 설치된 기본 바이러스 백신을 사용하지 않는 경우에는 적용되지 않습니다.

**Windows Defender 또는 기타 바이러스 백신은 설치 단계를 수행하는 속도에 상당한 영향을 미칩니다**.
사용 중이라면 빌드 및 설정을 모두 마칠 때까지 **일시적으로 꺼두었다가** 다시 켜는 것이 좋습니다.

--------------

OpenCL SDK 설치
-----------------------

적절한 OpenCL SDK를 설치하려면 올바른 벤더의 SDK 소스를 다운로드해야 합니다.
LightGBM의 용도를 알아야 합니다!

-  인텔에서 실행하는 경우 `Intel SDK for OpenCL`_ 을 다운받습니다(**추천하지 않음**).

-  AMD에서 실행하려면 AMD APP SDK를 다운로드하십시오(``Linux용``_ 및 ``Windows용``_ 다운로드). 드라이버와 함께 제공된 라이브러리에 일부 기능이 없는 경우 GPU 드라이버 패키지의 ``OpenCL.dll``을 SDK의 드라이버로 교체할 수 있습니다.

-  NVIDIA에서 실행하려면 `CUDA 툴킷`_ 을 다운로드합니다.

-  또는 `Khronos 공식 OpenCL 헤더`_ 를 사용하면 CMake 모듈이 시스템에서 사용되는 OpenCL 라이브러리를 자동으로 찾을 수 있지만 결과는 이식성이 없을 수 있습니다.

추가 읽기 및 대응 표(특히 AMD APP SDK를 사용하는 인텔 CPU와 같은 크로스 플랫폼 장치를 사용하려는 경우): `GPU SDK 대응 및 디바이스 타겟팅 표 <./GPU-Targets.rst>`__.

**경고**: 인텔 OpenCL을 사용하는 것은 권장되지 않으며, OpenCL 표준을 준수하지 않아 컴퓨터가 충돌할 수 있습니다.
CPU에서 LightGBM + OpenCL을 사용하는 것이 목적이라면 대신 AMD APP SDK를 사용하세요(인텔 CPU에서도 문제 없이 실행 가능).

--------------

MinGW 올바른 컴파일러 선택
--------------------------------

R 없이 LightGBM을 사용하려면 MinGW를 설치해야 합니다.
MinGW 설치는 간단합니다. `이것`_ 을 다운로드하세요.

x86\_64 아키텍처를 사용하고 있는지 확인하고 다른 것을 수정하지 마세요.
이전 MinGW 버전이 필요한 경우 최신 버전이 아닌 다른 버전을 선택할 수 있습니다.

.. image:: ./_static/images/screenshot-mingw-installation.png
   :align: center
   :target: ./_static/images/screenshot-mingw-installation.png
   :alt: A screenshot of the Min G W installation setup settings window.

그런 다음 PATH에 다음을 추가하여 MinGW 버전에 맞게 조정합니다:

::

    C:\Program Files\mingw-w64\x86_64-5.3.0-posix-seh-rt_v4-rev0\mingw64\bin

**경고**: R 사용자(R용 LightGBM을 원하지 않는 경우에도)

RTools와 MinGW가 설치되어 있고 R에서 LightGBM을 사용하려면,
PATH에서 MinGW을 삭제합니다 (유지할 경로: 32비트 R 설치의 경우 ``c:\Rtools\bin;c:\Rtools\mingw_32\bin``,
64비트 R 설치의 경우 ``c:\Rtools\bin;c:\Rtools\mingw_64\bin``).

명령 프롬프트에서 다음을 실행하여 사용 중인 MinGW 버전을 확인할 수 있습니다: ``gcc -v``:

.. image:: ./_static/images/screenshot-r-mingw-used.png
   :align: center
   :target: ./_static/images/screenshot-r-mingw-used.png
   :alt: A screenshot of the administrator command prompt where G C C version is being checked.

R에 32비트 또는 64비트 MinGW가 필요한지 확인하려면 평소와 같이 LightGBM을 설치하고 다음을 확인합니다:

.. code:: r

    * installing *source* package 'lightgbm' ...
    ** libs
    c:/Rtools/mingw_64/bin/g++

``mingw_64``라고 나오면 64비트 버전이 필요하고(``c:\Rtools\bin;c:\Rtools\mingw_64\bin``),
그렇지 않으면 32비트 버전이 필요합니다(``c:\Rtools\bin;c:\Rtools\mingw_32\bin``). 후자는 매우 드물고 검증되지 않은 사례입니다.

노트: If you are using `Rtools` 4.0 또는 그 이후 버전을 사용한다면 경로에 `mingw_64` 대신 `mingw64`를 사용하고 (`C:\rtools40\mingw64\bin`), `mingw_32` 대신 `mingw32`를 사용합니다(`C:\rtools40\mingw32\bin`). 32비트 버전은 Rtools 4.0에서 지원되지 않는 솔루션으로 남아 있습니다.

사전 빌드한 Boost 다운로드하기
---------------------------

Download  `Prebuilt Boost x86_64`_ 나 `Prebuilt Boost i686`_ 를 다운로드하고 `7zip`_ 으로 압축을 풉니다. 또는 소스에서 Boost를 빌드할 수 있습니다.

--------------

Boost 컴파일
-----------------

부스트를 설치하려면 부스트를 다운로드하고 설치해야 합니다.
CPU 속도와 네트워크 속도에 따라 약 10분에서 몇 시간까지 소요됩니다.

``C:\boost`` 에 설치와 일반 설치를 가정합니다 (유닉스 변형: 버전 관리와 타입 태그가 없음).

컴파일러를 확인하기 위한 필수 단계가 하나 있습니다:

-  **경고**: R 설치를 원하는 경우:
   PATH 변수에 이미 MinGW가 있는 경우, 이를 제거하세요(그렇지 않으면 잘못된 컴파일러로 연결됩니다).

-  **경고**: CLI 설치를 원하는 경우:
   PATH 변수에 이미 Rtools가 있는 경우 이를 제거하세요(그렇지 않으면 잘못된 컴파일러로 연결됩니다).

-  R 설치는 PATH에 Rtools가 있어야 합니다.

-  CLI / 파이썬 설치는 PATH에 (Rtools가 아니라) MinGW가 있어야 합니다.

또한 폴더 경로로 ``C:\boost``를 사용한다고 가정하면 PATH에 다음 경로를 추가해야 합니다: ``C:\boost\boost-build\bin``, ``C:\boost\boost-build\include\boost``
다른 곳에 설치한다면 ``C:\boost``를 수정하세요.

이제 필요한 부스트 라이브러리 다운로드하고 컴파일을 시작할 수 있습니다:

-  `Boost`_ 를 다운로드합니다(1.63.0 버전의 파일 이름은 ``boost_1_63_0.zip``입니다)

-  ``C:\boost``에 압축을 해제합니다.

-  명령 프롬프트를 열고 다음을 실행합니다.

   .. code::

       cd C:\boost\boost_1_63_0\tools\build
       bootstrap.bat gcc
       b2 install --prefix="C:\boost\boost-build" toolset=gcc
       cd C:\boost\boost_1_63_0

Boost 라이브러리를 빌드하려면 명령 프롬프트에서 두 가지 옵션이 있습니다:

-  단일 코어를 사용한다면 기본값을 사용할 수 있습니다.

   .. code::

       b2 install --build_dir="C:\boost\boost-build" --prefix="C:\boost\boost-build" toolset=gcc --with=filesystem,system threading=multi --layout=system release

-  멀티스레드 라이브러리를 원하는 경우 ``-j N``을 추가하고 N을 코어/스레드 개수로 바꿉니다.
   예를 들어 두 개의 코어를 가지고 있다면 다음과 같습니다.

   .. code::

       b2 install --build_dir="C:\boost\boost-build" --prefix="C:\boost\boost-build" toolset=gcc --with=filesystem,system threading=multi --layout=system release -j 2

파이썬 등에서 발생하는 오류는 모두 무시해도 상관없습니다.

폴더는 마지막에 다음과 같이 표시되어야 합니다(완전히 상세하지는 않음):

::

    - C
      |--- boost
      |------ boost_1_63_0
      |--------- some folders and files
      |------ boost-build
      |--------- bin
      |--------- include
      |------------ boost
      |--------- lib
      |--------- share

부스트 컴파일이 끝나면 (대략적으로) 이런 결과를 얻을 수 있습니다:

.. image:: ./_static/images/screenshot-boost-compiled.png
   :align: center
   :target: ./_static/images/screenshot-boost-compiled.png
   :alt: A screenshot of the command prompt that ends with text that reads - updated 14621 targets.

오류가 발생하는 경우:

-  부스트 디렉터리 지웁니다.

-  명령 프롬프트를 닫습니다.

-  PATH에 
   ``C:\boost\boost-build\bin``, ``C:\boost\boost-build\include\boost``를 추가했는지 확인하세요 (다른 폴더를 사용하는 경우 적절하게 수정합니다)

-  Boost 컴파일 단계를 다시 수행합니다 (추출 => 명령 프롬프트 => ``cd`` => ``bootstrap`` => ``b2`` => ``cd`` => ``b2``

--------------

깃 설치
----------------

윈도우용 깃(Git) 설치는 간단합니다. 다음 `링크`_ 를 사용하세요.

.. image:: ./_static/images/screenshot-git-for-windows.png
   :align: center
   :target: ./_static/images/screenshot-git-for-windows.png
   :alt: A screenshot of the website to download git that shows various versions of git compatible with 32 bit and 64 bit Windows separately.

이제 깃허브에서 LightGBM 저장소를 가져올 수 있습니다. Git Bash를 열고 다음 명령을 실행합니다:

::

    cd C:/
    mkdir github_repos
    cd github_repos
    git clone --recursive https://github.com/microsoft/LightGBM

이제 LightGBM 저장소 사본이 ``C:\github_repos\LightGBM`` 아래에 있어야 합니다.
자유롭게 다른 폴더를 사용할 수 있지만 적절하게 맞추어야 합니다.

Git Bash를 그대로 열어 두세요.

--------------

CMake 설치, 설정, 생성
---------------------------------------------

**CLI / 파이썬 사용자 전용**

CMake를 설치하려면 먼저 다운로드한 다음 LightGBM을 위한 많은 설정이 필요합니다:

.. image:: ./_static/images/screenshot-downloading-cmake.png
   :align: center
   :target: ./_static/images/screenshot-downloading-cmake.png
   :alt: A screenshot of the binary distributions of C Make for downloading on 64 bit Windows.

-  `CMake`_ 를 다운로드합니다(3.8 또는 그 이상)

-  CMake를 설치합니다.

-  cmake-gui를 실행합니다.

-  ``Where is the source code``를 위해 LightGBM을 저장한 폴더를 선택합니다. 앞서 선택한 폴더는 ``C:/github_repos/LightGBM``입니다.

-  ``Where to build the binaries``를 위해 폴더 이름을 복사하고 ``/build``를 덧붙입니다.
   우리의 경우에 ``C:/github_repos/LightGBM/build``가 됩니다.

-  ``Configure``를 클릭합니다.

   .. image:: ./_static/images/screenshot-create-directory.png
      :align: center
      :target: ./_static/images/screenshot-create-directory.png
      :alt: A screenshot with a pop-up window that reads - Build directory does not exist, should I create it?

   .. image:: ./_static/images/screenshot-mingw-makefiles-to-use.png
      :align: center
      :target: ./_static/images/screenshot-mingw-makefiles-to-use.png
      :alt: A screenshot that asks to specify the generator for the project which should be selected as Min G W makefiles and selected as the use default native compilers option.

-  ``USE_GPU``를 찾아 체크박스에 체크를 합니다.

   .. image:: ./_static/images/screenshot-use-gpu.png
      :align: center
      :target: ./_static/images/screenshot-use-gpu.png
      :alt: A screenshot of the C Make window where the checkbox with the test Use G P U is checked.

-  ``Configure``를 클릭합니다.

   Configure을 클릭하면 대략 다음과 같은 메시지가 표시됩니다:

   .. image:: ./_static/images/screenshot-configured-lightgbm.png
      :align: center
      :target: ./_static/images/screenshot-configured-lightgbm.png
      :alt: A screenshot of the C Make window after clicking on the configure button.

   ::

       Looking for CL_VERSION_2_0
       Looking for CL_VERSION_2_0 - found
       Found OpenCL: C:/Windows/System32/OpenCL.dll (found version "2.0")
       OpenCL include directory:C:/Program Files (x86)/AMD APP SDK/3.0/include
       Boost version: 1.63.0
       Found the following Boost libraries:
         filesystem
         system
       Configuring done

-  ``Generate``를 클릭하면 다음과 같은 메시지가 나옵니다:

   ::

       Generating done

CMake가 올바른 요소를 찾는 데 큰 도움을 주기 때문에 간단합니다.

--------------

LightGBM 컴파일 (CLI: 최종 단계)
--------------------------------------

CLI에서 설치
~~~~~~~~~~~~~~~~~~~

**CLI / 파이썬 사용자**

중요하고 어려운 단계는 모두 이전에 완료되었기 때문에 LightGBM 라이브러리 생성은 매우 간단합니다.

열어둔 Git Bash 콘솔에서 모든 작업을 수행할 수 있습니다:

-  이전에 Git Bash 콘솔을 닫았다면 다음 명령으로 빌드 폴더로 돌아갑니다:

   ::

       cd C:/github_repos/LightGBM/build

-  이전에 Git Bash 콘솔을 닫지 않았다면 다음 명령으로 빌드 폴더로 이동합니다:

   ::

       cd LightGBM/build

-  ``make``로 MinGW를 설정합니다.

   ::

       alias make='mingw32-make'

   그렇지 않으면 에러와 이름 충돌에 주의하세요!

-  Git Bash에서 ``make``를 실행하면 LightGBM이 설치되는 것을 확인할 수 있습니다!

.. image:: ./_static/images/screenshot-lightgbm-with-gpu-support-compiled.png
   :align: center
   :target: ./_static/images/screenshot-lightgbm-with-gpu-support-compiled.png
   :alt: A screenshot of the git bash window with Light G B M successfully installed.

모든 것이 올바르게 완료되었다면 이제 GPU를 지원하는 CLI LightGBM을 컴파일했습니다!

CLI에서 테스트
~~~~~~~~~~~~~~

이제 CLI에서 **명령 프롬프트**(Git Bash가 아닌)로 직접 LightGBM을 테스트할 수 있습니다:

::

    cd C:/github_repos/LightGBM/examples/binary_classification
    "../../lightgbm.exe" config=train.conf data=binary.train valid=binary.test objective=binary device=gpu

.. image:: ./_static/images/screenshot-lightgbm-in-cli-with-gpu.png
   :align: center
   :target: ./_static/images/screenshot-lightgbm-in-cli-with-gpu.png
   :alt: A screenshot of the command prompt where a binary classification model is being trained using Light G B M.

이 단계에 도달한 것을 축하합니다!

훈련을 위해 올바른 CPU 또는 GPU를 지정하는 방법을 알아보려면 다음을 참조하세요.: `GPU SDK Correspondence and Device Targeting Table <./GPU-Targets.rst>`__.

--------------

CLI에서 LightGBM 충돌 디버깅하기
---------------------------------

이제 LightGBM을 컴파일하고 실행해보면, GPU를 사용하는 경우 항상 세그멘테이션 오류 또는 문서화되지 않은 충돌이 발생합니다:

.. image:: ./_static/images/screenshot-segmentation-fault.png
   :align: center
   :target: ./_static/images/screenshot-segmentation-fault.png
   :alt: A screenshot of the command prompt where a segmentation fault has occurred while using Light G B M.

올바른 장치를 사용하고 있는지 확인하세요(``Using GPU device: ...``). `GPUCapsViewer`_ 를 사용하여 OpenCL 장치 목록을 확인할 수 있으며, 통합 (인텔) 및 외장형 GPU가 모두 설치되어 있는 경우 외장형(AMD/NVIDIA) GPU를 사용하고 있는지 확인합니다.
또한, 첫 번째 플랫폼과 디바이스 또는 기본 플랫폼과 디바이스를 사용하려면 ``gpu_device_id = 0`` 및 ``gpu_platform_id = 0`` 또는 ``gpu_device_id = -1`` 및 ``gpu_platform_id = -1``을 설정해 보세요.
그래도 작동하지 않으면 아래의 모든 단계를 따라야 합니다.

디버깅 모드를 추가하려면 LightGBM의 컴파일 단계를 다시 수행해야 합니다. 여기에는 다음이 포함됩니다:

-  ``C:/github_repos/LightGBM/build`` 폴더를 삭제합니다.

-  ``lightgbm.exe``, ``lib_lightgbm.dll``, ``lib_lightgbm.dll.a`` 파일을 삭제합니다.

.. image:: ./_static/images/screenshot-files-to-remove.png
   :align: center
   :target: ./_static/images/screenshot-files-to-remove.png
   :alt: A screenshot of the Light G B M folder with 1 folder and 3 files selected to be removed.

파일을 제거한 후 CMake로 이동하여 일반적인 단계를 따릅니다.
"Generate"를 클릭하기 전에 "Add Entry"를 클릭합니다:

.. image:: ./_static/images/screenshot-added-manual-entry-in-cmake.png
   :align: center
   :target: ./_static/images/screenshot-added-manual-entry-in-cmake.png
   :alt: A screenshot of the Cache Entry popup where the name is set to CMAKE_BUILD_TYPE in all caps, the type is set to STRING in all caps and the value is set to Debug.

또한 Configure와 Generate를 클릭합니다:

.. image:: ./_static/images/screenshot-configured-and-generated-cmake.png
   :align: center
   :target: ./_static/images/screenshot-configured-and-generated-cmake.png
   :alt: A screenshot of the C Make window after clicking on configure and generate.

그런 다음 거기에서 일반적인 LightGBM CLI 설치를 따르세요.

LightGBM CLI를 설치한 후, LightGBM이 ``C:\github_repos\LightGBM``에 있다고 가정합니다,
명령 프롬프트를 열고 다음을 실행합니다:

::

    gdb --args "../../lightgbm.exe" config=train.conf data=binary.train valid=binary.test objective=binary device=gpu

.. image:: ./_static/images/screenshot-debug-run.png
   :align: center
   :target: ./_static/images/screenshot-debug-run.png
   :alt: A screenshot of the command prompt after the command above is run.

``run``를 입력하게 엔터키를 누르세요.

아마 다음과 비슷한 결과가 나올 것입니다:

::

    [LightGBM] [Info] This is the GPU trainer!!
    [LightGBM] [Info] Total Bins 6143
    [LightGBM] [Info] Number of data: 7000, number of used features: 28
    [New Thread 105220.0x1a62c]
    [LightGBM] [Info] Using GPU Device: Oland, Vendor: Advanced Micro Devices, Inc.
    [LightGBM] [Info] Compiling OpenCL Kernel with 256 bins...

    Program received signal SIGSEGV, Segmentation fault.
    0x00007ffbb37c11f1 in strlen () from C:\Windows\system32\msvcrt.dll
    (gdb)

거기에 ``backtrace``를 작성하고 gdb가 두 가지 선택을 요청하는 횟수만큼 Enter 키를 누릅니다:

::

    Program received signal SIGSEGV, Segmentation fault.
    0x00007ffbb37c11f1 in strlen () from C:\Windows\system32\msvcrt.dll
    (gdb) backtrace
    #0  0x00007ffbb37c11f1 in strlen () from C:\Windows\system32\msvcrt.dll
    #1  0x000000000048bbe5 in std::char_traits<char>::length (__s=0x0)
        at C:/PROGRA~1/MINGW-~1/X86_64~1.0-P/mingw64/x86_64-w64-mingw32/include/c++/bits/char_traits.h:267
    #2  std::operator+<char, std::char_traits<char>, std::allocator<char> > (__rhs="\\", __lhs=0x0)
        at C:/PROGRA~1/MINGW-~1/X86_64~1.0-P/mingw64/x86_64-w64-mingw32/include/c++/bits/basic_string.tcc:1157
    #3  boost::compute::detail::appdata_path[abi:cxx11]() () at C:/boost/boost-build/include/boost/compute/detail/path.hpp:38
    #4  0x000000000048eec3 in boost::compute::detail::program_binary_path (hash="d27987d5bd61e2d28cd32b8d7a7916126354dc81", create=create@entry=false)
        at C:/boost/boost-build/include/boost/compute/detail/path.hpp:46
    #5  0x00000000004913de in boost::compute::program::load_program_binary (hash="d27987d5bd61e2d28cd32b8d7a7916126354dc81", ctx=...)
        at C:/boost/boost-build/include/boost/compute/program.hpp:605
    #6  0x0000000000490ece in boost::compute::program::build_with_source (
        source="\n#ifndef _HISTOGRAM_256_KERNEL_\n#define _HISTOGRAM_256_KERNEL_\n\n#pragma OPENCL EXTENSION cl_khr_local_int32_base_atomics : enable\n#pragma OPENC
    L EXTENSION cl_khr_global_int32_base_atomics : enable\n\n//"..., context=...,
        options=" -D POWER_FEATURE_WORKGROUPS=5 -D USE_CONSTANT_BUF=0 -D USE_DP_FLOAT=0 -D CONST_HESSIAN=0 -cl-strict-aliasing -cl-mad-enable -cl-no-signed-zeros -c
    l-fast-relaxed-math") at C:/boost/boost-build/include/boost/compute/program.hpp:549
    #7  0x0000000000454339 in LightGBM::GPUTreeLearner::BuildGPUKernels () at C:\LightGBM\src\treelearner\gpu_tree_learner.cpp:583
    #8  0x00000000636044f2 in libgomp-1!GOMP_parallel () from C:\Program Files\mingw-w64\x86_64-5.3.0-posix-seh-rt_v4-rev0\mingw64\bin\libgomp-1.dll
    #9  0x0000000000455e7e in LightGBM::GPUTreeLearner::BuildGPUKernels (this=this@entry=0x3b9cac0)
        at C:\LightGBM\src\treelearner\gpu_tree_learner.cpp:569
    #10 0x0000000000457b49 in LightGBM::GPUTreeLearner::InitGPU (this=0x3b9cac0, platform_id=<optimized out>, device_id=<optimized out>)
        at C:\LightGBM\src\treelearner\gpu_tree_learner.cpp:720
    #11 0x0000000000410395 in LightGBM::GBDT::ResetTrainingData (this=0x1f26c90, config=<optimized out>, train_data=0x1f28180, objective_function=0x1f280e0,
        training_metrics=std::vector of length 2, capacity 2 = {...}) at C:\LightGBM\src\boosting\gbdt.cpp:98
    #12 0x0000000000402e93 in LightGBM::Application::InitTrain (this=this@entry=0x23f9d0) at C:\LightGBM\src\application\application.cpp:213
    ---Type <return> to continue, or q <return> to quit---
    #13 0x00000000004f0b55 in LightGBM::Application::Run (this=0x23f9d0) at C:/LightGBM/include/LightGBM/application.h:84
    #14 main (argc=6, argv=0x1f21e90) at C:\LightGBM\src\main.cpp:7

명령 프롬프트를 마우스 오른쪽 버튼으로 클릭하고 "Mark"를 클릭한 다음 첫 번째 줄(명령 프롬프트에 gdb가 포함된 경우)부터 인쇄된 마지막 줄까지 모든 로그가 포함된 모든 텍스트를 선택합니다:

::

    C:\LightGBM\examples\binary_classification>gdb --args "../../lightgbm.exe" config=train.conf data=binary.train valid=binary.test objective=binary device=gpu
    GNU gdb (GDB) 7.10.1
    Copyright (C) 2015 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-w64-mingw32".
    Type "show configuration" for configuration details.
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>.
    Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.
    For help, type "help".
    Type "apropos word" to search for commands related to "word"...
    Reading symbols from ../../lightgbm.exe...done.
    (gdb) run
    Starting program: C:\LightGBM\lightgbm.exe "config=train.conf" "data=binary.train" "valid=binary.test" "objective=binary" "device=gpu"
    [New Thread 105220.0x199b8]
    [New Thread 105220.0x783c]
    [Thread 105220.0x783c exited with code 0]
    [LightGBM] [Info] Finished loading parameters
    [New Thread 105220.0x19490]
    [New Thread 105220.0x1a71c]
    [New Thread 105220.0x19a24]
    [New Thread 105220.0x4fb0]
    [Thread 105220.0x4fb0 exited with code 0]
    [LightGBM] [Info] Loading weights...
    [New Thread 105220.0x19988]
    [Thread 105220.0x19988 exited with code 0]
    [New Thread 105220.0x1a8fc]
    [Thread 105220.0x1a8fc exited with code 0]
    [LightGBM] [Info] Loading weights...
    [New Thread 105220.0x1a90c]
    [Thread 105220.0x1a90c exited with code 0]
    [LightGBM] [Info] Finished loading data in 1.011408 seconds
    [LightGBM] [Info] Number of positive: 3716, number of negative: 3284
    [LightGBM] [Info] This is the GPU trainer!!
    [LightGBM] [Info] Total Bins 6143
    [LightGBM] [Info] Number of data: 7000, number of used features: 28
    [New Thread 105220.0x1a62c]
    [LightGBM] [Info] Using GPU Device: Oland, Vendor: Advanced Micro Devices, Inc.
    [LightGBM] [Info] Compiling OpenCL Kernel with 256 bins...

    Program received signal SIGSEGV, Segmentation fault.
    0x00007ffbb37c11f1 in strlen () from C:\Windows\system32\msvcrt.dll
    (gdb) backtrace
    #0  0x00007ffbb37c11f1 in strlen () from C:\Windows\system32\msvcrt.dll
    #1  0x000000000048bbe5 in std::char_traits<char>::length (__s=0x0)
        at C:/PROGRA~1/MINGW-~1/X86_64~1.0-P/mingw64/x86_64-w64-mingw32/include/c++/bits/char_traits.h:267
    #2  std::operator+<char, std::char_traits<char>, std::allocator<char> > (__rhs="\\", __lhs=0x0)
        at C:/PROGRA~1/MINGW-~1/X86_64~1.0-P/mingw64/x86_64-w64-mingw32/include/c++/bits/basic_string.tcc:1157
    #3  boost::compute::detail::appdata_path[abi:cxx11]() () at C:/boost/boost-build/include/boost/compute/detail/path.hpp:38
    #4  0x000000000048eec3 in boost::compute::detail::program_binary_path (hash="d27987d5bd61e2d28cd32b8d7a7916126354dc81", create=create@entry=false)
        at C:/boost/boost-build/include/boost/compute/detail/path.hpp:46
    #5  0x00000000004913de in boost::compute::program::load_program_binary (hash="d27987d5bd61e2d28cd32b8d7a7916126354dc81", ctx=...)
        at C:/boost/boost-build/include/boost/compute/program.hpp:605
    #6  0x0000000000490ece in boost::compute::program::build_with_source (
        source="\n#ifndef _HISTOGRAM_256_KERNEL_\n#define _HISTOGRAM_256_KERNEL_\n\n#pragma OPENCL EXTENSION cl_khr_local_int32_base_atomics : enable\n#pragma OPENCL EXTENSION cl_khr_global_int32_base_atomics : enable\n\n//"..., context=...,
        options=" -D POWER_FEATURE_WORKGROUPS=5 -D USE_CONSTANT_BUF=0 -D USE_DP_FLOAT=0 -D CONST_HESSIAN=0 -cl-strict-aliasing -cl-mad-enable -cl-no-signed-zeros -cl-fast-relaxed-math") at C:/boost/boost-build/include/boost/compute/program.hpp:549
    #7  0x0000000000454339 in LightGBM::GPUTreeLearner::BuildGPUKernels () at C:\LightGBM\src\treelearner\gpu_tree_learner.cpp:583
    #8  0x00000000636044f2 in libgomp-1!GOMP_parallel () from C:\Program Files\mingw-w64\x86_64-5.3.0-posix-seh-rt_v4-rev0\mingw64\bin\libgomp-1.dll
    #9  0x0000000000455e7e in LightGBM::GPUTreeLearner::BuildGPUKernels (this=this@entry=0x3b9cac0)
        at C:\LightGBM\src\treelearner\gpu_tree_learner.cpp:569
    #10 0x0000000000457b49 in LightGBM::GPUTreeLearner::InitGPU (this=0x3b9cac0, platform_id=<optimized out>, device_id=<optimized out>)
        at C:\LightGBM\src\treelearner\gpu_tree_learner.cpp:720
    #11 0x0000000000410395 in LightGBM::GBDT::ResetTrainingData (this=0x1f26c90, config=<optimized out>, train_data=0x1f28180, objective_function=0x1f280e0,
        training_metrics=std::vector of length 2, capacity 2 = {...}) at C:\LightGBM\src\boosting\gbdt.cpp:98
    #12 0x0000000000402e93 in LightGBM::Application::InitTrain (this=this@entry=0x23f9d0) at C:\LightGBM\src\application\application.cpp:213
    ---Type <return> to continue, or q <return> to quit---
    #13 0x00000000004f0b55 in LightGBM::Application::Run (this=0x23f9d0) at C:/LightGBM/include/LightGBM/application.h:84
    #14 main (argc=6, argv=0x1f21e90) at C:\LightGBM\src\main.cpp:7

그리고 해당 로그를 사용하여 GitHub `이슈`_ 를 엽니다.

.. _Intel SDK for OpenCL: https://software.intel.com/en-us/articles/opencl-drivers

.. _CUDA 툴킷: https://developer.nvidia.com/cuda-downloads

.. _Linux용: https://github.com/microsoft/LightGBM/releases/download/v2.0.12/AMD-APP-SDKInstaller-v3.0.130.136-GA-linux64.tar.bz2

.. _Windows용: https://github.com/microsoft/LightGBM/releases/download/v2.0.12/AMD-APP-SDKInstaller-v3.0.130.135-GA-windows-F-x64.exe

.. _Khronos 공식 OpenCL 헤더: https://github.com/KhronosGroup/OpenCL-Headers

.. _이것: http://iweb.dl.sourceforge.net/project/mingw-w64/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/installer/mingw-w64-install.exe

.. _Boost: https://www.boost.org/users/history/

.. _Prebuilt Boost x86_64: https://mirror.linux-ia64.org/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/mingw64-boost-static-1.66.0-6.fc32.noarch.rpm

.. _Prebuilt Boost i686: https://mirror.linux-ia64.org/fedora/linux/releases/32/Everything/x86_64/os/Packages/m/mingw32-boost-static-1.66.0-6.fc32.noarch.rpm

.. _7zip: https://www.7-zip.org/download.html

.. _링크: https://git-scm.com/download/win

.. _CMake: https://cmake.org/download/

.. _이슈: https://github.com/microsoft/LightGBM/issues

.. _GPUCapsViewer: http://www.ozone3d.net/gpu_caps_viewer/
