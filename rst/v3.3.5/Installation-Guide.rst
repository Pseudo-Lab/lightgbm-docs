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

Windows 버전
~~~~~~~~~~

Windows에서 LightGBM은 다음과 같은 방법으로 빌드할 수 있습니다.

- **Visual Studio**;

- **CMake** 및 **VS Build Tools**;

- **CMake** 및 **MinGW**.

Visual Studio (또는 VS Build Tools)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

GUI 활용
********

1. `Visual Studio`_ (2015 버전 이상)를 설치합니다.

2. `zip archive`_ 파일을 다운로드하고 압축을 풉니다.

3. ``LightGBM-master/windows`` 폴더로 이동합니다.

4. **Visual Studio** 로 ``LightGBM.sln`` 파일을 열고 ``Release`` 설정을 선택한 후 ``BUILD`` 를 클릭합니다. -> ``솔루션 빌드(Ctrl+Shift+B)``

   **플랫폼 툴셋(Platform Toolset)** 에 오류가 있는 경우 ``PROJECT`` -> ``Properties`` -> ``Configuration Properties`` -> ``General`` 로 이동하여 사용 중인 머신에 설치된 툴셋을 선택합니다.

``.exe`` 파일은 ``LightGBM-master/windows/x64/Release`` 폴더에 있습니다.

명령줄(Command Line) 활용
*****************

1. Windows용 Git(`Git for Windows`_), `CMake`_ (3.8 버전 이상), `VS Build Tools`_ 를 설치합니다(**Visual Studio** (2015 버전 이상)가 이미 설치되어 있는 경우 **VS Build Tools**는 설치할 필요 없음).

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 ..
     cmake --build . --target ALL_BUILD --config Release

``.exe`` 및 ``.dll`` 파일이 ``LightGBM/`` 폴더에 생성될 것입니다.

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

``.exe`` 및 ``.dll`` 파일이 ``LightGBM/`` 폴더에 생성될 것입니다.

**참고**: ``sh.exe was found in your PATH`` 오류가 발생하면 ``cmake -G "MinGW Makefiles" ..`` 를 한 번 더 실행해야 할 수 있습니다.

멀티코어 시스템에서는 **Windows**의 멀티스레딩 효율이 더 좋으므로 **Visual Studio**를 사용하는 것이 좋습니다
(`Question 4 <./FAQ.rst#i-am-using-windows-should-i-use-visual-studio-or-mingw-for-compiling-lightgbm>`__ 및 `Question 8 <./FAQ.rst#cpu-usage-is-low-like-10-in-windows-when-using-lightgbm-on-very-large-datasets-with-many-core-systems>`__ 참조).

또한 `gcc Tips <./gcc-Tips.rst>`__ 를 읽어보시기 바랍니다.

Linux 버전
~~~~~~~~

On Linux LightGBM can be built using **CMake** and **gcc** or **Clang**.

1. Install `CMake`_.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake ..
     make -j4

**Note**: glibc >= 2.14 is required.

**Note**: In some rare cases you may need to install OpenMP runtime library separately (use your package manager and search for ``lib[g|i]omp`` for doing this).

Also, you may want to read `gcc Tips <./gcc-Tips.rst>`__.

macOS
~~~~~

On macOS LightGBM can be installed using **Homebrew**, or can be built using **CMake** and **Apple Clang** or **gcc**.

Apple Clang
^^^^^^^^^^^

Only **Apple Clang** version 8.1 or higher is supported.

Install Using ``Homebrew``
**************************

.. code::

  brew install lightgbm

Build from GitHub
*****************

1. Install `CMake`_ (3.16 or higher):

   .. code::

     brew install cmake

2. Install **OpenMP**:

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

1. Install `CMake`_ (3.2 or higher):

   .. code::

     brew install cmake

2. Install **gcc**:

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

Also, you may want to read `gcc Tips <./gcc-Tips.rst>`__.

Docker
~~~~~~

Refer to `Docker folder <https://github.com/microsoft/LightGBM/tree/master/docker>`__.

Build Threadless Version (not Recommended)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The default build version of LightGBM is based on OpenMP.
You can build LightGBM without OpenMP support but it is **strongly not recommended**.

Windows
^^^^^^^

On Windows a version of LightGBM without OpenMP support can be built using

- **Visual Studio**;

- **CMake** and **VS Build Tools**;

- **CMake** and **MinGW**.

Visual Studio (or VS Build Tools)
*********************************

With GUI
--------

1. Install `Visual Studio`_ (2015 or newer).

2. Download `zip archive`_ and unzip it.

3. Go to ``LightGBM-master/windows`` folder.

4. Open ``LightGBM.sln`` file with **Visual Studio**.

5. Go to ``PROJECT`` -> ``Properties`` -> ``Configuration Properties`` -> ``C/C++`` -> ``Language`` and change the ``OpenMP Support`` property to ``No (/openmp-)``.

6. Get back to the project's main screen, then choose ``Release`` configuration and click ``BUILD`` -> ``Build Solution (Ctrl+Shift+B)``.

   If you have errors about **Platform Toolset**, go to ``PROJECT`` -> ``Properties`` -> ``Configuration Properties`` -> ``General`` and select the toolset installed on your machine.

The ``.exe`` file will be in ``LightGBM-master/windows/x64/Release`` folder.

From Command Line
-----------------

1. Install `Git for Windows`_, `CMake`_ (3.8 or higher) and `VS Build Tools`_ (**VS Build Tools** is not needed if **Visual Studio** (2015 or newer) is already installed).

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DUSE_OPENMP=OFF ..
     cmake --build . --target ALL_BUILD --config Release

The ``.exe`` and ``.dll`` files will be in ``LightGBM/Release`` folder.

MinGW-w64
*********

1. Install `Git for Windows`_, `CMake`_ and `MinGW-w64`_.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -G "MinGW Makefiles" -DUSE_OPENMP=OFF ..
     mingw32-make.exe -j4

The ``.exe`` and ``.dll`` files will be in ``LightGBM/`` folder.

**Note**: You may need to run the ``cmake -G "MinGW Makefiles" -DUSE_OPENMP=OFF ..`` one more time if you encounter the ``sh.exe was found in your PATH`` error.

Linux
^^^^^

On Linux a version of LightGBM without OpenMP support can be built using **CMake** and **gcc** or **Clang**.

1. Install `CMake`_.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_OPENMP=OFF ..
     make -j4

**Note**: glibc >= 2.14 is required.

macOS
^^^^^

On macOS a version of LightGBM without OpenMP support can be built using **CMake** and **Apple Clang** or **gcc**.

Apple Clang
***********

Only **Apple Clang** version 8.1 or higher is supported.

1. Install `CMake`_ (3.16 or higher):

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

1. Install `CMake`_ (3.2 or higher):

   .. code::

     brew install cmake

2. Install **gcc**:

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

Build MPI Version
~~~~~~~~~~~~~~~~~

The default build version of LightGBM is based on socket. LightGBM also supports MPI.
`MPI`_ is a high performance communication approach with `RDMA`_ support.

If you need to run a distributed learning application with high performance communication, you can build the LightGBM with MPI support.

Windows
^^^^^^^

On Windows an MPI version of LightGBM can be built using

- **MS MPI** and **Visual Studio**;

- **MS MPI**, **CMake** and **VS Build Tools**.

With GUI
********

1. You need to install `MS MPI`_ first. Both ``msmpisdk.msi`` and ``msmpisetup.exe`` are needed.

2. Install `Visual Studio`_ (2015 or newer).

3. Download `zip archive`_ and unzip it.

4. Go to ``LightGBM-master/windows`` folder.

5. Open ``LightGBM.sln`` file with **Visual Studio**, choose ``Release_mpi`` configuration and click ``BUILD`` -> ``Build Solution (Ctrl+Shift+B)``.

   If you have errors about **Platform Toolset**, go to ``PROJECT`` -> ``Properties`` -> ``Configuration Properties`` -> ``General`` and select the toolset installed on your machine.

The ``.exe`` file will be in ``LightGBM-master/windows/x64/Release_mpi`` folder.

From Command Line
*****************

1. You need to install `MS MPI`_ first. Both ``msmpisdk.msi`` and ``msmpisetup.exe`` are needed.

2. Install `Git for Windows`_, `CMake`_ (3.8 or higher) and `VS Build Tools`_ (**VS Build Tools** is not needed if **Visual Studio** (2015 or newer) is already installed).

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DUSE_MPI=ON ..
     cmake --build . --target ALL_BUILD --config Release

The ``.exe`` and ``.dll`` files will be in ``LightGBM/Release`` folder.

**Note**: Building MPI version by **MinGW** is not supported due to the miss of MPI library in it.

Linux
^^^^^

On Linux an MPI version of LightGBM can be built using **Open MPI**, **CMake** and **gcc** or **Clang**.

1. Install `Open MPI`_.

2. Install `CMake`_.

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_MPI=ON ..
     make -j4

**Note**: glibc >= 2.14 is required.

**Note**: In some rare cases you may need to install OpenMP runtime library separately (use your package manager and search for ``lib[g|i]omp`` for doing this).

macOS
^^^^^

On macOS an MPI version of LightGBM can be built using **Open MPI**, **CMake** and **Apple Clang** or **gcc**.

Apple Clang
***********

Only **Apple Clang** version 8.1 or higher is supported.

1. Install `CMake`_ (3.16 or higher):

   .. code::

     brew install cmake

2. Install **OpenMP**:

   .. code::

     brew install libomp

3. Install **Open MPI**:

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

1. Install `CMake`_ (3.2 or higher):

   .. code::

     brew install cmake

2. Install **gcc**:

   .. code::

     brew install gcc

3. Install **Open MPI**:

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

Build GPU Version
~~~~~~~~~~~~~~~~~

Linux
^^^^^

On Linux a GPU version of LightGBM (``device_type=gpu``) can be built using **OpenCL**, **Boost**, **CMake** and **gcc** or **Clang**.

The following dependencies should be installed before compilation:

-  **OpenCL** 1.2 headers and libraries, which is usually provided by GPU manufacture.

   The generic OpenCL ICD packages (for example, Debian package ``ocl-icd-libopencl1`` and ``ocl-icd-opencl-dev``) can also be used.

-  **libboost** 1.56 or later (1.61 or later is recommended).

   We use Boost.Compute as the interface to GPU, which is part of the Boost library since version 1.61. However, since we include the source code of Boost.Compute as a submodule, we only require the host has Boost 1.56 or later installed. We also use Boost.Align for memory allocation. Boost.Compute requires Boost.System and Boost.Filesystem to store offline kernel cache.

   The following Debian packages should provide necessary Boost libraries: ``libboost-dev``, ``libboost-system-dev``, ``libboost-filesystem-dev``.

-  **CMake** 3.2 or later.

To build LightGBM GPU version, 다음 명령을 실행합니다:

.. code::

  git clone --recursive https://github.com/microsoft/LightGBM
  cd LightGBM
  mkdir build
  cd build
  cmake -DUSE_GPU=1 ..
  # if you have installed NVIDIA CUDA to a customized location, you should specify paths to OpenCL headers and library like the following:
  # cmake -DUSE_GPU=1 -DOpenCL_LIBRARY=/usr/local/cuda/lib64/libOpenCL.so -DOpenCL_INCLUDE_DIR=/usr/local/cuda/include/ ..
  make -j4

**Note**: glibc >= 2.14 is required.

**Note**: In some rare cases you may need to install OpenMP runtime library separately (use your package manager and search for ``lib[g|i]omp`` for doing this).

Windows
^^^^^^^

On Windows a GPU version of LightGBM (``device_type=gpu``) can be built using **OpenCL**, **Boost**, **CMake** and **VS Build Tools** or **MinGW**.

If you use **MinGW**, the build procedure is similar to the build on Linux. Refer to `GPU Windows Compilation <./GPU-Windows.rst>`__ to get more details.

Following procedure is for the **MSVC** (Microsoft Visual C++) build.

1. Install `Git for Windows`_, `CMake`_ (3.8 or higher) and `VS Build Tools`_ (**VS Build Tools** is not needed if **Visual Studio** (2015 or newer) is installed).

2. Install **OpenCL** for Windows. The installation depends on the brand (NVIDIA, AMD, Intel) of your GPU card.

   - For running on Intel, get `Intel SDK for OpenCL`_.

   - For running on AMD, get AMD APP SDK.

   - For running on NVIDIA, get `CUDA Toolkit`_.

   Further reading and correspondence table: `GPU SDK Correspondence and Device Targeting Table <./GPU-Targets.rst>`__.

3. Install `Boost Binaries`_.

   **Note**: Match your Visual C++ version:
   
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

   **Note**: ``C:/local/boost_1_63_0`` and ``C:/local/boost_1_63_0/lib64-msvc-14.0`` are locations of your **Boost** binaries (assuming you've downloaded 1.63.0 version for Visual Studio 2015).

Docker
^^^^^^

Refer to `GPU Docker folder <https://github.com/microsoft/LightGBM/tree/master/docker/gpu>`__.

Build CUDA Version (Experimental)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The `original GPU build <#build-gpu-version>`__ of LightGBM (``device_type=gpu``) is based on OpenCL.

The CUDA-based build (``device_type=cuda``) is a separate implementation and requires an NVIDIA graphics card with compute capability 6.0 and higher. It should be considered experimental, and we suggest using it only when it is impossible to use OpenCL version (for example, on IBM POWER microprocessors).

**Note**: only Linux is supported, other operating systems are not supported yet.

Linux
^^^^^

On Linux a CUDA version of LightGBM can be built using **CUDA**, **CMake** and **gcc** or **Clang**.

The following dependencies should be installed before compilation:

-  **CUDA** 9.0 or later libraries. Please refer to `this detailed guide`_. Pay great attention to the minimum required versions of host compilers listed in the table from that guide and use only recommended versions of compilers.

-  **CMake** 3.16 or later.

To build LightGBM CUDA version, 다음 명령을 실행합니다:

.. code::

  git clone --recursive https://github.com/microsoft/LightGBM
  cd LightGBM
  mkdir build
  cd build
  cmake -DUSE_CUDA=1 ..
  make -j4

**Note**: glibc >= 2.14 is required.

**Note**: In some rare cases you may need to install OpenMP runtime library separately (use your package manager and search for ``lib[g|i]omp`` for doing this).

Build HDFS Version
~~~~~~~~~~~~~~~~~~

The HDFS version of LightGBM was tested on CDH-5.14.4 cluster.

Linux
^^^^^

On Linux a HDFS version of LightGBM can be built using **CMake** and **gcc**.

1. Install `CMake`_.

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

**Note**: glibc >= 2.14 is required.

**Note**: In some rare cases you may need to install OpenMP runtime library separately (use your package manager and search for ``lib[g|i]omp`` for doing this).

Build Java Wrapper
~~~~~~~~~~~~~~~~~~

Using the following instructions you can generate a JAR file containing the LightGBM `C API <./Development-Guide.rst#c-api>`__ wrapped by **SWIG**.

Windows
^^^^^^^

On Windows a Java wrapper of LightGBM can be built using **Java**, **SWIG**, **CMake** and **VS Build Tools** or **MinGW**.

VS Build Tools
**************

1. Install `Git for Windows`_, `CMake`_ (3.8 or higher) and `VS Build Tools`_ (**VS Build Tools** is not needed if **Visual Studio** (2015 or newer) is already installed).

2. Install `SWIG`_ and **Java** (also make sure that ``JAVA_HOME`` is set properly).

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DUSE_SWIG=ON ..
     cmake --build . --target ALL_BUILD --config Release

The ``.jar`` file will be in ``LightGBM/build`` folder and the ``.dll`` files will be in ``LightGBM/Release`` folder.

MinGW-w64
*********

1. Install `Git for Windows`_, `CMake`_ and `MinGW-w64`_.

2. Install `SWIG`_ and **Java** (also make sure that ``JAVA_HOME`` is set properly).

3. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -G "MinGW Makefiles" -DUSE_SWIG=ON ..
     mingw32-make.exe -j4

The ``.jar`` file will be in ``LightGBM/build`` folder and the ``.dll`` files will be in ``LightGBM/`` folder.

**Note**: You may need to run the ``cmake -G "MinGW Makefiles" -DUSE_SWIG=ON ..`` one more time if you encounter the ``sh.exe was found in your PATH`` error.

It is recommended to use **VS Build Tools (Visual Studio)** since it has better multithreading efficiency in **Windows** for many-core systems
(see `Question 4 <./FAQ.rst#i-am-using-windows-should-i-use-visual-studio-or-mingw-for-compiling-lightgbm>`__ and `Question 8 <./FAQ.rst#cpu-usage-is-low-like-10-in-windows-when-using-lightgbm-on-very-large-datasets-with-many-core-systems>`__).

Also, you may want to read `gcc Tips <./gcc-Tips.rst>`__.

Linux
^^^^^

On Linux a Java wrapper of LightGBM can be built using **Java**, **SWIG**, **CMake** and **gcc** or **Clang**.

1. Install `CMake`_, `SWIG`_ and **Java** (also make sure that ``JAVA_HOME`` is set properly).

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DUSE_SWIG=ON ..
     make -j4

**Note**: glibc >= 2.14 is required.

**Note**: In some rare cases you may need to install OpenMP runtime library separately (use your package manager and search for ``lib[g|i]omp`` for doing this).

macOS
^^^^^

On macOS a Java wrapper of LightGBM can be built using **Java**, **SWIG**, **CMake** and **Apple Clang** or **gcc**.

First, install `SWIG`_ and **Java** (also make sure that ``JAVA_HOME`` is set properly).
Then, either follow the **Apple Clang** or **gcc** installation instructions below.

Apple Clang
***********

Only **Apple Clang** version 8.1 or higher is supported.

1. Install `CMake`_ (3.16 or higher):

   .. code::

     brew install cmake

2. Install **OpenMP**:

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

1. Install `CMake`_ (3.2 or higher):

   .. code::

     brew install cmake

2. Install **gcc**:

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

Also, you may want to read `gcc Tips <./gcc-Tips.rst>`__.

Build C++ Unit Tests
~~~~~~~~~~~~~~~~~~~~

Windows
^^^^^^^

On Windows, C++ unit tests of LightGBM can be built using **CMake** and **VS Build Tools**.

1. Install `Git for Windows`_, `CMake`_ (3.8 or higher) and `VS Build Tools`_ (**VS Build Tools** is not needed if **Visual Studio** (2015 or newer) is already installed).

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -A x64 -DBUILD_CPP_TEST=ON -DUSE_OPENMP=OFF ..
     cmake --build . --target testlightgbm --config Debug

The ``.exe`` file will be in ``LightGBM/Debug`` folder.

Linux
^^^^^

On Linux a C++ unit tests of LightGBM can be built using **CMake** and **gcc** or **Clang**.

1. Install `CMake`_.

2. 다음 명령을 실행합니다:

   .. code::

     git clone --recursive https://github.com/microsoft/LightGBM
     cd LightGBM
     mkdir build
     cd build
     cmake -DBUILD_CPP_TEST=ON -DUSE_OPENMP=OFF ..
     make testlightgbm -j4

**Note**: glibc >= 2.14 is required.

macOS
^^^^^

On macOS a C++ unit tests of LightGBM can be built using **CMake** and **Apple Clang** or **gcc**.

Apple Clang
***********

Only **Apple Clang** version 8.1 or higher is supported.

1. Install `CMake`_ (3.16 or higher):

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

1. Install `CMake`_ (3.2 or higher):

   .. code::

     brew install cmake

2. Install **gcc**:

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
