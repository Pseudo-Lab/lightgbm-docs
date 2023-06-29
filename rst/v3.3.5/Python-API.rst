파이썬 API
==========

.. currentmodule:: lightgbm

데이터 구조 API
------------------

.. autosummary::
    :toctree: pythonapi/

    Dataset
    Booster
    CVBooster
    Sequence

훈련 API
------------

.. autosummary::
    :toctree: pythonapi/

    train
    cv

사이킷런 API
----------------

.. autosummary::
    :toctree: pythonapi/

    LGBMModel
    LGBMClassifier
    LGBMRegressor
    LGBMRanker

대스크 API
--------

.. versionadded:: 3.2.0

.. autosummary::
    :toctree: pythonapi/

    DaskLGBMClassifier
    DaskLGBMRegressor
    DaskLGBMRanker

콜백
---------

.. autosummary::
    :toctree: pythonapi/

    early_stopping
    log_evaluation
    record_evaluation
    reset_parameter

그래프
--------

.. autosummary::
    :toctree: pythonapi/

    plot_importance
    plot_split_value_histogram
    plot_metric
    plot_tree
    create_tree_digraph

유틸리티
---------

.. autosummary::
    :toctree: pythonapi/

    register_logger
