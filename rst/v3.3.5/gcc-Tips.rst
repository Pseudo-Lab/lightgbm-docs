GCC를 사용할 때의 권장사항
==============================

LightGBM을 학습할 때 최고 속도를 내기 위해선 ``-O3 -mtune=native`` 옵션을 사용하는 것을 권장합니다.

Intel Ivy Bridge CPU를 사용하여 1M x 1K Bosch dataset를 학습할 경우, 컴파일 옵션에 따라 다음과 같은 성능 향상을 보입니다.

+-------------------------------------+---------------------+
| 컴파일 플래그                        | 성능 지수            |
+=====================================+=====================+
| ``-O2 -mtune=core2``                | 100.00%             |
+-------------------------------------+---------------------+
| ``-O2 -mtune=native``               | 100.90%             |
+-------------------------------------+---------------------+
| ``-O3 -mtune=native``               | 102.78%             |
+-------------------------------------+---------------------+
| ``-O3 -ffast-math -mtune=native``   | 100.64%             |
+-------------------------------------+---------------------+

아래의 실험 예시에서 더 자세한 내용을 찾을 수 있습니다:

-  `Laurae++/Benchmarks <https://sites.google.com/view/lauraepp/benchmarks/xgb-vs-lgb-feb-2017>`__

-  `Laurae2/gbt\_benchmarks <https://github.com/Laurae2/gbt_benchmarks>`__

-  `Laurae's Benchmark Master Data (Interactive) <https://public.tableau.com/views/gbt_benchmarks/Master-Data?:showVizHome=no>`__

-  `Kaggle Paris Meetup #12 Slides <https://drive.google.com/file/d/0B6qJBmoIxFe0ZHNCOXdoRWMxUm8/view>`__

아래 사진은 ``-O2 --mtune=core2`` 옵션으로 컴파일 하는 기본 설정(baseline)과 그 이외의 다양한 컴파일러 옵션으로 설정했을 때의 학습 실행시간을 비교한 것입니다.
세 옵션 모두 기본 설정(baseline)보다 더 빨랐습니다. ``-O3 --mtune=native`` 옵션을 주었을 때가 가장 성능이 좋았습니다.

.. image:: ./_static/images/gcc-comparison-2.png
   :align: center
   :target: ./_static/images/gcc-comparison-2.png
   :alt: Picture with a chart grouped by compiler set of options using O2 M tune equals core2 as the baseline. All the other 3 options are faster, with O3 M tune equals native being the fastest.
