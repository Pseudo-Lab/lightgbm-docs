심화 주제
===============

결측치 처리
--------------------

-  LightGBM은 결측치 처리가 기본설정 (default)입니다. 비활성화 하려면 ``use_missing=false`` 설정을 해줘야 합니다.

-  LightGBM은 결측치를 NA (NaN)로 표기하는 것이 기본설정입니다. 0으로 변경하려면 ``zero_as_missing=true`` 으로 설정해 줘야 합니다.

-  ``zero_as_missing=false`` 인 경우 (기본설정), sparse matrices (and LightSVM)의 결측값들은 0으로 처리됩니다. 

- ``zero_as_missing=true`` 인 경우에는, NA와 0들은 (sparse matrices (and LightSVM)의 결측치 포함) 모두 결측치로 처리됩니다.

범주형 특성변수 지원
---------------------------

-  LightGBM은 정수값의 범주형 특성변수에 대해 우수한 정확도를 보여줍니다. LightGBM은 `Fisher (1958) <https://www.tandfonline.com/doi/abs/10.1080/01621459.1958.10501479>`_ 을 적용하여 범주의 최적의 분할점을 찾습니다 (`described here <./Features.rst#optimal-split-for-categorical-features>`_ ). 이 방식은 원핫 인코딩  one-hot encoding 보다 대게 더 좋은 성능을 보입니다.

-  Use ``categorical_feature`` to specify the categorical features.
   Refer to the parameter ``categorical_feature`` in `Parameters <./Parameters.rst#categorical_feature>`__.

-  Categorical features must be encoded as non-negative integers (``int``) less than ``Int32.MaxValue`` (2147483647).
   It is best to use a contiguous range of integers started from zero.

-  Use ``min_data_per_group``, ``cat_smooth`` to deal with over-fitting (when ``#data`` is small or ``#category`` is large).

-  For a categorical feature with high cardinality (``#category`` is large), it often works best to
   treat the feature as numeric, either by simply ignoring the categorical interpretation of the integers or
   by embedding the categories in a low-dimensional numeric space.

람다랭크
----------

-  The label should be of type ``int``, such that larger numbers correspond to higher relevance (e.g. 0:bad, 1:fair, 2:good, 3:perfect).

-  Use ``label_gain`` to set the gain(weight) of ``int`` label.

-  Use ``lambdarank_truncation_level`` to truncate the max DCG.

Cost Efficient Gradient Boosting
--------------------------------

`Cost Efficient Gradient Boosting <https://papers.nips.cc/paper/6753-cost-efficient-gradient-boosting.pdf>`_ (CEGB)  makes it possible to penalise boosting based on the cost of obtaining feature values.
CEGB penalises learning in the following ways:

- Each time a tree is split, a penalty of ``cegb_penalty_split`` is applied.
- When a feature is used for the first time, ``cegb_penalty_feature_coupled`` is applied. This penalty can be different for each feature and should be specified as one ``double`` per feature.
- When a feature is used for the first time for a data row, ``cegb_penalty_feature_lazy`` is applied. Like ``cegb_penalty_feature_coupled``, this penalty is specified as one ``double`` per feature.

Each of the penalties above is scaled by ``cegb_tradeoff``.
Using this parameter, it is possible to change the overall strength of the CEGB penalties by changing only one parameter.

파라미터 튜닝
-----------------

-  Refer to `Parameters Tuning <./Parameters-Tuning.rst>`__.

.. _Parallel Learning:

Distributed Learning
--------------------

-  Refer to `Distributed Learning Guide <./Parallel-Learning-Guide.rst>`__.

GPU 지원
-----------

-  `GPU Tutorial <./GPU-Tutorial.rst>`__ 와 `GPU Targets <./GPU-Targets.rst>`__ 을 참고해 주세요. 

Recommendations for gcc Users (MinGW, \*nix)
--------------------------------------------

-  `gcc Tips <./gcc-Tips.rst>`__ 을 참고해 주세요.
