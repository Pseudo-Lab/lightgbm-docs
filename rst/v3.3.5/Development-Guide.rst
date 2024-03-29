개발 가이드
=================

알고리즘
----------

LightGBM에서 사용하는 주요 알고리즘에 대한 설명은 `Features <./Features.rst>`__ 을 참고해 주세요.

클래스와 코드 구조
--------------------------

주요 클래스들
~~~~~~~~~~~~~~~~~

+-----------------------+------------------------------------------------------------------------------------------------------------------+
| 클래스                | 설명                                                                                                              |
+=======================+==================================================================================================================+
| ``Application``       | 학습과 예측 로직을 포함한 어플리케이션의 시작 부분입니다.                                                             |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``Bin``               | 불연속적인 피쳐 값을 저장할 때 사용하는 데이터 구조가 정의되어 있습니다. (실수 값에서 변환됩니다.)                       |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``Boosting``          | 부스팅 방식 (GBDT, DART, GOSS 등)이 정의되어 있습니다.                                                              |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``Config``            | 파라미터와 설정요소들을 저장합니다.                                                                                 |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``Dataset``           | 데이터셋의 정보를 저장합니다.                                                                                      |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``DatasetLoader``     | 데이터셋을 구성할 때 사용합니다.                                                                                   |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``FeatureGroup``      | 피쳐 데이터를 저장하며, 피쳐가 여러 개로 구성될 수도 있습니다.                                                       |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``Metric``            | 평가 지표가 정의되어 있습니다.                                                                                     |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``Network``           | 네트워크 인터페이스와 통신 알고리즘이 정의되어 있습니다.                                                             |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``ObjectiveFunction`` | 학습시 사용하는 목적함수가 정의되어 있습니다.                                                                       |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``Tree``              | 트리 모델의 정보를 저장합니다.                                                                                     |
+-----------------------+------------------------------------------------------------------------------------------------------------------+
| ``TreeLearner``       | 트리를 학습할 때 사용합니다.                                                                                       |
+-----------------------+------------------------------------------------------------------------------------------------------------------+

코드 구조
~~~~~~~~~~~~~~

+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| 파일 경로          | 설명                                                                                                                           |
+===================+================================================================================================================================+
| ./include         | 헤더 파일이 있습니다.                                                                                                            |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| ./include/utils   | 일부 주요 함수들이 정의되어 있습니다.                                                                                             |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| ./src/application | 학습과 예측 로직이 정의되어 있습니다.                                                                                             |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| ./src/boosting    | 부스팅 모델이 정의되어 있습니다.                                                                                                 |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| ./src/io          | 입출력 관련 클래스가 정의되어 있으며, ``Bin``, ``Config``, ``Dataset``, ``DatasetLoader``, ``Feature``, ``Tree`` 등이 있습니다.    |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| ./src/metric      | 평가 지표가 정의되어 있습니다.                                                                                                   |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| ./src/network     | 네트워크 함수가 정의되어 있습니다.                                                                                               |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| ./src/objective   | 목적함수가 정의되어 있습니다.                                                                                                    |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+
| ./src/treelearner | 트리 학습기가 정의되어 있습니다.                                                                                                 |
+-------------------+--------------------------------------------------------------------------------------------------------------------------------+

문서 API
-------------

`docs README <./README.rst>`__ 를 참고해 주세요.

C API
-----

`C API <./C-API.rst>`__ 이나 `c\_api.h <https://github.com/microsoft/LightGBM/blob/master/include/LightGBM/c_api.h>`__ 파일 내 언급내용을 확인해주세요.

테스트
-----

C++ 단위 테스트는 ``./tests/cpp_tests`` 폴더 내에 있으며, 구글 테스트 프레임워크 (Google Test Framework)의 도움을 받아 작성했습니다.
로컬로 테스트를 실행하기 위해선 먼저 `Installation Guide <./Installation-Guide.rst#build-c-unit-tests>`__ 를 참조하여 테스트를 빌드하는 방법을 숙지하고, 그런 뒤 간단히 컴파일된 실행 파일을 실행시키면 됩니다.
`sanitizers <./Installation-Guide.rst#sanitizers>`__ 를 이용해서 테스트를 빌드하는 것을 강력히 권장합니다.


고급 프로그래밍 언어 패키지
---------------------------

`Python-package <https://github.com/microsoft/LightGBM/tree/master/python-package>`__ 와 `R-package <https://github.com/microsoft/LightGBM/tree/master/R-package>`__ 내 구현 코드를 확인해 주세요.


질문사항
---------

`FAQ <./FAQ.rst>`__ 를 참고해 주세요.

만약 문제가 생길 경우 언제든지 `issues <https://github.com/microsoft/LightGBM/issues>`__ 를 열어 문의할 수 있습니다.
