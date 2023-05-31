심화 주제
===============

결측치 처리
--------------------

-  LightGBM은 결측치 처리를 기본설정(default)으로 합니다. 비활성화 하려면 ``use_missing=false`` 설정을 해줘야 합니다.

-  LightGBM은 결측치를 NA (NaN)로 표기하는 것이 기본설정입니다. 0으로 변경하려면 ``zero_as_missing=true`` 으로 설정해야 합니다.

-  ``zero_as_missing=false`` 인 경우 (기본설정), sparse matrices (and LightSVM)의 결측값들은 0으로 처리됩니다. 

- ``zero_as_missing=true`` 인 경우에는, NA와 0들은 (sparse matrices (and LightSVM)의 결측치 포함) 모두 결측치로 처리됩니다.

범주형 특성변수 지원
---------------------------

-  LightGBM은 정수값의 범주형 특성변수에 대해 우수한 정확도를 보여줍니다. LightGBM은 `Fisher (1958) <https://www.tandfonline.com/doi/abs/10.1080/01621459.1958.10501479>`_ 을 적용하여 범주의 최적의 분할점을 찾습니다 (`자세한 설명 <./Features.rst#optimal-split-for-categorical-features>`_). 이 방식은 원-핫 인코딩 (one-hot encoding) 보다 대게 더 좋은 성능을 보입니다.

-  LightGBM은 범주형 특성변수를 특정하기 위해 ``categorical_feature`` 을 사용합니다.
   `Parameters <./Parameters.rst#categorical_feature>`__ 의 파라미터 ``categorical_feature``을 참고해 주세요. 

-  범주형 변수는 반드시 ``Int32.MaxValue`` (2147483647) 보다는 작은 음수가 아닌 정수값을 가져야 합니다. 
   0부터 시작하는 서로 인접한 범위 내의 정수값을 사용하는 것이 가장 좋습니다.

-  (데이터 사이즈 ``#data`` 가 작거나 범주의 개수 ``#category`` 가 너무 클 경우) ``min_data_per_group``, ``cat_smooth`` 을 사용하여 과적합 문제를 처리할 수 있습니다.

-  카디널리티가 높은 범주형 변수의 경우 (범주의 개수 ``#category``가 큰 경우), 해당 특성변수를 수치형(numeric)으로 다루는 것이 대게 가장 좋은 결과를 가져옵니다. 이때, 수치형으로 다루는 방법으로는 정수값들에 대해 범주형 변수로 해석하지 않고 간단하게 무시하는 것 혹은 범주들을 저차원 (low-dimension)의 수치 공간 (numeric space)으로 치환(embedding)하는 것 등이 있습니다.

람다랭크 (LambdaRank)
-------------------

-  라벨 (label)의 타입은 반드시 정수 ``int`` 를 취해야 하며, 더 큰 숫자가 더 높은 연관성(중요도)을 나타내야 합니다 (예. 0:나쁨, 1:보통, 2:좋음, 3:아주 좋음).

-  ``label_gain`` 을 사용하여 "int" 레이블의 정보획득(gain; 가중치)를 설정합니다.

-  ``lambdarank_truncation_level`` 을 사용하여 최대 DCG(Discriminative Cumulative Gain)을 잘라냅니다.

Cost Efficient Gradient Boosting
--------------------------------

`Cost Efficient Gradient Boosting <https://papers.nips.cc/paper/6753-cost-efficient-gradient-boosting.pdf>`_ (CEGB)는 특성변수 값을 얻는 데 드는 비용에 기반하여 부스팅에 패널티를 줍니다. makes it possible to penalise boosting based on the cost of obtaining feature values.
CEGB는 아래와 같은 방식으로 학습에 패널티를 부여합니다 :

- 나무가 분기할 때 마다, ``cegb_penalty_split`` 에 의해 패널티가 부과됩니다.
- 특정 특성변수가 처음 사용됐을 경우, ``cegb_penalty_feature_coupled`` 가 적용됩니다. 이 패널티는 특성변수 마다 다르며, 변수마다 반드시 단일의 ``double`` 로 지정/설정되어야 합니다. 
  his penalty can be different for each feature and should be specified as one ``double`` per feature.
- When a feature is used for the first time for a data row, ``cegb_penalty_feature_lazy`` is applied. Like ``cegb_penalty_feature_coupled``, this penalty is specified as one ``double`` per feature.

Each of the penalties above is scaled by ``cegb_tradeoff``.
Using this parameter, it is possible to change the overall strength of the CEGB penalties by changing only one parameter.

파라미터 튜닝
-----------------

-  `Parameters Tuning <./Parameters-Tuning.rst>`__ 을 참고해 주세요. 

.. _Parallel Learning:

분산학습 (Distributed Learning)
-----------------------------

-  `Distributed Learning Guide <./Parallel-Learning-Guide.rst>`__ 을 참고해 주세요. 

GPU 지원
-----------

-  `GPU Tutorial <./GPU-Tutorial.rst>`__ 와 `GPU Targets <./GPU-Targets.rst>`__ 을 참고해 주세요. 

Recommendations for gcc Users (MinGW, \*nix)
--------------------------------------------

-  `gcc Tips <./gcc-Tips.rst>`__ 을 참고해 주세요.
