LightGBM GPU 튜토리얼
=====================

이 문서의 목적은 GPU 훈련에 대한 간단한 단계별 튜토리얼을 제공하는 것입니다.

Windows의 경우, `GPU Windows Tutorial <./GPU-Windows.rst>`__ 을 참조하세요.

데모에는 `Microsoft Azure 클라우드 컴퓨팅 플랫폼`_ 의 GPU 인스턴스를 사용합니다. 단, 최신 AMD 또는 NVIDIA GPU가 탑재된 모든 머신을 사용할 수 있습니다. 

GPU 설정하기
---------

Azure(미국 동부, 미국 중북부, 미국 중남부, 서유럽 및 동남아시아 지역에서 사용 가능)에서 ``NV`` 유형 인스턴스를 시작하고, 운영 체제로 Ubuntu 16.04 LTS를 선택합니다. 

테스트용으로는 8GB 메모리, 180GB/s 메모리 대역폭, 4,825 GFLOPS 피크 연산 성능을 갖춘 1/2 M60 GPU가 포함된 가장 작은 ``NV6`` 유형 가상 머신이면 충분합니다. 
GPU(K80)의 경우, 이전 아키텍처(Kepler)를 기반으로 하므로 ``NC`` 유형 인스턴스는 사용하지 마시기 바랍니다. 

먼저 최소한의 NVIDIA 드라이버와 OpenCL 개발 환경을 설치해야 합니다.

::

    sudo apt-get update
    sudo apt-get install --no-install-recommends nvidia-375
    sudo apt-get install --no-install-recommends nvidia-opencl-icd-375 nvidia-opencl-dev opencl-headers

드라이버를 설치한 후 서버를 다시 시작해야 합니다.

::

    sudo init 6

약 30초 후 서버가 다시 가동됩니다.

AMD GPU를 사용하는 경우, `AMDGPU-Pro`_ 드라이버를 다운로드하여 설치하고 ``ocl-icd-libopencl1`` 및 ``ocl-icd-opencl-dev`` 패키지도 설치해야 합니다.

LightGBM 구축하기
--------------

이제 구축에 필요한 도구와 종속 요소를 설치합니다.

::

    sudo apt-get install --no-install-recommends git cmake build-essential libboost-dev libboost-system-dev libboost-filesystem-dev

``NV6`` GPU 인스턴스의 경우, ``/mnt`` 에 320GB 초고속 SSD가 장착되어 있습니다. 
이를 작업 공간으로 설정하겠습니다. (개인 PC를 사용하는 경우 이 부분은 건너뛰세요)

::

    sudo mkdir -p /mnt/workspace
    sudo chown $(whoami):$(whoami) /mnt/workspace
    cd /mnt/workspace

이제 LightGBM을 체크아웃하고 GPU 지원으로 컴파일할 준비가 되었습니다.

::

    git clone --recursive https://github.com/microsoft/LightGBM
    cd LightGBM
    mkdir build
    cd build
    cmake -DUSE_GPU=1 .. 
    # if you have installed NVIDIA CUDA to a customized location, you should specify paths to OpenCL headers and library like the following:
    # cmake -DUSE_GPU=1 -DOpenCL_LIBRARY=/usr/local/cuda/lib64/libOpenCL.so -DOpenCL_INCLUDE_DIR=/usr/local/cuda/include/ ..
    make -j$(nproc)
    cd ..

두 개의 바이너리(``lightgbm`` 과 ``lib_lightgbm.so``)가 생성된 것을 확인할 수 있습니다.

macOS에서 구축하는 경우, Boost.Compute의 알려진 버그를 피하려면 ``src/treelearner/gpu_tree_learner.h``에서 ``BOOST_COMPUTE_USE_OFFLINE_CACHE`` 매크로를 제거해야 할 수도 있습니다.

Python 인터페이스 설치하기 (선택 사항)
-----------------------------------

LightGBM의 Python 인터페이스를 사용하려면, 필요한 Python 패키지 종속 요소 몇 가지와 함께 설치하면 됩니다.

::

    sudo apt-get -y install python-pip
    sudo -H pip install setuptools numpy scipy scikit-learn -U
    cd python-package/
    sudo python setup.py install --precompile
    cd ..

Python에서 GPU를 사용하려면 ``learning_rate``, ``num_leaves`` 등의 다른 옵션과 함께 추가 매개변수인 ``"device" : "gpu"`` 를 설정해야 합니다. 

Python 인터페이스의 사용 방법에 대한 자세한 내용은 `Python 패키지 예제`_ 를 참조하세요.

데이터셋 준비하기
-------------------

다음 명령을 사용하여 힉스(Higgs) 데이터셋을 준비합니다.

::

    git clone https://github.com/guolinke/boosting_tree_benchmarks.git
    cd boosting_tree_benchmarks/data
    wget "https://archive.ics.uci.edu/ml/machine-learning-databases/00280/HIGGS.csv.gz"
    gunzip HIGGS.csv.gz
    python higgs2libsvm.py
    cd ../..
    ln -s boosting_tree_benchmarks/data/higgs.train
    ln -s boosting_tree_benchmarks/data/higgs.test

이제 다음 명령을 실행하여 LightGBM용 구성 파일을 생성합니다. 블록을 모두 복사하여 전체로 실행하세요.

::

    cat > lightgbm_gpu.conf <<EOF
    max_bin = 63
    num_leaves = 255
    num_iterations = 50
    learning_rate = 0.1
    tree_learner = serial
    task = train
    is_training_metric = false
    min_data_in_leaf = 1
    min_sum_hessian_in_leaf = 100
    ndcg_eval_at = 1,3,5,10
    device = gpu
    gpu_platform_id = 0
    gpu_device_id = 0
    EOF
    echo "num_threads=$(nproc)" >> lightgbm_gpu.conf

방금 생성한 구성 파일에서 ``device=gpu`` 를 설정하여 GPU를 활성화합니다. 
이 구성에서는 시스템에 설치된 첫 번째 GPU(``gpu_platform_id=0`` 및 ``gpu_device_id=0``)를 사용합니다. ``gpu_platform_id`` 또는 ``gpu_device_id`` 가 설정되지 않은 경우, 기본 플랫폼과 GPU가 선택됩니다. 
여러 플랫폼(AMD/Intel/NVIDIA) 또는 GPU를 사용할 수 있습니다. `clinfo`_ 유틸리티를 사용하여 각 플랫폼의 GPU를. 식별할 수 있습니다. Ubuntu의 경우, ``sudo apt-get install clinfo`` 를 실행하여 ``clinfo`` 를 설치할 수 있습니다. AMD/NVIDIA의 외장형 GPU와 Intel의 통합형 GPU를 사용하는 경우, 외장형 GPU를 사용하려면 올바른 ``gpu_platform_id`` 를 선택해야 합니다.

GPU에서 첫 학습 작업 실행하기
-----------------------------------

이제 GPU 훈련을 시작할 준비가 완료되었습니다.

먼저 GPU가 올바르게 작동하는지 확인합니다.
다음 명령을 실행하여 GPU에서 훈련을 실행하고 50회 반복 후 AUC를 기록합니다.

::

    ./lightgbm config=lightgbm_gpu.conf data=higgs.train valid=higgs.test objective=binary metric=auc

이제 다음 명령을 사용하여 CPU에서 동일한 데이터셋을 훈련합니다. 비슷한 AUC를 관찰할 수 있을 것입니다.

::

    ./lightgbm config=lightgbm_gpu.conf data=higgs.train valid=higgs.test objective=binary metric=auc device=cpu

이제 각 반복마다 AUC를 계산하지 않고도 GPU에서 속도 테스트를 수행할 수 있습니다.

::

    ./lightgbm config=lightgbm_gpu.conf data=higgs.train objective=binary metric=auc

CPU상에서 속도 테스트는 다음과 같이 진행합니다.

::

    ./lightgbm config=lightgbm_gpu.conf data=higgs.train objective=binary metric=auc device=cpu

이 GPU에서 3배 이상의 속도 향상을 관찰할 수 있습니다.

GPU 가속은 다른 작업 및 지표(회귀, 다중 클래스 분류, 랭킹 등)에도 사용할 수 있습니다.
예를 들면, 회귀 작업으로 GPU에서 힉스 데이터셋을 훈련할 수 있습니다.

::

    ./lightgbm config=lightgbm_gpu.conf data=higgs.train objective=regression_l2 metric=l2

또한 훈련 속도를 CPU와 비교할 수 있습니다.

::

    ./lightgbm config=lightgbm_gpu.conf data=higgs.train objective=regression_l2 metric=l2 device=cpu

추가 정보
---------------

- `GPU 튜닝 가이드 및 성능 비교 <./GPU-Performance.rst>`__

- `GPU SDK 대응 및 디바이스 타겟팅 테이블 <./GPU-Targets.rst>`__

- `GPU Windows 튜토리얼 <./GPU-Windows.rst>`__

참고자료
---------

GPU 가속이 유용하다고 생각될 경우, 다음 문헌을 출판물에 인용해 주시기 바랍니다. 

Huan Zhang, Si Si and Cho-Jui Hsieh. "`대규모 트리 부스팅을 위한 GPU 가속화`_." SysML Conference, 2018.

.. _Microsoft Azure 클라우드 컴퓨팅 플랫폼: https://azure.microsoft.com/

.. _AMDGPU-Pro: https://www.amd.com/en/support

.. _Python 패키지 예제: https://github.com/microsoft/LightGBM/tree/master/examples/python-guide

.. _대규모 트리 부스팅을 위한 GPU 가속화: https://arxiv.org/abs/1706.08359

.. _clinfo: https://github.com/Oblomov/clinfo
