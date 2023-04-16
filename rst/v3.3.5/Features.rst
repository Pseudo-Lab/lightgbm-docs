Features
========

본 글은 LightGBM이 어떻게 작동하는가에 대한 개요입니다 \ `[1] <#references>`__. 독자가 의사결정나무 부스팅 알고리즘 (decision tree boosting algorithms)에 익숙하다는 전제 하에 LightGBM이 다른 부스팅 패키지와 어떻게 다른가에 초점을 맞추었습니다. 알고리즘에 대한 상세한 설명은 인용글 혹은 소스코드를 참고 부탁합니다.


연산속도와 메모리 사용량 최적화 
--------------------------------------

많은 부스팅 패키지들은 의사결정나무 학습을 위해 사전 정렬 기반의 알고리즘을 사용합니다 \ `[2, 3] <#references>`__ (예. xgboost의 기본설정 알고리즘). 이 해법은 간단한 대신 최적화가 쉽지 않습니다.

LightGBM은 히스토그램 기반의 알고리즘을 사용함으로써\ `[4, 5, 6] <#references>`__, 연속형 특성변수 (특성)를 이산형으로 범주화 시킵니다. 이는 학습 속도를 올리며 메모리 사용량을 감소시킵니다. 히스토그램 기반의 알고리즘이 갖는 우위는 다음과 같습니다:

-  **각 분할(split)의 정보획득(gain) 연산비용 감소**

   -  사전 정렬 기반의 알고리즘은 ``0(#data)`` 의 시간 복잡도(time complexity)를 가집니다 (인풋 데이터 양과 연산속도의 관계가 선형적). 
   
   -  히스토그램을 계산하는 것도 ``O(#data)`` 의 시간 복잡도를 가지지만, 이는 오직 빠른 합산 연산에서만 일어납니다. 히스토그램이 만들어지면 히스토그램 기반 알고리즘은 ``O(#bins)`` 의 시간 복잡도를 가지며, 여기서 #bins는 #data보다 훨씬 작으므로 연산비용도 훨씬 작습니다.  

-  **히스토그램 감법(뺄셈)을 통한 추가적 속도향상**

   -  이진(binary) 나무에서 한 잎사귀(leaf)의 히스토그램을 계산하기 위해, 그 잎사귀의 부모노드와 이웃 잎사귀의 차를 이용합니다. 

   -  따라서 이 트리는 오직 하나의 잎사귀에 대해서만 히스토그램을 만들면 되며, 더 작은 데이터(``#data``)를 가진 이웃 잎사귀가 히스토그램을 만들게 됩니다. 그 다음, 나머지 잎사귀의 히스토그램은 히스토그램 감법을 통해 작은 연산비용 (``O(#bins)``)으로 얻을 수 있게 됩니다.   
   
-  **메모리 사용량 감소**

   -  연속형 변수를 이산형으로 바꿉니다. 만약 구간 개수 (``#bins``)가 적다면 작은 데이터 타입 (예. uint8\_t)을 사용하여 학습 데이터를 저장할 수도 있습니다. (???)      

   -  특성변수를 사전 정렬하는 데 드는 추가적 정보를 저장할 필요도 없습니다. (???)

-  **Reduce communication cost for distributed learning**

희소 최적화 (Sparse Optimization)
------------------------------

-  희소행렬의 변수 (sparse feature)에 대한 히스토그램을 만들 때에도 오직 '2 * 0이 아닌 데이터의 개수' (``O(2 * #non_zero_data)``)만 있으면 됩니다. 

정확도 최적화 (Accuracy Optimization)
----------------------------------

잎사귀단위의 (Best-first) 나무 성장
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

대부분의 의사결정 나무 학습 알고리즘은 아래 그림과 같이, 나무를 수준단위 (level-wise;depth-wise)로 성장시킵니다.

.. image:: ./_static/images/level-wise.png
   :align: center
   :alt: 수준단위 나무 성장을 표현하는 도표로, 가장 최선의 노드가 새로운 수준을 하나를 만들게 됨을 보여준다. 이 전략은 대칭형태의 나무를 만들어 내며, 한 수준(level) 내의 모든 노드들은 자식 노드들을 가짐으로써 추가적으로 층을 형성시킨다.

LightGBM은 잎사귀단위의 (Best-first) 증가를 보입니다\ `[7] <#references>`__. It will choose the leaf with max delta loss to grow.
Holding ``#leaf`` fixed, leaf-wise algorithms tend to achieve lower loss than level-wise algorithms.

Leaf-wise may cause over-fitting when ``#data`` is small, so LightGBM includes the ``max_depth`` parameter to limit tree depth. However, trees still grow leaf-wise even when ``max_depth`` is specified.

.. image:: ./_static/images/leaf-wise.png
   :align: center
   :alt: A diagram depicting leaf wise tree growth in which only the node with the highest loss change is split and not bother with the rest of the nodes in the same level. This results in an asymmetrical tree where subsequent splitting is happening only on one side of the tree.

Optimal Split for Categorical Features
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is common to represent categorical features with one-hot encoding, but this approach is suboptimal for tree learners. Particularly for high-cardinality categorical features, a tree built on one-hot features tends to be unbalanced and needs to grow very deep to achieve good accuracy.

Instead of one-hot encoding, the optimal solution is to split on a categorical feature by partitioning its categories into 2 subsets. If the feature has ``k`` categories, there are ``2^(k-1) - 1`` possible partitions.
But there is an efficient solution for regression trees\ `[8] <#references>`__. It needs about ``O(k * log(k))`` to find the optimal partition.

The basic idea is to sort the categories according to the training objective at each split.
More specifically, LightGBM sorts the histogram (for a categorical feature) according to its accumulated values (``sum_gradient / sum_hessian``) and then finds the best split on the sorted histogram.

Optimization in Network Communication
-------------------------------------

It only needs to use some collective communication algorithms, like "All reduce", "All gather" and "Reduce scatter", in distributed learning of LightGBM.
LightGBM implements state-of-art algorithms\ `[9] <#references>`__.
These collective communication algorithms can provide much better performance than point-to-point communication.

.. _Optimization in Parallel Learning:

Optimization in Distributed Learning
------------------------------------

LightGBM provides the following distributed learning algorithms.

Feature Parallel
~~~~~~~~~~~~~~~~

Traditional Algorithm
^^^^^^^^^^^^^^^^^^^^^

Feature parallel aims to parallelize the "Find Best Split" in the decision tree. The procedure of traditional feature parallel is:

1. Partition data vertically (different machines have different feature set).

2. Workers find local best split point {feature, threshold} on local feature set.

3. Communicate local best splits with each other and get the best one.

4. Worker with best split to perform split, then send the split result of data to other workers.

5. Other workers split data according to received data.

The shortcomings of traditional feature parallel:

-  Has computation overhead, since it cannot speed up "split", whose time complexity is ``O(#data)``.
   Thus, feature parallel cannot speed up well when ``#data`` is large.

-  Need communication of split result, which costs about ``O(#data / 8)`` (one bit for one data).

Feature Parallel in LightGBM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Since feature parallel cannot speed up well when ``#data`` is large, we make a little change: instead of partitioning data vertically, every worker holds the full data.
Thus, LightGBM doesn't need to communicate for split result of data since every worker knows how to split data.
And ``#data`` won't be larger, so it is reasonable to hold the full data in every machine.

The procedure of feature parallel in LightGBM:

1. Workers find local best split point {feature, threshold} on local feature set.

2. Communicate local best splits with each other and get the best one.

3. Perform best split.

However, this feature parallel algorithm still suffers from computation overhead for "split" when ``#data`` is large.
So it will be better to use data parallel when ``#data`` is large.

Data Parallel
~~~~~~~~~~~~~

Traditional Algorithm
^^^^^^^^^^^^^^^^^^^^^

Data parallel aims to parallelize the whole decision learning. The procedure of data parallel is:

1. Partition data horizontally.

2. Workers use local data to construct local histograms.

3. Merge global histograms from all local histograms.

4. Find best split from merged global histograms, then perform splits.

The shortcomings of traditional data parallel:

-  High communication cost.
   If using point-to-point communication algorithm, communication cost for one machine is about ``O(#machine * #feature * #bin)``.
   If using collective communication algorithm (e.g. "All Reduce"), communication cost is about ``O(2 * #feature * #bin)`` (check cost of "All Reduce" in chapter 4.5 at `[9] <#references>`__).

Data Parallel in LightGBM
^^^^^^^^^^^^^^^^^^^^^^^^^

We reduce communication cost of data parallel in LightGBM:

1. Instead of "Merge global histograms from all local histograms", LightGBM uses "Reduce Scatter" to merge histograms of different (non-overlapping) features for different workers.
   Then workers find the local best split on local merged histograms and sync up the global best split.

2. As aforementioned, LightGBM uses histogram subtraction to speed up training.
   Based on this, we can communicate histograms only for one leaf, and get its neighbor's histograms by subtraction as well.

All things considered, data parallel in LightGBM has time complexity ``O(0.5 * #feature * #bin)``.

Voting Parallel
~~~~~~~~~~~~~~~

Voting parallel further reduces the communication cost in `Data Parallel <#data-parallel>`__ to constant cost.
It uses two-stage voting to reduce the communication cost of feature histograms\ `[10] <#references>`__.

GPU 지원
-----------

기여해 주신 `@huanzhang12 <https://github.com/huanzhang12>`__ 님 감사합니다. 더 자세한 것은 `[11] <#references>`__ 을 참고 부탁드립니다. 

- `GPU 설치 <./Installation-Guide.rst#build-gpu-version>`__

- `GPU 튜토리얼 <./GPU-Tutorial.rst>`__

Applications and Metrics
------------------------

LightGBM은 다음과 같은 활용이 가능합니다:

-  회귀, 목적함수는 L2 loss

-  이진 분류, 목적함수는 logloss

-  다중 분류

-  크로스 엔트로피, 목적함수는 logloss 그리고 이진 클래스가 아닌 경우에 대해서도 학습을 지원함

-  LambdaRank, 목적함수는 LambdaRank with NDCG

LightGBM이 지원하는 평가 매트릭스는 다음과 같습니다:

-  L1 loss

-  L2 loss

-  Log loss

-  Classification error rate

-  AUC

-  NDCG

-  MAP

-  Multi-class log loss

-  Multi-class error rate

-  AUC-mu ``(new in v3.0.0)``

-  Average precision ``(new in v3.1.0)``

-  Fair

-  Huber

-  Poisson

-  Quantile

-  MAPE

-  Kullback-Leibler

-  Gamma

-  Tweedie

더 자세한 것은 `Parameters <./Parameters.rst#metric-parameters>`__ 을 참고 부탁드립니다.

기타 피쳐
--------------

-  나무가 잎사귀단위로 증가하면서도 ``max_depth`` 를 제한시킴

-  `DART <https://arxiv.org/abs/1505.01866>`__

-  L1/L2 정규화

-  배깅 (Bagging)

-  컬럼 (특성변수) 부분추출

-  Continued train with input GBDT model

-  Continued train with the input score file

-  가중치 학습

-  Validation metric output during training

-  다수의 검증 (validation) 데이터

-  다수의 평가 매트릭스

-  Early stopping (학습, 예측 모두)

-  Prediction for leaf index

더 자세한 것은 `Parameters <./Parameters.rst>`__ 을 참고 부탁드립니다.

참고문헌
----------

[1] Guolin Ke, Qi Meng, Thomas Finley, Taifeng Wang, Wei Chen, Weidong Ma, Qiwei Ye, Tie-Yan Liu. "`LightGBM\: A Highly Efficient Gradient Boosting Decision Tree`_." Advances in Neural Information Processing Systems 30 (NIPS 2017), pp. 3149-3157.

[2] Mehta, Manish, Rakesh Agrawal, and Jorma Rissanen. "SLIQ: A fast scalable classifier for data mining." International Conference on Extending Database Technology. Springer Berlin Heidelberg, 1996.

[3] Shafer, John, Rakesh Agrawal, and Manish Mehta. "SPRINT: A scalable parallel classifier for data mining." Proc. 1996 Int. Conf. Very Large Data Bases. 1996.

[4] Ranka, Sanjay, and V. Singh. "CLOUDS: A decision tree classifier for large datasets." Proceedings of the 4th Knowledge Discovery and Data Mining Conference. 1998.

[5] Machado, F. P. "Communication and memory efficient parallel decision tree construction." (2003).

[6] Li, Ping, Qiang Wu, and Christopher J. Burges. "Mcrank: Learning to rank using multiple classification and gradient boosting." Advances in Neural Information Processing Systems 20 (NIPS 2007).

[7] Shi, Haijian. "Best-first decision tree learning." Diss. The University of Waikato, 2007.

[8] Walter D. Fisher. "`On Grouping for Maximum Homogeneity`_." Journal of the American Statistical Association. Vol. 53, No. 284 (Dec., 1958), pp. 789-798.

[9] Thakur, Rajeev, Rolf Rabenseifner, and William Gropp. "`Optimization of collective communication operations in MPICH`_." International Journal of High Performance Computing Applications 19.1 (2005), pp. 49-66.

[10] Qi Meng, Guolin Ke, Taifeng Wang, Wei Chen, Qiwei Ye, Zhi-Ming Ma, Tie-Yan Liu. "`A Communication-Efficient Parallel Algorithm for Decision Tree`_." Advances in Neural Information Processing Systems 29 (NIPS 2016), pp. 1279-1287.

[11] Huan Zhang, Si Si and Cho-Jui Hsieh. "`GPU Acceleration for Large-scale Tree Boosting`_." SysML Conference, 2018.

.. _LightGBM\: A Highly Efficient Gradient Boosting Decision Tree: https://papers.nips.cc/paper/6907-lightgbm-a-highly-efficient-gradient-boosting-decision-tree.pdf

.. _On Grouping for Maximum Homogeneity: https://www.tandfonline.com/doi/abs/10.1080/01621459.1958.10501479

.. _Optimization of collective communication operations in MPICH: https://www.mcs.anl.gov/~thakur/papers/ijhpca-coll.pdf

.. _A Communication-Efficient Parallel Algorithm for Decision Tree: http://papers.nips.cc/paper/6381-a-communication-efficient-parallel-algorithm-for-decision-tree

.. _GPU Acceleration for Large-scale Tree Boosting: https://arxiv.org/abs/1706.08359
