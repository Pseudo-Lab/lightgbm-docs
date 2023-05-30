파이썬-패키지 입문
===============================

이 문서에서는 LightGBM 파이썬 패키지의 기본적인 설명을 제공합니다.

**그 외 도움이 될만한 링크 목록**

-  `파이썬 예시 <https://github.com/microsoft/LightGBM/tree/master/examples/python-guide>`__

-  `파이썬 API <./Python-API.rst>`__

-  `파라미터 튜닝 (Parameters Tuning) <./Parameters-Tuning.rst>`__

설치
------------

LightGBM을 설치하는 가장 일반적인 방법은 pip를 사용하는 것입니다:

::

    pip install lightgbm

자세한 설치 가이드는 `Python-package`_ 폴더를 참고하세요.

설치 완료 여부를 확인하기 위해, 파이썬에서 ``import lightgbm`` 를 해보세요:

::

    import lightgbm as lgb


데이터 인터페이스
--------------

LightGBM 파이썬 모듈은 아래와 같은 곳에서 데이터를 불러올 수 있습니다:

-  LibSVM (0-기반) / TSV / CSV 형식의 텍스트 파일

-  NumPy 2차원 배열, pandas 데이터프레임, H2O 데이터테이블의 프레임, SciPy의 희소 행렬

-  LightGBM 이진 파일

-  LightGBM ``Sequence`` 객체

이 데이터는 ``Dataset`` 객체로 저장됩니다.

이번 장에서 다루는 많은 예제들은 ``numpy``의 기능을 사용합니다. 예제를 실행하려면 세션에서 ``numpy``를 임포트해야 합니다.

.. code:: python

    import numpy as np

**LibSVM (0-기반) 텍스트 파일 또는 LightGBM 이진 파일을 Dataset으로 가져오려면 다음과 같이 하세요:** 

.. code:: python

    train_data = lgb.Dataset('train.svm.bin')

**numpy 배열을 Dataset으로 가져오려면 다음과 같이 하세요:**

.. code:: python

    data = np.random.rand(500, 10)  # 500 entities, each contains 10 features
    label = np.random.randint(2, size=500)  # binary target
    train_data = lgb.Dataset(data, label=label)

**scipy.sparse.csr\_matrix를 Dataset으로 가져오려면 다음과 같이 하세요:**

.. code:: python

    import scipy
    csr = scipy.sparse.csr_matrix((dat, (row, col)))
    train_data = lgb.Dataset(csr)

**Sequence 객체를 가져오려면 다음과 같이 하세요:**

이진 파일을 읽기 위해 ``Sequence`` 인터페이스를 사용할 수 있습니다. 다음은 ``h5py`` 를 사용하여 HDF5 파일을 읽는 예제입니다.

.. code:: python

    import h5py

    class HDFSequence(lgb.Sequence):
        def __init__(self, hdf_dataset, batch_size):
            self.data = hdf_dataset
            self.batch_size = batch_size

        def __getitem__(self, idx):
            return self.data[idx]

        def __len__(self):
            return len(self.data)

    f = h5py.File('train.hdf5', 'r')
    train_data = lgb.Dataset(HDFSequence(f['X'], 8192), label=f['Y'][:])

``Sequence`` 인터페이스 사용 상의 특성:

- 데이터를 무작위로 추출하기에 데이터셋 전체를 훑지 않습니다.
- 데이터를 배치 (Batch) 단위로 읽으므로 ``Dataset`` 객체를 구성할 때 메모리를 절약합니다.
- 여러 개의 데이터 파일로부터 ``Dataset`` 을 생성할 수 있습니다.

``Sequence`` `API doc <./Python-API.rst#data-structure-api>`__ 를 참고해주세요.

구체적인 예시는 `dataset_from_multi_hdf5.py <https://github.com/microsoft/LightGBM/blob/master/examples/python-guide/dataset_from_multi_hdf5.py>`__ 에 있습니다.

**Dataset을 LightGBM 이진 파일 형태로 저장하는 게 불러오는 속도가 더 빠릅니다:**

.. code:: python

    train_data = lgb.Dataset('train.svm.txt')
    train_data.save_binary('train.bin')

**검증 데이터 (Validation Data) 생성하기:**

.. code:: python

    validation_data = train_data.create_valid('validation.svm')

or

.. code:: python

    validation_data = lgb.Dataset('validation.svm', reference=train_data)

LightGBM에서 검증 데이터 (Validation Data)는 학습 데이터와 같은 방식으로 전처리를 해야 합니다.
.. 'the validation data should be aligned with training data.'' 관련 내용
.. https://stackoverflow.com/questions/56804254/what-does-lightgbm-python-dataset-reference-parameter-mean

**특정한 변수 이름과 범주형 변수:**

.. code:: python

    train_data = lgb.Dataset(data, label=label, feature_name=['c1', 'c2', 'c3'], categorical_feature=['c3'])

LightGBM 범주형 변수를 입력으로 바로 사용할 수 있습니다.
원핫 인코딩 (One-Hot Encoding)으로 변환할 필요가 없고, 원핫 인코딩보다 훨씬 빠릅니다. (약 8배 빠른 속도)

**참고**: ``Dataset`` 을 생성하기 전에 범주형 변수를 ``int`` 형으로 바꿔야 합니다.

**필요에 따라 가중치 (Weights)를 설정할 수 있습니다:**

.. code:: python

    w = np.random.rand(500, )
    train_data = lgb.Dataset(data, label=label, weight=w)

or

.. code:: python

    train_data = lgb.Dataset(data, label=label)
    w = np.random.rand(500, )
    train_data.set_weight(w)

그리고 초기 점수를 설정하려면 ``Dataset.set_init_score()`` 를 사용하면 되고, 순위 문제 (Ranking Task)에서 위한 그룹/질문 데이터를 설정하기 위해 ``Dataset.set_group()`` 을 사용할 수 있습니다.

**효율적인 메모리 사용법:**

LightGBM의 ``Dataset`` 객체는 메모리 효율성이 매우 높고, 불연속적인 이진수 형태로만 저장하면 됩니다.
그러나, Numpy/배열/Pandas 객체는 메모리 비용이 많이 듭니다. 
만약 메모리 사용량에 신경 쓴다면, 다음과 같은 방식으로 메모리를 절약할 수 있습니다:

1. ``Dataset`` 을 만들 때 ``free_raw_data=True`` 옵션을 설정하여라. (기본값은 ``True`` 이다.)

2. ``Dataset`` 이 생성된 직후에 명시적으로 ``raw_data=None`` 을 설정하여라.

3. ``gc`` (가비지 컬렉터)를 호출해라.


파라미터 설정
--------------------

LightGBM에서 `Parameters <./Parameters.rst>`__ 를 설정하기 위해 딕셔너리 (Dictionary) 타입을 이용할 수 있습니다.
예를 들어:

-  부스터 파라미터 (Booster Parameters):

   .. code:: python

       param = {'num_leaves': 31, 'objective': 'binary'}
       param['metric'] = 'auc'

-  또한 여러 개의 평가 지표 (Eval Metrics)를 사용할 수도 있습니다:

   .. code:: python

       param['metric'] = ['auc', 'binary_logloss']


학습
--------------

모델을 학습시키기 위해선 파라미터 목록과 데이터셋이 필요합니다:

.. code:: python

    num_round = 10
    bst = lgb.train(param, train_data, num_round, valid_sets=[validation_data])

학습 이후 모델을 저장할 수 있습니다:

.. code:: python

    bst.save_model('model.txt')

학습된 모델을 JSON 형태로 복사할 수도 있습니다:

.. code:: python

    json_model = bst.dump_model()

저장된 모델을 불러올 수도 있습니다:

.. code:: python

    bst = lgb.Booster(model_file='model.txt')  # init model


교차 검증 (Cross Validation)
-----------------------------------

5번의 교차 검증 (Cross Validation)을 하여 학습하는 건 다음과 같이 할 수 있습니다:

.. code:: python

    lgb.cv(param, train_data, num_round, nfold=5)


조기 학습 종료 (Early Stopping)
----------------------------------------

만약 검증 데이터셋 (Validation Set)을 가지고 있다면, 궁긍적인 부스팅 횟수를 찾기 위해 조기 학습 종료 (Early Stopping)를 해볼 수 있습니다.
조기 학습 종료 (Early Stopping)는 적어도 하나의 ``valid_sets`` 안에 있는 데이터셋을 필요로 합니다. 만약 한 개보다 많다면, 학습 데이터를 제외한 모든 것이 사용됩니다:

.. code:: python

    bst = lgb.train(param, train_data, num_round, valid_sets=valid_sets, early_stopping_rounds=5)
    bst.save_model('model.txt', num_iteration=bst.best_iteration)

모델은 검증 점수가 개선되지 않을 때까지 학습을 진행합니다.
학습을 계속하려면 검증 점수는 최소한 모든 ``early_stopping_rounds`` 마다 향상되어야 합니다.

만약 ``early_stopping_rounds`` 옵션으로 조기 학습 종료 (Early Stopping)가 진행되었다면, 최고 성능을 가진 반복 횟수 인덱스가 ``best_iteration`` 영역에 저장됩니다.
``train()`` 은 최고 성능일 때의 모델을 반환값으로 준다는 걸 유의하세요.

이는 최소화(L2, 로그 손실 등)와 최대화(NDCG, AUC 등)를 하는 두 가지 평가 지표 모두에서 작동합니다.
둘 이상의 평가 지표를 지정하는 경우, 모든 지표가 조기 학습 종료 (Early Stopping)에 사용된다는 걸 유의해 주세요.
그러나 ``param`` 이나 ``early_stopping`` 콜백 생성자에게 ``first_metric_only=True`` 옵션값을 넘겨주므로써 위의 동작을 변경하고 LightGBM이 조기 학습 종료에 대한 첫 번째 평가 지표만 확인하도록 할 수 있습니다. 


예측 (Prediction)
----------------------------

학습됬거나 불러온 모델은 데이터셋에 대한 예측을 할 수 있습니다:

.. code:: python

    # 7 entities, each contains 10 features
    data = np.random.rand(7, 10)
    ypred = bst.predict(data)

만약 조기 학습 종료 (Early Stopping)가 학습 중에 진행되었다면, ``bst.best_iteration`` 옵션을 이용하여 가장 성능이 좋었던 걸로 예측할 수 있습니다.
.. code:: python

    ypred = bst.predict(data, num_iteration=bst.best_iteration)

.. _파이썬-패키지: https://github.com/microsoft/LightGBM/tree/master/python-package
