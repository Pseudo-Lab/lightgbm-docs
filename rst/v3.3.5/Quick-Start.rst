퀵 스타트
===========

이 문서는 LightGBM CLI 버전을 위한 퀵 스타트 가이드입니다.

먼저 `Installation Guide <./Installation-Guide.rst>`__ 를 따라 LightGBM을 설치해 주세요.

**도움이 될만한 다른 링크**

-  `파라미터 <./Parameters.rst>`__

-  `파라미터 튜닝 <./Parameters-Tuning.rst>`__

-  `파이썬 패키지 퀵 스타트 <./Python-Intro.rst>`__

-  `파이썬 API <./Python-API.rst>`__

훈련 데이터 포맷
----------------------------

LightGBM은 `CSV`_, `TSV`_ 및 `LibSVM`_ (0 기반) 포맷의 입력 데이터 파일을 지원합니다.

파일은 `헤더 <./Parameters.rst#header>`__ 를 포함하거나 포함하지 않을 수 있습니다.

`레이블 열 <./Parameters.rst#label_column>`__ 은 인덱스나 이름으로 지정할 수 있습니다.

일부 열은 `무시할 <./Parameters.rst#ignore_column>`__ 수 있습니다.

범주형 변수 지원
~~~~~~~~~~~~~~~~~~~~~~~~~~~

LightGBM은 (원핫 인코딩) 없이 범주형 변수를 직접 사용할 수 있습니다.
`Expo data`_ 를 대상으로 실험한 결과, 원핫 인코딩 대비 약 8배의 속도 향상을 보였습니다.

자세한 설정은 ``categorical_feature`` `파라미터 <./Parameters.rst#categorical_feature>`__ 을 참고하세요.

가중치 및 쿼리/그룹 데이터
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LightGBM은 가중치 훈련도 지원하므로 추가적인 `weight data <./Parameters.rst#weight-data>`__ 가 필요합니다.
그리고 랭킹 작업을 위해 `쿼리 데이터가 <./Parameters.rst#query-data>`_ 추가로 필요합니다.

또한, `가중치 <./Parameters.rst#weight_column>`__ 및 `쿼리 <./Parameters.rst#group_column>`__ 데이터는 레이블과 같은 방식으로 훈련 데이터에 있는 열로 지정할 수 있습니다.

파라미터 둘러보기
---------------------------

파라미터 포맷은 ``key1=value1 key2=value2 ...`` 입니다.

파라미터는 설정 파일과 명령줄 모두에서 설정할 수 있습니다.
하나의 파라미터가 명령줄과 설정 파일에 모두 나타나면, LightGBM은 명령줄의 파라미터를 사용합니다.

새로운 사용자가 살펴봐야 할 가장 중요한 파라미터는 `핵심 파라미터 <./Parameters.rst#core-parameters>`__ 와
`학습 제어 파라미터 <./Parameters.rst#learning-control-parameters>`__ 맨 위에 있습니다.
`LightGBM 파라미터 <./Parameters.rst>`__ 문서에서 전체 목록을 확인할 수 있습니다.

LightGBM 실행하기
------------------------

::

    "./lightgbm" config=your_config_file other_args ...

파라미터는 설정 파일과 명령줄 모두에서 설정할 수 있으며, 명령줄의 파라미터는 설정 파일보다 우선순위가 높습니다.
예를 들어, 다음 명령줄은 ``num_trees=10``을 사용하고 설정 파일에 있는 동일한 파라미터를 무시합니다.

::

    "./lightgbm" config=train.conf num_trees=10

예제
------------

-  `Binary Classification <https://github.com/microsoft/LightGBM/tree/master/examples/binary_classification>`__

-  `Regression <https://github.com/microsoft/LightGBM/tree/master/examples/regression>`__

-  `Lambdarank <https://github.com/microsoft/LightGBM/tree/master/examples/lambdarank>`__

-  `Distributed Learning <https://github.com/microsoft/LightGBM/tree/master/examples/parallel_learning>`__

.. _CSV: https://en.wikipedia.org/wiki/Comma-separated_values

.. _TSV: https://en.wikipedia.org/wiki/Tab-separated_values

.. _LibSVM: https://www.csie.ntu.edu.tw/~cjlin/libsvm/

.. _Expo data: http://stat-computing.org/dataexpo/2009/
