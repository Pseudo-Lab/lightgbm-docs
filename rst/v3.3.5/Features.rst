Features
========

본 글은 LightGBM이 어떻게 작동하는가에 대한 개요입니다 \ `[1] <#references>`__. 독자가 의사결정나무 부스팅 알고리즘 (decision tree boosting algorithms)과 친숙하다는 전제 하에, LightGBM이 다른 부스팅 패키지와 어떻게 다른지에 초점을 맞추었습니다. 알고리즘에 대한 상세한 설명은 인용글 혹은 소스코드를 참고해 주세요.


연산속도와 메모리 사용량 최적화 
--------------------------------------

많은 부스팅 패키지들은 사전 정렬 기반의 알고리즘을 사용하여 의사결정나무를 학습시킵니다 \ `[2, 3] <#references>`__ (예. xgboost의 기본설정 알고리즘). 이 해법은 간단한 대신 최적화가 쉽지 않습니다.

LightGBM은 히스토그램 기반의 알고리즘을 사용함으로써\ `[4, 5, 6] <#references>`__, 연속형 특성변수 (feature; attribute)를 이산형으로 범주화 시킵니다. 이는 학습 속도를 올리며 메모리 사용량을 감소시킵니다. 히스토그램 기반의 알고리즘이 갖는 우위는 다음과 같습니다:

-  **각 분할(split)의 정보획득(gain) 연산비용 감소**

   -  사전 정렬 기반의 알고리즘은 ``0(#data)`` 의 시간 복잡도(time complexity)를 가집니다 (인풋 데이터 양과 연산속도의 관계가 선형적). 
   
   -  히스토그램을 계산하는 것도 ``O(#data)`` 의 시간 복잡도를 가지지만, 이는 오직 빠른 합산 연산에서만 일어납니다. 히스토그램이 일단 만들어지면, 히스토그램 기반 알고리즘은 ``O(#bins)`` 의 시간 복잡도를 가지며, 여기서 #bins는 #data보다 훨씬 작으므로 연산비용도 훨씬 작습니다.  

-  **히스토그램 감법(뺄셈)을 통한 추가적 속도향상**

   -  이진(binary) 나무에서 한 잎사귀(leaf)의 히스토그램을 계산할 때 그 잎사귀의 부모노드와 이웃 잎사귀의 히스토그램 간 차를 이용합니다. 

   -  따라서 이 나무는 오직 하나의 잎사귀에 대해서만 히스토그램을 만들게 되며, 더 작은 데이터(``#data``)를 가진 이웃 잎사귀가 히스토그램을 만듭니다. 그 다음, 나머지 한 잎사귀의 히스토그램은 히스토그램 감법을 통해 작은 연산비용 (``O(#bins)``)으로 구해집니다. 
   
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

LightGBM은 (최고의 1등) 잎사귀단위로 성장합니다. \ `[7] <#references>`__. 알고리즘은 최대 델타 손실 (maximun delta loss)을 만드는 잎사귀가 자라도록 합니다. 잎사귀수 (``#leaf``)는 둘 다 같다고 했을 때, 잎사귀단위 알고리즘이 수준단위 알고리즘보다 손실이 작은 편입니다.

잎사귀단위 증가는 데이터 (`#data``)가 작을 때 과적합을 일으킬 수 있으므로 LightGBM은 이 나무의 깊이를 제한시키는 ``max_depth``라는 파라미터를 갖고있습니다. 하지만, ``max_depth``가 지정되어도 여전히 나무는 잎사귀 단위의 성장을 계속합니다. 


.. image:: ./_static/images/leaf-wise.png
   :align: center
   :alt: A diagram depicting leaf wise tree growth in which only the node with the highest loss change is split and not bother with the rest of the nodes in the same level. This results in an asymmetrical tree where subsequent splitting is happening only on one side of the tree.

범주형 특성변수에 최적의 분할 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

범주형 변수는 보통 원-핫 인코딩 (one-hot encoding)으로 변환하기는 하지만, 사실 나무 학습기에는 최선책이 아닙니다. 특히, 카디널리티가 높은 범주형 변수를 원-핫 인코딩으로 변환한 나무모형은 불균형한 경향이 있으며, 준수한 정확도를 얻으려면 매우 깊게 자라야 합니다.      

원-핫 인코딩 대신, 최적의 해법은 범주들을 두 개의 부분집합으로 나눠서 분기 (split)하는 것입니다. 예를 들어 ``k`` 개의 범주를 가진 변수라면, 총 ``2^(k-1) - 1`` 개의 분할 경우의 수가 생깁니다. 회귀나무에서는 이 연산에 대해 효율적인 해법을 갖고 있습니다 \ `[8] <#references>`__. 회귀나무에서는 범주형 변수의 최적의 분할을 찾는 데 약 ``O(k * log(k))`` 정도가 걸립니다.  

기본 아이디어는 각 분할점에서 학습 목적에 맞게 범주들을 정렬해주는 것입니다. 구체적으로, LightGBM은 (범주형 변수에 대해) 히스토그램을 ``sum_gradient / sum_hessian`` 의 누적값을 기준으로 정렬시키고 나서 정렬된 히스토그램에서 최고의 분할점을 찾습니다.    

네트워크 커뮤니케이션에서의 최적화 (???)
-------------------------------------

LightGBM의 분산학습에서는 "All reduce", "All gather", "Reduce scatter" 같은 몇 개의 collective communication 알고리즘만 사용하면 됩니다. (???)
LightGBM은 최신 알고리즘을 사용했습니다 \ `[9] <#references>`__. (???)
이 collective communication 알고리즘들은 point-to-point communication보다 훨씬 좋은 성능을 제공합니다. (???)

.. _병렬학습에서의 최적화 (Optimization in Parallel Learning):

분산학습에서의 최적화 
------------------------------------

LightGBM이 제공하는 분산학습 알고리즘은 아래와 같습니다.

특성 병렬 (Feature Parallel)
~~~~~~~~~~~~~~~~~~~~~~~~~~

전통적 알고리즘
^^^^^^^^^^^^^^^^^^^^^

특성 병렬은 의사결정 나무에서 "최고의 분할점 찾기"를 병렬처리 하는 것을 목표로 합니다. 전통적 특성 병렬 과정: 

1. 데이터를 수직 방향으로 분할합니다 (기계들은 서로 다른 피처셋 (feature set)을 갖고 있음).

2. 작업기기가 국소 피처셋에서 국소 최적 분할점 (local best split) {특성변수, 임계치}을 찾습니다.

3. 서로 국소 최적 분할점들에 대해 통신 후 가장 최적의 값을 선택합니다.

4. 가장 좋은 분할점을 찾은 기계가 분할을 수행하고, 그 결과 데이터를 다른 기기에 전달합니다.

5. 다른 작업기기들은 전달받은 데이터에 따라 데이터를 분할합니다.

전통적 특성 병렬의 한계점:

- 시간 복잡도가 ``O(#data)``를 따라서 "분할"에 속도를 낼 수 없기 때문에, 계산비용이 있습니다.
   따라서, 데이터 사이즈 (``#data``)가 클떄 특성 병렬은 속도를 잘 낼 수 없습니다. 

- 분할 결과에 대해 소통이 필요하며, 이는 대략 ``O(#data / 8)`` (한 데이터 당 1 bit) 정도를 소모시킵니다.

LightGBM의 특성 병렬
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

특성 병렬은 데이터 사이즈 (``#data``)가 크면 속도를 잘 낼 수 없기 때문에, LightGBM은 작은 변화를 주었습니다: 데이터를 수직으로 분할하는 것이 아니라, 모든 작업기기들은 전체 데이터셋을 갖고 있습니다. 그러므로, LightGBM은 분할 결과 데이터를 통신할 필요가 없습니다. 모든 작업기기가 어떻게 데이터를 분할하는지 알고있기 때문입니다. 그리고 데이터가 더 커지지는 않을 것이기 때문에, 각 장치마다 전체 데이터셋을 갖고 있는 것은 합리적이라 할 수 있습니다.       
LightGBM의 특성 병렬 과정:

1. 작업기기들이 국소 피처셋에서 국소 최적 분할점 (local best split) {특성변수, 임계치}을 찾습니다.

2. 서로 국소 최적 분할점들에 대해 통신 후 가장 최적의 값을 선택합니다.

3. 가장 최적의 분할을 수행합니다.

그러나, 이 특성 병렬 알고리즘도 데이터 (#data)가 클 때는 여전히 "분할"에 드는 연산비용의 부담이 있습니다. 그래서 데이터 (#data)가 클 때는 데이터 병렬을 사용하는 것이 낫습니다.   

데이터 병렬
~~~~~~~~~~~~~

전통적 알고리즘
^^^^^^^^^^^^^^^^^^^^^

데이터 병렬은 전체 의사결정 학습을 병렬처리 하는 것을 목표로 합니다. 데이터 병렬의 과정:

1. 데이터를 수평으로 분할합니다.

2. 작업기기들이 국소 데이터를 사용하여 국소 히스토그램을 만듭니다. (???)

3. 모든 국소 히스토그램들로부터 전역 (global) 히스토그램들을 병합합니다. (???)

4. 병합한 글로벌 히스토그램들로부터 가장 최적의 분할점을 찾고, 분할작업들을 수행합니다. (???)

전통적 데이터 병렬의 한계점:

-  높은 커뮤니케이션 비용.
   만약 point-to-point 커뮤니케이션 알고리즘을 사용한다면, 한 장치 당 커뮤니케이션 비용은 대략 ``O(#machine * #feature * #bin)`` 이 듭니다.
   만약 collective 커뮤니케이션 알고리즘 (예. "All Reduce")을 사용한다면, 이 비용은 약 ``O(2 * #feature * #bin)`` 정도입니다 ("All Reduce" 가격 4.5 at `[9] <#references>`__).

LightGBM의 데이터 병렬
^^^^^^^^^^^^^^^^^^^^^^^^^

LightGBM은 데이터 병렬에서 커뮤니케이션 비용을 줄였습니다.  

1. "모든 국소 히스토그램들로부터 전역 (global) 히스토그램들을 병합"하는 것 대신, LightGBM은 "Reduce Scatter"을 사용하여 서로 다른 (포개지지 않는) 변수들의 히스토그램들을 병합합니다. 그 다음, 작업장치들은 국소 병합 히스토그램들 내에서 국소 최적 분할점을 찾고 전역 최적 분할점을 동기화 합니다. (???)
   
2. 앞에서 언급했듯이, LightGBM은 학습 속도를 높이기 위해 히스토그램 감법 (subtraction)을 사용합니다. 이를 기반으로, 알고리즘은 한쪽 잎사귀에 대해서만 히스토그램을 연산하면 되는 것이고, 이웃의 히스토그램 또한 감법을 이용하여 구할 수 있습니다.

모든 것을 고려했을 때, LightGBM의 데이터 병렬은 ``O(0.5 * #feature * #bin)``의 시간복잡도를 가집니다.

투표 병렬 (Voting Parallel) (???)
~~~~~~~~~~~~~~~~~~~~~~~~

투표 병렬은 `Data Parallel <#data-parallel>`__ 에서의 커뮤니케이션 비용을 constant cost로 보다 크게 줄여줍니다. (???)

특성변수 히스토그램의 커뮤니케이션 비용을 줄이기 위해 2단계 투표를 사용합니다 \ `[10] <#references>`__.

GPU 지원
-----------

기여해 주신 `@huanzhang12 <https://github.com/huanzhang12>`__ 님 감사합니다. 더 자세한 것은 `[11] <#references>`__ 을 참고 부탁드립니다. 

- `GPU 설치 <./Installation-Guide.rst#build-gpu-version>`__

- `GPU 튜토리얼 <./GPU-Tutorial.rst>`__

응용, 평가 매트릭스
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

-  나무가 잎사귀단위로 증가하면서도 ``max_depth`` 로 제한

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
