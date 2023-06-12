설치 가이드
==================

이 문서는 LightGBM 명령줄 인터페이스(CLI)를 빌드하기 위한 가이드입니다. Python 패키지 또는 R 패키지를 빌드하려면 각각 `Python-package`_와 `R-package`_ 폴더를 참조하십시오.

다음의 모든 지침은 64비트 버전의 LightGBM을 컴파일하기 위한 것입니다.
32비트 버전은 환경 제약이 있는 매우 드문 특수한 경우에만 컴파일할 필요가 있습니다.
32비트 버전은 속도가 느리고 검증되지 않았으므로 사용자의 책임하에 사용해야 하며, 설치 시 아래 명령어 중 일부를 조정하는 것을 잊지 마십시오.

공유 라이브러리 대신 정적 라이브러리를 빌드하려면 CMake 플래그에 ``-DBUILD_STATIC_LIB=ON``을 추가하면 됩니다.

벤치마킹을 수행하고자 할 경우, 사용자는 CMake 플래그에 ``-DUSE_TIMETAG=ON``을 추가하여 다양한 내부 루틴에 대한 LightGBM 출력 시간 비용(output time costs)을 만들 수 있습니다.

디버그 모드에서 LightGBM을 빌드할 수 있습니다. 이 모드에서는 모든 컴파일러 최적화가 비활성화되고 LightGBM이 내부적으로 더 많은 검사를 수행합니다. 디버그 모드를 활성화하려면 CMake 플래그(flags)에 ``-DUSE_DEBUG=ON``을 추가하거나 LightGBM 빌드 방법에 따라 Visual Studio에서 ``Debug_*`` 설정(예: ``Debug_DLL``, ``Debug_mpi``) 중에서 선택하십시오.

.. _sanitizers:

디버그 모드 이외에도 컴파일러 검사기(sanitizer)를 사용하여 LightGBM을 빌드할 수 있습니다.
이를 활성화하려면 CMake 플래그에 ``-DUSE_SANITIZER=ON -DENABLED_SANITIZERS="address;leak;undefined"``를 추가합니다.
이 파라미터들은 다음과 같은 검사기를 지칭합니다:

- ``address`` - 주소 검사기(AddressSanitizer; ASan);
- ``leak`` - 누수 검사기(LeakSanitizer; LSan);
- ``undefined`` - 정의되지 않은 동작 검사기(UndefinedBehaviorSanitizer; UBSan);
- ``thread`` - 스레드 검사기(ThreadSanitizer; TSan).

스레드 검사기는 다른 검사기와 함께 사용할 수 없다는 점에 유의하십시오.
자세한 내용 및 기타 검사기에 대한 파라미터는 `_following docs`_를 참조하세요.
검사기를 사용하여 ``C++ 유닛 테스트 <#build-c-unit-tests>`__를 빌드하는 것은 매우 유용합니다.

.. _nightly-builds:

마스터 브랜치(master branch)에서 가장 최근에 성공한 빌드(개발자 버전)의 아티팩트를 여기에서 다운로드할 수도 있습니다: |download artifacts|.

.. contents:: **Contents**
    :depth: 1
    :local:
    :backlinks: none

Windows
~~~~~~~~~~

Windows에서 LightGBM은 다음과 같은 방법으로 빌드할 수 있습니다.

- **Visual Studio**;

- **CMake** 및 **VS Build Tools**;

- **CMake** 및 **MinGW**.

Visual Studio (또는 VS Build Tools)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

GUI 활용
********

1. `Visual Studio`_(2015 버전 이상)를 설치합니다.

2. `zip archive`_ 파일을 다운로드하고 압축을 풉니다.

3. ``LightGBM-master/windows`` 폴더로 이동합니다.

4. **Visual Studio** 로 ``LightGBM.sln`` 파일을 열고 ``Release`` 설정을 선택한 후 ``BUILD`` 클릭 -> ``솔루션 빌드(Ctrl+Shift+B)``

   **플랫폼 툴셋(Platform Toolset)** 에 오류가 있는 경우 ``PROJECT`` -> ``Properties`` -> ``Configuration Properties`` -> ``General`` 로 이동하여 사용 중인 머신에 설치된 툴셋을 선택합니다.

``.exe`` 파일이 ``LightGBM-master/windows/x64/Release`` 폴더에 저장됩니다.

명령줄(Command Line) 활용
*****************

1. Windows용 Git(`Git for Windows`_), `CMake`_(3.8 버전 이상), `VS Build Tools`_ 를 설치합니다(**Visual Studio**(2015 버전 이상)가 이미 설치되어 있는 경우 **VS Build Tools** 는 설치할 필요 없음).

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 ..
     cmake --build . --target ALL_BUILD --config Release

``.exe`` 및 ``.dll`` 파일이 ``LightGBM/`` 폴더에 저장됩니다.

MinGW-w64
^^^^^^^^^

1. Windows용 Git(`Git for Windows`_), `CMake`_, `MinGW-w64`_ 를 설치합니다.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -G "MinGW Makefiles" ..
     mingw32-make.exe -j4

``.exe`` 및 ``.dll`` 파일이 ``LightGBM/`` 폴더에 저장됩니다.

**주의**: ``sh.exe was found in your PATH`` 오류가 발생하면 ``cmake -G "MinGW Makefiles" ..`` 를 한 번 더 실행해야 할 수 있습니다.

멀티코어 시스템에서는 **Windows**의 멀티스레딩 효율이 더 좋으므로 **Visual Studio**를 사용하는 것이 좋습니다
(`Question 4 <./FAQ.rst#i-am-using-windows-should-i-use-visual-studio-or-mingw-for-compiling-lightgbm>`__ 및 `Question 8 <./FAQ.rst#cpu-usage-is-low-like-10-in-windows-when-using-lightgbm-on-very-large-datasets-with-many-core-systems>`__ 참조).

또한 `gcc Tips <./gcc-Tips.rst>`__ 를 읽어보시기 바랍니다.

Linux
~~~~~~~~

Linux에서 LightGBM은 **CMake**, 그리고 **gcc** 또는 **Clang** 를 사용하여 빌드할 수 있습니다.

1. `CMake`_ 를 설치합니다.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake ..
     make -j4

**주의**: glibc(2.14 버전 이상)가 필요합니다.

**주의**: 드물지만 OpenMP 런타임 라이브러리를 별도로 설치해야 하는 경우도 있습니다(패키지 관리자를 이용해 ``lib[g|i]omp`` 를 검색하여 이 작업을 수행할 수 있습니다).

또한 `gcc Tips <./gcc-Tips.rst>`__ 를 읽어보십시오.

macOS
~~~~~

macOS에서 LightGBM은 **Homebrew**를 사용하여 설치하거나, **CMake**, 그리고 **Apple Clang** 또는 **gcc**를 사용하여 빌드할 수 있습니다.

Apple Clang
^^^^^^^^^^^

**Apple Clang**(8.1 버전 이상)만 지원됩니다.

``Homebrew`` 를 사용하여 설치하기
**************************

.. code::

  brew install lightgbm

GitHub에서 빌드
*****************

1. `CMake`_(3.16 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. **OpenMP** 를 설치합니다:

   .. code::

     brew install libomp

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake ..
     make -j4

gcc
^^^

1. `CMake`_(3.2 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. **gcc** 를 설치합니다:

   .. code::

     brew install gcc

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     export CXX=g++-7 CC=gcc-7  # replace "7" with version of gcc installed on your machine
     mkdir build
     cd build
     cmake ..
     make -j4

또한 `gcc Tips <./gcc-Tips.rst>`__ 를 읽어보십시오.

Docker
~~~~~~

`Docker folder <https://github.com/microsoft/LightGBM/tree/master/docker>`__ 를 확인하십시오.

쓰레드 지원이 없는 버전 빌드하기(권장되지 않음)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LightGBM의 기본 빌드 버전은 OpenMP를 기반으로 합니다.
OpenMP를 지원하지 않고도 LightGBM을 빌드할 수 있지만 **강력히 권장하지 않습니다**.

Windows
^^^^^^^

Windows에서 OpenMP를 지원하지 않는 LightGBM 버전은 다음을 사용하여 빌드할 수 있습니다.

- **Visual Studio**;

- **CMake** 및 **VS Build Tools**;

- **CMake** 및 **MinGW**.

Visual Studio (또는 VS Build Tools)
*********************************

GUI 활용
--------

1. `Visual Studio`_(2015 버전 이상)를 설치합니다.

2. `zip archive`_ 파일을 다운로드하고 압축을 풉니다.

3. ``LightGBM-master/windows`` 폴더로 이동합니다.

4. **Visual Studio** 로 ``LightGBM.sln`` 파일을 엽니다.

5. ``PROJECT`` -> ``Properties`` -> ``Configuration Properties`` -> ``C/C++`` -> ``Language`` 로 이동 후, ``OpenMP Support`` 속성을 ``No (/openmp-)`` 로 변경하십시오.

6. 프로젝트의 메인 화면으로 돌아와 ``Release`` 구성을 선택한 다음 ``BUILD`` -> ``Build Solution (Ctrl+Shift+B)`` 를 클릭합니다.

   **플랫폼 툴셋(Platform Toolset)** 에 오류가 있는 경우 ``PROJECT`` -> ``Properties`` -> ``Configuration Properties`` -> ``General`` 로 이동하여 사용 중인 머신에 설치된 툴셋을 선택합니다.

``.exe`` 파일이 ``LightGBM-master/windows/x64/Release`` 폴더에 저장됩니다.

명령줄(Command Line) 활용
-----------------

1. Windows용 Git(`Git for Windows`_), `CMake`_(3.8 버전 이상), `VS Build Tools`_ 를 설치합니다(**Visual Studio**(2015 버전 이상)가 이미 설치되어 있는 경우 **VS Build Tools** 는 설치할 필요 없음).

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DUSE_OPENMP=OFF ..
     cmake --build . --target ALL_BUILD --config Release

``.exe`` 및 ``.dll`` 파일이 ``LightGBM/`` 폴더에 저장됩니다.

MinGW-w64
*********

1. Windows용 Git(`Git for Windows`_), `CMake`_, `MinGW-w64`_ 를 설치합니다.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -G "MinGW Makefiles" -DUSE_OPENMP=OFF ..
     mingw32-make.exe -j4

``.exe`` 및 ``.dll`` 파일이 ``LightGBM/`` 폴더에 저장됩니다.

**주의**: ``sh.exe was found in your PATH`` 오류가 발생하면 ``cmake -G "MinGW Makefiles" ..`` 를 한 번 더 실행해야 할 수 있습니다.

Linux
^^^^^

Linux에서 OpenMP를 지원하지 않는 LightGBM 버전은 **CMake**, 그리고 **gcc** 또는 **Clang** 를 사용하여 빌드할 수 있습니다.

1. `CMake`_ 를 설치합니다.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_OPENMP=OFF ..
     make -j4

**주의**: glibc(2.14 버전 이상)가 필요합니다.

macOS
^^^^^

macOS에서 OpenMP를 지원하지 않는 LightGBM 버전은 **CMake**, 그리고 **Apple Clang** 또는 **gcc** 를 사용하여 빌드할 수 있습니다.

Apple Clang
***********

**Apple Clang**(8.1 버전 이상)만 지원됩니다.

1. `CMake`_(3.16 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_OPENMP=OFF ..
     make -j4

gcc
***

1. `CMake`_(3.2 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. **gcc** 를 설치합니다:

   .. code::

     brew install gcc

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     export CXX=g++-7 CC=gcc-7  # replace "7" with version of gcc installed on your machine
     mkdir build
     cd build
     cmake -DUSE_OPENMP=OFF ..
     make -j4

MPI 버전 빌드하기
~~~~~~~~~~~~~~~~~

LightGBM의 기본 빌드 버전은 소켓(socket)을 기반으로 합니다. LightGBM은 MPI도 지원합니다.
`MPI`_ 는 `RDMA`_ 를 지원하는 고성능 통신 방식입니다.

고성능 통신으로 분산 학습 애플리케이션을 구동해야 하는 경우, LightGBM의 MPI 버전을 빌드할 수 있습니다.

Windows
^^^^^^^

Windows에서 LightGBM의 MPI 버전은 다음을 사용하여 빌드할 수 있습니다.

- **MS MPI** 및 **Visual Studio**;

- **MS MPI**, **CMake** 및 **VS Build Tools**.

GUI 활용
********

1. 1. 먼저 `MS MPI`_ 를 설치해야 합니다. ``msmpisdk.msi`` 와 ``msmpisetup.exe`` 둘 다 필요합니다.

2. `Visual Studio`_(2015 버전 이상)를 설치합니다.

3. `zip archive`_ 파일을 다운로드하고 압축을 풉니다.

4. ``LightGBM-master/windows`` 폴더로 이동합니다.

5. **Visual Studio** 로 ``LightGBM.sln`` 파일을 열고 ``Release`` 설정을 선택한 후 ``BUILD`` 클릭 -> ``솔루션 빌드(Ctrl+Shift+B)``

   **플랫폼 툴셋(Platform Toolset)** 에 오류가 있는 경우 ``PROJECT`` -> ``Properties`` -> ``Configuration Properties`` -> ``General`` 로 이동하여 사용 중인 머신에 설치된 툴셋을 선택합니다.

``.exe`` 파일이 ``LightGBM-master/windows/x64/Release_mpi`` 폴더에 저장됩니다.

명령줄(Command Line) 활용
*****************

1. 1. 먼저 `MS MPI`_ 를 설치해야 합니다. ``msmpisdk.msi`` 와 ``msmpisetup.exe`` 둘 다 필요합니다.

2. Windows용 Git(`Git for Windows`_), `CMake`_(3.8 버전 이상), `VS Build Tools`_ 를 설치합니다(**Visual Studio**(2015 버전 이상)가 이미 설치되어 있는 경우 **VS Build Tools** 는 설치할 필요 없음).

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DUSE_MPI=ON ..
     cmake --build . --target ALL_BUILD --config Release

``.exe`` 및 ``.dll`` 파일이 ``LightGBM/Release`` 폴더에 저장됩니다.

**MinGW** 의 MPI 버전을 빌드하는 것은 MPI 라이브러리의 부재로 지원되지 않습니다.

Linux
^^^^^

Linux에서 LightGBM의 MPI 버전은 **Open MPI**, **CMake**, 그리고 **gcc** 또는 **Clang** 를 사용하여 빌드할 수 있습니다.

1. `Open MPI`_ 를 설치합니다.

2. `CMake`_ 를 설치합니다.

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_MPI=ON ..
     make -j4

**주의**: glibc(2.14 버전 이상)가 필요합니다.

**주의**: 드물지만 OpenMP 런타임 라이브러리를 별도로 설치해야 하는 경우도 있습니다(패키지 관리자를 이용해 ``lib[g|i]omp`` 를 검색하여 이 작업을 수행할 수 있습니다).

macOS
^^^^^

macOS에서 LightGBM의 MPI 버전은 **Open MPI**, **CMake**, 그리고 **Apple Clang** 또는 **gcc**를 사용하여 빌드할 수 있습니다.

Apple Clang
***********

**Apple Clang**(8.1 버전 이상)만 지원됩니다.

1. `CMake`_(3.16 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. **OpenMP** 를 설치합니다:

   .. code::

     brew install libomp

3. **Open MPI** 를 설치합니다:

   .. code::

     brew install open-mpi

4. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_MPI=ON ..
     make -j4

gcc
***

1. `CMake`_(3.2 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. **gcc** 를 설치합니다:

   .. code::

     brew install gcc

3. **Open MPI** 를 설치합니다:

   .. code::

     brew install open-mpi

4. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     export CXX=g++-7 CC=gcc-7  # replace "7" with version of gcc installed on your machine
     mkdir build
     cd build
     cmake -DUSE_MPI=ON ..
     make -j4

GPU 버전 빌드하기
~~~~~~~~~~~~~~~~~

Linux
^^^^^

Linux에서 LightGBM의 GPU 버전(``device_type=gpu``)은 **OpenCL**, **Boost**, **CMake**, 그리고 **gcc** 또는 **Clang** 을 사용하여 빌드할 수 있습니다.

컴파일하기 전에 다음 종속 요소들(dependencies)을 설치해야 합니다:

-  **OpenCL** 1.2 헤더 및 라이브러리(대개 GPU 제조사에서 제공).

   일반적인 OpenCL ICD 패키지(예를 들어, 데비안 패키지 ``ocl-icd-libopencl1`` 및 ``ocl-icd-opencl-dev``)도 사용할 수 있습니다.

-  **libboost** 1.56 버전 이상(1.61 버전 이상 권장).

   1.61 버전부터 Boost 라이브러리의 일부인 Boost.Compute를 GPU 인터페이스로 사용하고 있습니다. 하지만 Boost.Compute의 소스 코드를 하위모듈로 포함하기 때문에 호스트에 1.56 버전 이상만 설치되어 있으면 됩니다. 또한 메모리 할당을 위해 Boost.Align을 사용합니다. Boost.Compute는 오프라인 커널 캐시를 저장하기 위해 Boost.System 및 Boost.Filesystem이 필요합니다.

   다음의 데비안 패키지는 필요한 Boost 라이브러리를 제공해야 합니다: ``libboost-dev``, ``libboost-system-dev``, ``libboost-filesystem-dev``.

-  **CMake**(3.2 버전 이상).

LightGBM의 GPU 버전을 빌드하기 위해 다음 명령을 실행합니다:

.. code::

  git clone --recursive https://github.com/microsoft/LightGBM
  cd LightGBM
  mkdir build
  cd build
  cmake -DUSE_GPU=1 ..
  # if you have installed NVIDIA CUDA to a customized location, you should specify paths to OpenCL headers and library like the following:
  # cmake -DUSE_GPU=1 -DOpenCL_LIBRARY=/usr/local/cuda/lib64/libOpenCL.so -DOpenCL_INCLUDE_DIR=/usr/local/cuda/include/ ..
  make -j4

**주의**: glibc(2.14 버전 이상)가 필요합니다.

**주의**: 드물지만 OpenMP 런타임 라이브러리를 별도로 설치해야 하는 경우도 있습니다(패키지 관리자를 이용해 ``lib[g|i]omp`` 를 검색하여 이 작업을 수행할 수 있습니다).

Windows
^^^^^^^

Windows에서 LightGBM의 GPU 버전(``device_type=gpu``)은 **OpenCL**, **Boost**, **CMake**, 그리고 **VS Build Tools** 또는 **MinGW** 를 사용하여 빌드할 수 있습니다.

**MinGW** 를 사용하는 경우, 빌드 절차는 Linux의 빌드와 유사합니다. 자세한 내용은 `GPU Windows Compilation <./GPU-Windows.rst>`__ 를 참조하십시오.

다음은 **MSVC**(Microsoft Visual C++) 빌드에 대한 절차입니다.

1. Windows용 Git(`Git for Windows`_), `CMake`_(3.8 버전 이상), `VS Build Tools`_ 를 설치합니다(**Visual Studio**(2015 버전 이상)가 이미 설치되어 있는 경우 **VS Build Tools** 는 설치할 필요 없음).

2. Windows용 **OpenCL** 을 설치합니다. 설치 방법은 GPU 카드의 브랜드(NVIDIA, AMD, Intel)에 따라 다릅니다.

   - Intel의 경우, `Intel SDK for OpenCL`_ 를 다운로드하세요.

   - AMD의 경우, AMD APP SDK를 다운로드하세요.

   - NVIDIA의 경우, `CUDA Toolkit`_ 을 다운로드하세요.

   추가 내용 및 대응 표: `GPU SDK Correspondence and Device Targeting Table <./GPU-Targets.rst>`__.

3. `Boost Binaries`_ 를 설치합니다.

   **주의**: 사용중인 Visual C++ 버전에 맞춰 설치하십시오:
   
   Visual Studio 2015 -> ``msvc-14.0-64.exe``,

   Visual Studio 2017 -> ``msvc-14.1-64.exe``,

   Visual Studio 2019 -> ``msvc-14.2-64.exe``.

4. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DUSE_GPU=1 -DBOOST_ROOT=C:/local/boost_1_63_0 -DBOOST_LIBRARYDIR=C:/local/boost_1_63_0/lib64-msvc-14.0 ..
     # if you have installed NVIDIA CUDA to a customized location, you should specify paths to OpenCL headers and library like the following:
     # cmake -A x64 -DUSE_GPU=1 -DBOOST_ROOT=C:/local/boost_1_63_0 -DBOOST_LIBRARYDIR=C:/local/boost_1_63_0/lib64-msvc-14.0 -DOpenCL_LIBRARY="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v10.0/lib/x64/OpenCL.lib" -DOpenCL_INCLUDE_DIR="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v10.0/include" ..
     cmake --build . --target ALL_BUILD --config Release

   **주의**: ``C:/local/boost_1_63_0`` 및 ``C:/local/boost_1_63_0/lib64-msvc-14.0`` 은 **Boost** 바이너리의 위치입니다(Visual Studio 2015용 1.63.0 버전을 다운로드 했을 때를 가정).

Docker
^^^^^^

`GPU Docker folder <https://github.com/microsoft/LightGBM/tree/master/docker/gpu>`__ 를 참조하세요.

CUDA 버전 빌드하기 (실험단계)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LightGBM(``device_type=gpu``)의 원래 빌드(`original GPU build <#build-gpu-version>`__)는 OpenCL을 기반으로 합니다.

CUDA 기반 빌드(``device_type=cuda``)는 별개의 실행 방식이며, capability 6.0 이상의 NVIDIA 그래픽 카드를 필요로 합니다. 이 빌드는 실험단계로 간주해야하며, OpenCL 버전을 사용할 수 없는 경우(예: IBM POWER 마이크로프로세서)에만 사용하는 것이 좋습니다.

**주의**: Linux만 지원되며, 다른 운영체제는 아직 지원하지 않습니다.

Linux
^^^^^

Linux에서 LightGBM의 CUDA 버전은 **CUDA**, **CMake**, 그리고 **gcc** 또는 **Clang** 를 사용하여 빌드할 수 있습니다.

컴파일하기 전에 다음 종속 요소들(dependencies)을 설치해야 합니다:

-  **CUDA** 9.0 버전 이상의 라이브러리. `this detailed guide`_ 를 참조하십시오. 해당 가이드의 표에 나열된 호스트 컴파일러의 최소 요구 버전에 주의를 기울이고 권장 버전의 컴파일러만 사용하십시오.

-  **CMake**(3.16 버전 이상).

LightGBM의 CUDA 버전을 빌드하기 위해 다음 명령을 실행합니다:

.. code::

  git clone --recursive https://github.com/microsoft/LightGBM
  cd LightGBM
  mkdir build
  cd build
  cmake -DUSE_CUDA=1 ..
  make -j4

**주의**: glibc(2.14 버전 이상)가 필요합니다.

**주의**: 드물지만 OpenMP 런타임 라이브러리를 별도로 설치해야 하는 경우도 있습니다(패키지 관리자를 이용해 ``lib[g|i]omp`` 를 검색하여 이 작업을 수행할 수 있습니다).

HDFS 버전 빌드하기
~~~~~~~~~~~~~~~~~~

LightGBM의 HDFS 버전은 CDH-5.14.4 클러스터에서 테스트되었습니다.

Linux
^^^^^

Linux에서 LightGBM의 HDFS 버전은 **CMake** 및 **gcc** 를 사용하여 빌드할 수 있습니다.

1. `CMake`_ 를 설치합니다.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_HDFS=ON ..
     # if you have installed HDFS to a customized location, you should specify paths to HDFS headers (hdfs.h) and library (libhdfs.so) like the following:
     # cmake \
     #   -DUSE_HDFS=ON \
     #   -DHDFS_LIB="/opt/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/lib64/libhdfs.so" \
     #   -DHDFS_INCLUDE_DIR="/opt/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/include/" \
     #   ..
     make -j4

**주의**: glibc(2.14 버전 이상)가 필요합니다.

**주의**: 드물지만 OpenMP 런타임 라이브러리를 별도로 설치해야 하는 경우도 있습니다(패키지 관리자를 이용해 ``lib[g|i]omp`` 를 검색하여 이 작업을 수행할 수 있습니다).

Java 래퍼(Wrapper) 빌드
~~~~~~~~~~~~~~~~~~

다음 지침에 따라 **SWIG** 로 래핑된 LightGBM `C API <./Development-Guide.rst#c-api>`__ 가 포함된 JAR 파일을 생성할 수 있습니다.

Windows
^^^^^^^

Windows에서 LightGBM의 Java 래퍼(Wrapper)는 **Java**, **SWIG**, **CMake**, 그리고 **VS Build Tools** 또는 **MinGW** 를 사용하여 빌드할 수 있습니다.

VS Build Tools
**************

1. Windows용 Git(`Git for Windows`_), `CMake`_(3.8 버전 이상), `VS Build Tools`_ 를 설치합니다(**Visual Studio**(2015 버전 이상)가 이미 설치되어 있는 경우 **VS Build Tools** 는 설치할 필요 없음).

2. `SWIG`_ 및 **Java** 를 설치합니다(또한 ``JAVA_HOME`` 이 올바르게 설정되었는지 확인합니다).

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DUSE_SWIG=ON ..
     cmake --build . --target ALL_BUILD --config Release

``.jar`` 파일은 ``LightGBM/build`` 폴더에, ``.dll`` 파일은 ``LightGBM/Release`` 폴더에 저장됩니다.

MinGW-w64
*********

1. Windows용 Git(`Git for Windows`_), `CMake`_, `MinGW-w64`_ 를 설치합니다.

2. `SWIG`_ 및 **Java** 를 설치합니다(또한 ``JAVA_HOME`` 이 올바르게 설정되었는지 확인합니다).

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -G "MinGW Makefiles" -DUSE_SWIG=ON ..
     mingw32-make.exe -j4

``.jar`` 파일은 ``LightGBM/build`` 폴더에, ``.dll`` 파일은 ``LightGBM/`` 폴더에 저장됩니다.

**주의**: ``sh.exe was found in your PATH`` 오류가 발생하면 ``cmake -G "MinGW Makefiles" -DUSE_SWIG=ON ..`` 를 한 번 더 실행해야 할 수 있습니다.

멀티 코어 시스템의 경우 **Windows** 에서 멀티 스레딩 효율이 더 좋으므로 **VS Build Tools (Visual Studio)** 를 사용하는 것이 좋습니다(`Question 4 <./FAQ.rst#i-am-using-windows-should-i-use-visual-studio-or-mingw-for-compiling-lightgbm>`__ and `Question 8 <./FAQ.rst#cpu-usage-is-low-like-10-in-windows-when-using-lightgbm-on-very-large-datasets-with-many-core-systems>`__ 를 확인하십시오).

또한 `gcc Tips <./gcc-Tips.rst>`__ 를 읽어보십시오.

Linux
^^^^^

Linux에서 LightGBM의 Java 래퍼(Wrapper)는 **Java**, **SWIG**, **CMake**, 그리고 **gcc** 또는 **Clang** 를 사용하여 빌드할 수 있습니다.

1. `CMake`_, `SWIG`_ 및 **Java** 를 설치합니다(또한 ``JAVA_HOME`` 이 올바르게 설정되었는지 확인합니다).

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_SWIG=ON ..
     make -j4

**주의**: glibc(2.14 버전 이상)가 필요합니다.

**주의**: 드물지만 OpenMP 런타임 라이브러리를 별도로 설치해야 하는 경우도 있습니다(패키지 관리자를 이용해 ``lib[g|i]omp`` 를 검색하여 이 작업을 수행할 수 있습니다).

macOS
^^^^^

macOS에서 LightGBM의 Java 래퍼(Wrapper)는 **Java**, **SWIG**, **CMake**, 그리고 **Apple Clang** 또는 **gcc** 를 사용하여 빌드할 수 있습니다.

먼저 `SWIG`_ 및 **Java** 를 설치합니다(또한 ``JAVA_HOME`` 이 올바르게 설정되었는지 확인합니다).
그 후, 아래의 **Apple Clang** 또는 **gcc** 설치 지침을 따르세요.

Apple Clang
***********

**Apple Clang**(8.1 버전 이상)만 지원됩니다.

1. `CMake`_(3.16 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. **OpenMP** 를 설치합니다:

   .. code::

     brew install libomp

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_SWIG=ON -DAPPLE_OUTPUT_DYLIB=ON ..
     make -j4

gcc
***

1. `CMake`_(3.2 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. **gcc** 를 설치합니다:

   .. code::

     brew install gcc

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     export CXX=g++-7 CC=gcc-7  # replace "7" with version of gcc installed on your machine
     mkdir build
     cd build
     cmake -DUSE_SWIG=ON -DAPPLE_OUTPUT_DYLIB=ON ..
     make -j4

또한 `gcc Tips <./gcc-Tips.rst>`__ 를 읽어보십시오.

C++ 유닛 테스트 빌드하기
~~~~~~~~~~~~~~~~~~~~

Windows
^^^^^^^

Windows에서 LightGBM의 C++ 유닛 테스트는 **CMake** 및 **VS Build Tools** 를 사용하여 빌드할 수 있습니다.

1. Windows용 Git(`Git for Windows`_), `CMake`_(3.8 버전 이상), `VS Build Tools`_ 를 설치합니다(**Visual Studio**(2015 버전 이상)가 이미 설치되어 있는 경우 **VS Build Tools** 는 설치할 필요 없음).

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DBUILD_CPP_TEST=ON -DUSE_OPENMP=OFF ..
     cmake --build . --target testlightgbm --config Debug

``.exe`` 파일이 ``LightGBM/Debug`` 폴더에 저장됩니다.

Linux
^^^^^

Linux에서 LightGBM의 C++ 유닛 테스트는 **CMake**, 그리고 **gcc** 또는 **Clang**.

1. `CMake`_ 를 설치합니다.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DBUILD_CPP_TEST=ON -DUSE_OPENMP=OFF ..
     make testlightgbm -j4

**주의**: glibc(2.14 버전 이상)가 필요합니다.

macOS
^^^^^

macOS에서 LightGBM의 C++ 유닛 테스트는 **CMake**, 그리고 **Apple Clang** 또는 **gcc**.

Apple Clang
***********

**Apple Clang**(8.1 버전 이상)만 지원됩니다.

1. `CMake`_(3.16 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DBUILD_CPP_TEST=ON -DUSE_OPENMP=OFF ..
     make testlightgbm -j4

gcc
***

1. `CMake`_(3.2 버전 이상)을 설치합니다:

   .. code::

     brew install cmake

2. **gcc** 를 설치합니다:

   .. code::

     brew install gcc

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     export CXX=g++-7 CC=gcc-7  # replace "7" with version of gcc installed on your machine
     mkdir build
     cd build
     cmake -DBUILD_CPP_TEST=ON -DUSE_OPENMP=OFF ..
     make testlightgbm -j4


.. |download artifacts| image:: ./_static/images/artifacts-not-available.svg
   :target: https://lightgbm.readthedocs.io/en/latest/Installation-Guide.html

.. _Python-package: https://github.com/microsoft/LightGBM/tree/master/python-package

.. _R-package: https://github.com/microsoft/LightGBM/tree/master/R-package

.. _zip archive: https://github.com/microsoft/LightGBM/archive/master.zip

.. _Visual Studio: https://visualstudio.microsoft.com/downloads/

.. _Git for Windows: https://git-scm.com/download/win

.. _CMake: https://cmake.org/

.. _VS Build Tools: https://visualstudio.microsoft.com/downloads/

.. _MinGW-w64: https://www.mingw-w64.org/downloads/

.. _MPI: https://en.wikipedia.org/wiki/Message_Passing_Interface

.. _RDMA: https://en.wikipedia.org/wiki/Remote_direct_memory_access

.. _MS MPI: https://docs.microsoft.com/en-us/message-passing-interface/microsoft-mpi-release-notes

.. _Open MPI: https://www.open-mpi.org/

.. _Intel SDK for OpenCL: https://software.intel.com/en-us/articles/opencl-drivers

.. _CUDA Toolkit: https://developer.nvidia.com/cuda-downloads

.. _Boost Binaries: https://sourceforge.net/projects/boost/files/boost-binaries/

.. _SWIG: http://www.swig.org/download.html

.. _this detailed guide: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html

.. _following docs: https://github.com/google/sanitizers/wiki
