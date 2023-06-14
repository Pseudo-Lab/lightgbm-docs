.. role:: raw-html(raw)
    :format: html

LightGBM 관련 자주 묻는 질문 (FAQ)
############

.. contents:: LightGBM 관련 자주묻는 질문 (FAQ)
    :depth: 1
    :local:
    :backlinks: none

------

중요한 문제
===============

**중요한 문제** 는 *충돌*, *예측 오류*, *무의미한 출력*, 또는 그 밖에 즉각적인 주의가 필요한 것일 수 있습니다.

이러한 문제는 `Microsoft/LightGBM 리포지토리 <https://github.com/microsoft/LightGBM/issues>`__ 에 게시해 주시기 바랍니다.

또한, 관련 전문 분야에 따라 핵심 팀원을 골뱅이 기호(@)와 함께 멘션하여 메시지를 보낼 수도 있습니다. 각 분야와 팀원에 대해서는 아래를 참조해 주시기 바랍니다. 

-  `@guolinke <https://github.com/guolinke>`__ **Guolin Ke** (C++ 코드 / R 패키지 / Python 패키지)
-  `@chivee <https://github.com/chivee>`__ **Qiwei Ye** (C++ 코드 / Python 패키지)
-  `@shiyu1994 <https://github.com/shiyu1994>`__ **Yu Shi** (C++ 코드 / Python 패키지)
-  `@btrotta <https://github.com/btrotta>`__ **Belinda Trotta** (C++ 코드)
-  `@Laurae2 <https://github.com/Laurae2>`__ **Damien Soukhavong** (R 패키지)
-  `@jameslamb <https://github.com/jameslamb>`__ **James Lamb** (R 패키지 / Dask 패키지)
-  `@jmoralez <https://github.com/jmoralez>`__ **José Morales** (Dask 패키지)
-  `@wxchan <https://github.com/wxchan>`__ **Wenxuan Chen** (Python 패키지)
-  `@henry0312 <https://github.com/henry0312>`__ **Tsukasa Omoto** (Python 패키지)
-  `@StrikerRUS <https://github.com/StrikerRUS>`__ **Nikita Titov** (Python 패키지)
-  `@huanzhang12 <https://github.com/huanzhang12>`__ **Huan Zhang** (GPU 서포트)
-  `@tongwu-msft <https://github.com/tongwu-msft>`__ **Tong Wu** (C++ 코드 / Python 패키지)
-  `@hzy46 <https://github.com/hzy46>`__ **Zhiyuan He** (C++ 코드 / Python 패키지)

중요한 문제를 전송할 시에는 다음과 같은 정보를 최대한 많이 기술해 주시기 바랍니다.

-  CLI(명령줄 인터페이스), R 및/또는 Python에서 재현 가능한가요?

-  래퍼 클래스와 관련된 문제인가요? (R 또는 Python?)

-  컴파일러와 관련된 문제인가요? (gcc 또는 Clang 버전인가요? MinGW 또는 Visual Studio 버전인가요?)

-  운영체제와 관련된 문제인가요? (Windows? Linux? macOS?)

-  간단한 사례를 통해 이 문제를 재현할 수 있나요?

-  모든 최적화 플래그를 제거하고 디버그 모드에서 LightGBM을 컴파일한 후에도 같은 문제가 지속되나요?

중요한 문제에 대한 지원 작업은 대부분 자원봉사로 이루어지며 연중무휴 24시간 제공이 어려울 수도 있다는 점을 염두에 두시기 바랍니다.

--------------

LightGBM에 관한 일반적인 질문
==========================

.. contents::
    :local:
    :backlinks: none

1. LightGBM 파라미터에 대한 자세한 내용은 어디에서 확인할 수 있나요?
----------------------------------------------------------

`파라미터 <./Parameters.rst>`__ 와 `Laurae++/Parameters <https://sites.google.com/view/lauraepp/parameters>`__ 페이지를 참조하시기 바랍니다.

2. 수백만 개의 기능이 있는 데이터 세트의 경우 학습이 시작되지 않거나 매우 오랜 시간이 지난 후에 학습이 시작됩니다.
-----------------------------------------------------------------------------------------------------

``bin_construct_sample_cnt`` 에는 작은 값을, ``min_data`` 에는 큰 값을 사용합니다.

3. 대규모의 데이터 세트에서 LightGBM을 실행할 경우 컴퓨터의 메모리가 부족합니다.
-------------------------------------------------------------------------

**다음의 다양한 해결책을 시도해 보세요**: ``histogram_pool_size`` 파라미터를 LightGBM에 사용할 사이즈(MB)로 설정하거나(histogram\_pool\_size + 데이터셋 사이즈 = 대략적인 RAM 사용량), ``num_leaves`` 또는 ``max_bin`` 을 낮춥니다. (`Microsoft/LightGBM#562 <https://github.com/microsoft/LightGBM/issues/562>`__ 참조)

4. Windows를 사용하고 있습니다. LightGBM을 컴파일할 때 Visual Studio를 사용해야 하나요, 아니면 MinGW를 사용해야 하나요?
----------------------------------------------------------------------------------

Visual Studio가 `LightGBM에서 가장 잘 작동합니다 <https://github.com/microsoft/LightGBM/issues/542>`__.

5. LightGBM GPU를 사용할 때 여러 번 실행해도 결과를 재현할 수 없습니다.
-------------------------------------------------------------------------

이는 정상적이며 예상되는 동작이나, 재현성을 위해 ``gpu_use_dp = true`` 를 사용해 볼 수 있습니다.
(`Microsoft/LightGBM#560 <https://github.com/microsoft/LightGBM/pull/560#issuecomment-304561654>`__ 참조).
CPU 버전을 사용해 볼 수도 있습니다.

6. 스레드 수 변경 시 배깅을 재현할 수 없습니다.
-------------------------------------------------------------------

:raw-html:`<strike>`
LightGBM에서 배깅은 멀티스레드이므로 출력 결과는 사용되는 스레드 수에 따라 달라집니다.
`현재로선 해결 방법이 없습니다 <https://github.com/microsoft/LightGBM/issues/632>`__.
:raw-html:`</strike>`

`#2804 <https://github.com/microsoft/LightGBM/pull/2804>`__ 부터 시작하는 배깅 결과는 스레드 수에 의존하지 않습니다. 
따라서 이 문제는 최신 버전에서 해결되어야 합니다.

7. 랜덤 포레스트 모드를 사용하려고 했는데 LightGBM과 충돌합니다.
-----------------------------------------------------------

이는 임의의 파라미터에 대해 예상되는 동작입니다. 랜덤 포레스트를 활성화하려면, ``bagging_freq`` 와 함께 1이 아닌 ``bagging_fraction`` 및 ``feature_fraction`` 을 사용해야 합니다. 
다음 `스레드 <https://github.com/microsoft/LightGBM/issues/691>`__ 에서 예제를 확인할 수 있습니다.

8. LightGBM을 멀티코어 시스템에서 매우 큰 데이터 세트에 대해 사용할 때 Windows의 CPU 사용량이 10% 정도로 낮습니다.
------------------------------------------------------------------------------------------------------------

`Visual Studio <https://visualstudio.microsoft.com/downloads/>`__ 를 사용하세요. 특히 매우 큰 트리의 경우 `MinGW 보다 10배 더 빠를 수 있습니다 <https://github.com/microsoft/LightGBM/issues/749>`__.

9. ``categorical_feature`` 파라미터를 사용하여 범주형 열을 지정하려고 할 때 다음과 같은 일련의 경고가 표시되지만 열에 음수 값이 없습니다.
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

.. code-block:: console

   [LightGBM] [Warning] Met negative value in categorical features, will convert it to NaN
   [LightGBM] [Warning] There are no meaningful features, as all feature values are constant.

``categorical_feature`` 를 통해 전달하려는 열에 매우 큰 값이 포함되어 있을 가능성이 높습니다.
LightGBM의 범주형 기능은 int32 범위에 의해 제한됩니다.
따라서 ``Int32.MaxValue`` (2147483647)보다 큰 값은 범주형 기능으로 전달할 수 없습니다. (`Microsoft/LightGBM#1359 <https://github.com/microsoft/LightGBM/issues/1359>`__ 참조).
먼저 0에서 범주 수 사이의 정수로 변환해야 합니다.

10. 다음과 같은 오류와 함께 LightGBM이 무작위로 충돌합니다: ``Initializing libiomp5.dylib, but found libomp.dylib already initialized.``
-------------------------------------------------------------------------------------------------------------------------------

.. code-block:: console

   OMP: Error #15: Initializing libiomp5.dylib, but found libomp.dylib already initialized.
   OMP: Hint: This means that multiple copies of the OpenMP runtime have been linked into the program. That is dangerous, since it can degrade performance or cause incorrect results. The best thing to do is to ensure that only a single OpenMP runtime is linked into the process, e.g. by avoiding static linking of the OpenMP runtime in any library. As an unsafe, unsupported, undocumented workaround you can set the environment variable KMP_DUPLICATE_LIB_OK=TRUE to allow the program to continue to execute, but that may cause crashes or silently produce incorrect results. For more information, please see http://www.intel.com/software/products/support/.

**가능한 원인**: 이 오류는 컴퓨터에 여러 개의 OpenMP 라이브러리가 설치되어 있고 서로 충돌한다는 의미입니다.
(오류 메시지의 파일 확장자는 운영 체제에 따라 다를 수 있습니다)

Conda에서 배포한 Python을 사용하는 경우, 이 오류는 ``mkl`` 패키지가 포함된 Conda의 ``numpy`` 패키지가 시스템 전체 라이브러리와 충돌하여 발생했을 가능성이 높습니다. 
이 경우, Conda에서 ``numpy`` 패키지를 업데이트하거나, Conda 환경 폴더 ``$CONDA_PREFIX/lib`` 에 심볼릭 링크를 생성하여 Conda의 OpenMP 라이브러리 인스턴스를 시스템 전체 인스턴스로 교체합니다.

**해결책**: Homebrew와 함께 macOS를 사용하는 경우, 현재 활성 상태인 Conda 환경의 OpenMP 라이브러리 파일을 Homebrew가 설치한 시스템 전체 라이브러리 파일에 대한 심볼릭 링크로 덮어쓰는 명령은 다음과 같습니다.

.. code-block:: bash

   ln -sf `ls -d "$(brew --cellar libomp)"/*/lib`/* $CONDA_PREFIX/lib

위의 내용은 OpenMP 8.0.0 버전 출시 이전에는 정상적으로 작동했습니다. 
8.0.0 버전부터는, OpenMP용 Homebrew 수식에 ``-DLIBOMP_INSTALL_ALIASES=OFF`` 옵션이 포함되어 위 내용을 통한 수정은 더 이상 작동하지 않습니다. 
단, 다음을 통해 라이브러리 별칭에 대한 심볼릭 링크를 수동으로 생성할 수 있습니다.

.. code-block:: bash

   for LIBOMP_ALIAS in libgomp.dylib libiomp5.dylib libomp.dylib; do sudo ln -sf "$(brew --cellar libomp)"/*/lib/libomp.dylib $CONDA_PREFIX/lib/$LIBOMP_ALIAS; done

또 다른 해결 방법은 Conda의 패키지에서 MKL 최적화를 완전히 제거하는 것입니다.

.. code-block:: bash

    conda install nomkl

이 경우에 해당하지 않는다면, 충돌하는 OpenMP 라이브러리 설치를 직접 찾아서 그 중 하나만 남겨 두어야 합니다.

11. Linux에서 멀티스레딩(OpenMP)과 포킹을 동시에 사용하면 LightGBM이 중단됩니다.
--------------------------------------------------------------------------------------------

LightGBM의 멀티스레딩을 비활성화하려면 ``nthreads=1`` 을 사용하세요. 멀티스레딩이 활성화된 상태에서 포크된 세션을 중단시키는 버그가 OpenMP에 존재합니다. 좀 더 수고스러운 해결책은 포크 대신 새 프로세스를 사용하는 것입니다. 단, 새 프로세스 생성 시 메모리를 복사하고 라이브러리를 로딩하는 작업이 필요하다는 점을 염두에 두세요. (예: 현재의 프로세스를 16회 포크할 경우, 메모리상에 데이터 세트의 복사본을 16개 생성해야 합니다)
(`Microsoft/LightGBM#1789 <https://github.com/microsoft/LightGBM/issues/1789#issuecomment-433713383>`__ 참조).

포크된 세션 내에서 멀티스레딩이 필요한 경우, LightGBM을 Intel 툴체인으로 컴파일하는 방법이 있습니다. Intel 컴파일러는 이 문제의 영향을 받지 않습니다. 

C/C++ 사용자의 경우, 포크가 발생하기 전에는 OpenMP 기능을 사용할 수 없습니다. 포크가 발생하기 전에 OpenMP 기능을 사용하면(예: 포크에 OpenMP 사용), OpenMP는 포크된 세션 내부에서 중단됩니다. 이 경우, 대신 새 프로세스를 사용하고, 포크 대신 새 프로세스를 생성하여 필요에 따라 메모리를 복사합니다(또는 Intel 컴파일러 사용). 

클라우드 플랫폼 컨테이너 서비스가 Linux 포크를 사용하여 단일 인스턴스상에서 복수의 컨테이너를 실행하는 경우, LightGBM이 중단될 수 있습니다. 예를 들면, LightGBM은 `ECS 에이전트 
<https://aws.amazon.com/batch/faqs/#Features>`__ 를 사용하여 실행 중인 여러 작업을 관리하는 AWS의 배치 배열 작업에서 중단됩니다. ``nthreads=1`` 을 설정하면 문제가 개선됩니다.

12. LightGBM에서 조기 중단이 기본적으로 활성화되지 않는 이유는 무엇인가요?
-------------------------------------------------------------

조기 중단은 각각의 반복 후 모델의 현재 상태를 평가하여 학습을 중단할 수 있는지 확인하는 데 사용되는 특수한 유형의 홀드아웃인 유효성 검사 집합의 선택을 포함합니다.

``LightGBM`` 에서는 `사용자가 이 세트를 직접 지정하도록 했습니다 <./Parameters.rst#valid>`_. 훈련 데이터를 훈련, 테스트 및 검증 세트로 분할하는 데는 여러 가지 옵션이 있습니다.

적절한 분할 전략은 데이터의 작업과 도메인, 그리고 모델러에는 있으나 범용 도구인 ``LightGBM`` 에는 없는 정보에 따라 달라집니다.

13. LightGBM은 제로 기반 또는 1 기반 LibSVM 형식 파일에서 데이터 직접 로딩을 지원하나요?
----------------------------------------------------------------------------------------------

LightGBM은 제로 기반 LibSVM 형식 파일에서 데이터를 직접 로드하는 기능을 지원합니다.

14. CMake가 MinGW로 LightGBM을 컴파일할 때 컴파일러를 찾을 수 없는 이유는 무엇인가요?
--------------------------------------------------------------------------

.. code-block:: bash

    CMake Error: CMAKE_C_COMPILER not set, after EnableLanguage
    CMake Error: CMAKE_CXX_COMPILER not set, after EnableLanguage

이것은 CMake의 알려진 문제로 MinGW 사용 시 발생합니다. 가장 쉬운 해결책은 ``cmake`` 명령을 다시 실행하여 CMake의 일회성 스토퍼를 우회하는 것입니다. 또는 CMake 버전을 최소 버전 3.17.0으로 업그레이드하는 방법이 있습니다.

자세한 내용은 '`Microsoft/LightGBM#3060 <https://github.com/microsoft/LightGBM/issues/3060#issuecomment-626338538>`__' 을 참조하세요.

15. 프레젠테이션에 사용할 LightGBM의 로고는 어디에서 찾을 수 있나요?
------------------------------------------------------------------

다음 `링크 <https://github.com/microsoft/LightGBM/tree/master/docs/logo>`__ 를 클릭하면 다양한 파일 형식과 해상도의 LightGBM 로고를 찾을 수 있습니다.

------

R 패키지
=========

.. contents::
    :local:
    :backlinks: none

1. 이전 LightGBM 모델을 학습하는 동안 오류가 발생한 후부터 LightGBM을 사용하는 모든 학습 명령이 작동하지 않습니다.
------------------------------------------------------------------------------------------------------------------------------

R 콘솔에서 ``lgb.unloader(wipe = TRUE)`` 를 실행하고 LightGBM 데이터 세트를 다시 생성합니다(이렇게 하면 모든 LightGBM 관련 변수가 지워집니다). 
포인터로 인해 변수를 지우지 않도록 선택해도 오류는 해결되지 않습니다.
이는 이미 알려진 문제입니다. `Microsoft/LightGBM#698 <https://github.com/microsoft/LightGBM/issues/698>`__ 을 참조하세요.

2. ``setinfo()`` 를 사용하여 ``lgb.Dataset`` 를 표시하려 했는데 R 콘솔이 반응하지 않습니다.
----------------------------------------------------------------------------------------

``setinfo`` 를 사용한 후 ``lgb.Dataset`` 을 표시하지 마세요. 
이는 이미 알려진 문제입니다. `Microsoft/LightGBM#539 <https://github.com/microsoft/LightGBM/issues/539>`__ 를 참조하세요. 

3. ``error in data.table::data.table()...argument 2 is NULL``
-------------------------------------------------------------

``lightgbm`` 실행 시 이 오류가 발생하는 경우, `#2715 <https://github.com/microsoft/LightGBM/issues/2715>`_ 및 `#2989 <https://github.com/microsoft/LightGBM/pull/2989#issuecomment-614374151>`_ 이후에서 보고된 것과 동일한 문제에 직면할 가능성이 있습니다. 일부 상황에서 ``data.table`` 1.11.x를 사용하면 이 문제가 발생하는 것으로 확인되었습니다. 이 문제를 해결하려면 ``data.table`` 을 버전 1.12.0이상으로 업그레이드하세요.

------

Python 패키지
==============

.. contents::
    :local:
    :backlinks: none

1. ``python setup.py install`` 을 사용하여 GitHub에서 설치 시 ``Error: setup script specifies an absolute path`` 라는 에러 메시지가 표시됩니다.
--------------------------------------------------------------------------------------------------------------------

.. code-block:: console

   error: Error: setup script specifies an absolute path:
   /Users/Microsoft/LightGBM/python-package/lightgbm/../../lib_lightgbm.so
   setup() arguments must *always* be /-separated paths relative to the setup.py directory, *never* absolute paths.

이 문제는 최신 버전에서 해결되어야 합니다. 
그럼에도 불구하고 이 문제가 계속해서 발생할 경우, Python 패키지에서 ``lightgbm.egg-info`` 폴더를 제거한 후 다시 설치해 보세요. 
또는 `Stack Overflow의 다음 스레드를 참조하세요 <http://stackoverflow.com/questions/18085571/pip-install-error-setup-script-specifies-an-absolute-path>`__.

2. ``Cannot ... before construct dataset`` 과 같은 에러 메시지가 표시됩니다.
-----------------------------------------------------------

다음과 같은 에러 메시지가 표시됩니다.

.. code-block:: console

   Cannot get/set label/weight/init_score/group/num_data/num_feature before construct dataset

하지만 이미 다음과 같은 코드로 데이터 세트를 이미 구성했습니다. 

.. code-block:: python

    train = lightgbm.Dataset(X_train, y_train)

또는 다음과 같은 에러 메시지가 표시됩니다.

.. code-block:: console

    Cannot set predictor/reference/categorical feature after freed raw data, set free_raw_data=False when construct Dataset to avoid this.

**해결책**: LightGBM은 bin 매퍼를 구성하여 트리를 구축하고, 같은 bin 매퍼, 범주형 기능, 기능 이름 등을 공유하는 하나의 부스터 내에서 데이터 세트를 훈련하고 검증합니다. 따라서 부스터 구성 시 데이터 세트의 객체도 함께 구성됩니다.
``free_raw_data=True`` (기본값)을 설정하면, 미가공 데이터(Python 데이터 구조 포함)가 해제됩니다.
다음을 참조하여 각 상황에 맞는 대응을 실시하세요.

-  데이터 세트를 구성하기 전에 label(또는 weight/init\_score/group/data)을 가져올 경우, 방법은 ``self.label`` 을 가져오는 방법과 동일합니다.

-  데이터 세트를 구성하기 전에 label(또는 weight/init\_score/group)을 설정할 경우, 방법은 ``self.label=some_label_array`` 와 동일합니다.

-  데이터 세트를 구성하기 전에 num\_data(또는 num\_feature)을 가져올 경우, ``self.data`` 를 통해 데이터를 얻을 수 있습니다.
   그리고 만약 데이터가 ``numpy.ndarray`` 인 경우, ``self.data.shape`` 와 같은 코드를 사용합니다. 단, 데이터 세트를 서브 세트로 설정한 후 이를 실행하게 되면 계속해서 ``None`` 을 결과 값으로 얻게 되니 주의하세요.

-  데이터 세트를 구성한 후에 predictor(또는 reference/categorical feature)를 설정할 경우, ``free_raw_data=False`` 를 설정하거나 동일한 미가공 데이터로 데이터 세트 객체를 초기화해야 합니다. 

3. PyPI에서 ``pip install lightgbm`` 을 사용하여 LightGBM을 설치한 후 세그멘테이션 오류(세그폴트)가 무작위로 발생합니다.
---------------------------------------------------------------------------------------------------------------------------

저희는 빠른 실행 속도와 동시에 모든 하드웨어, OS, 컴파일러 등과 호환되는 범용 휠을 제공하기 위해 최선을 다하고 있습니다.
그러나 특정 환경에서의 사용 가능성을 보장할 수 없는 경우도 있습니다(`Microsoft/LightGBM#1743 <https://github.com/microsoft/LightGBM/issues/1743>`__ 참조).

따라서, 세그폴트 발생 시 가장 먼저 시도해야 할 것은 ``pip install --no-binary :all: lightgbm`` 을 사용하여 **소스로부터 컴파일** 하는 것입니다.
OS별 전제 조건에 대해서는 다음 `가이드 <https://github.com/microsoft/LightGBM/blob/master/python-package/README.rst#user-content-build-from-sources>`__ 를 참조하세요.

또한 새로운 문제가 있다면 GitHub 리포지토리에 자유롭게 게시해 주세요. 저희는 항상 각각의 사례를 개별적으로 살펴보고 근본적인 원인을 찾으려고 노력합니다.
