快速入门指南
=============================

本文档是 LightGBM CLI 版本的快速入门指南。

参考 `安装指南 <./Installation-Guide.rst>`__ 先安装 LightGBM 。

**其他有帮助的链接列表**

-  `参数 <./Parameters.rst>`__

-  `参数调整 <./Parameters-Tuning.rst>`__

-  `Python 包快速入门 <./Python-Intro.rst>`__

-  `Python API <./Python-API.rst>`__

训练数据格式
-----------------------------

LightGBM 支持 `CSV`_, `TSV`_ 和 `LibSVM`_ 格式的输入数据文件。

Label 是第一列的数据，文件中是不包含 header（标题） 的。

类别特征支持
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

12/5/2016 更新:

LightGBM 可以直接使用 categorical feature（类别特征）（不需要单独编码）。
`Expo data`_ 实验显示，与 one-hot 编码相比，其速度提高了 8 倍。

有关配置的详细信息，请参阅 `参数 <./Parameters.rst>`__ 章节。

权重和 Query/Group 数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LightGBM 也支持加权训练，它需要一个额外的 `加权数据 <./Parameters.rst#io-parameters>`__ 。
它需要额外的 `query 数据 <./Parameters.rst#io-parameters>`_ 用于排名任务。

11/3/2016 更新:

1. 现在支持 header（标题）输入

2. 可以指定 label 列，权重列和 query/group id 列。
   索引和列都支持

3. 可以指定一个被忽略的列的列表

参数快速查看
--------------------------

参数格式是 ``key1=value1 key2=value2 ...`` 。
参数可以在配置文件和命令行中。

一些重要的参数如下 :

- ``config``, 默认=\ ``""``, type（类型）=string, alias（别名）=\ ``config_file``

  - 配置文件的路径

-  ``task``, 默认=\ ``train``, type（类型）=enum, options（可选）=\ ``train``, ``predict``, ``convert_model``

   -  ``train``, alias（别名）=\ ``training``, 用于训练

   -  ``predict``, alias（别名）=\ ``prediction``, ``test``, 用于预测。

   -  ``convert_model``, 用于将模型文件转换为 if-else 格式， 在 `转换模型参数 <./Parameters.rst#convert-model-parameters>`__ 中了解更多信息

-  ``application``, 默认=\ ``regression``, 类型=enum,
   可选=\ ``regression``, ``regression_l1``, ``huber``, ``fair``, ``poisson``, ``quantile``, ``quantile_l2``,
   ``binary``, ``multiclass``, ``multiclassova``, ``xentropy``, ``xentlambda``, ``lambdarank``,
   别名=\ ``objective``, ``app``

   -  回归 application

      -  ``regression_l2``, L2 损失, 别名=\ ``regression``, ``mean_squared_error``, ``mse``

      -  ``regression_l1``, L1 损失, 别名=\ ``mean_absolute_error``, ``mae``

      -  ``huber``, `Huber loss`_

      -  ``fair``, `Fair loss`_

      -  ``poisson``, `Poisson regression`_

      -  ``quantile``, `Quantile regression`_

      -  ``quantile_l2``, 与 ``quantile`` 类似, 但是使用 L2 损失

   -  ``binary``, 二进制`log loss`_ 分类 application

   -  多类别分类 application

      -  ``multiclass``, `softmax`_ 目标函数, ``num_class`` 也应该被设置

      -  ``multiclassova``, `One-vs-All`_ 二元目标函数, ``num_class`` 也应该被设置

   -  交叉熵 application

      -  ``xentropy``, 交叉熵的目标函数 (可选线性权重), 别名=\ ``cross_entropy``

      -  ``xentlambda``, 交叉熵的替代参数化, 别名=\ ``cross_entropy_lambda``

      -  label 是在 [0, 1] 间隔中的任何东西

   -  ``lambdarank``, `lambdarank`_ application

      -  在 lambdarank 任务中 label 应该是 ``int`` 类型，而较大的数字表示较高的相关性（例如，0:bad, 1:fair, 2:good, 3:perfect）

      -  ``label_gain`` 可以用来设置 ``int`` label 的 gain(weight)（增益（权重））

- ``boosting``, 默认=\ ``gbdt``, type=enum,
  选项=\ ``gbdt``, ``rf``, ``dart``, ``goss``,
  别名=\ ``boost``, ``boosting_type``

  - ``gbdt``, traditional Gradient Boosting Decision Tree（传统梯度提升决策树）

  - ``rf``, 随机森林

  - ``dart``, `Dropouts meet Multiple Additive Regression Trees`_

  - ``goss``, Gradient-based One-Side Sampling（基于梯度的单面采样）

- ``data``, 默认=\ ``""``, 类型=string, 别名=\ ``train``, ``train_data``

  - 训练数据， LightGBM 将从这个数据训练

- ``valid``, 默认=\ ``""``, 类型=multi-string, 别名=\ ``test``, ``valid_data``, ``test_data``

  - 验证/测试 数据，LightGBM 将输出这些数据的指标

  - 支持多个验证数据，使用 ``,`` 分开

- ``num_iterations``, 默认=\ ``100``, 类型=int,
  别名=\ ``num_iteration``, ``num_tree``, ``num_trees``, ``num_round``, ``num_rounds``, ``num_boost_round``

  - boosting iterations/trees 的数量

- ``learning_rate``, 默认=\ ``0.1``, 类型=double, 别名=\ ``shrinkage_rate``

  - shrinkage rate（收敛率）

- ``num_leaves``, 默认=\ ``31``, 类型=int, 别名=\ ``num_leaf``

  - 在一棵树中的叶子数量

-  ``tree_learner``, 默认=\ ``serial``, 类型=enum, 可选=\ ``serial``, ``feature``, ``data``, ``voting``, 别名=\ ``tree``

   -  ``serial``, 单个 machine tree 学习器

   -  ``feature``, 别名=\ ``feature_parallel``, feature parallel tree learner（特征并行树学习器）

   -  ``data``, 别名=\ ``data_parallel``, data parallel tree learner（数据并行树学习器）

   -  ``voting``, 别名=\ ``voting_parallel``, voting parallel tree learner（投票并行树学习器）

   -  参考 `Parallel Learning Guide（并行学习指南） <./Parallel-Learning-Guide.rst>`__ 来了解更多细节

- ``num_threads``, 默认=\ ``OpenMP_default``, 类型=int, 别名=\ ``num_thread``, ``nthread``

  - LightGBM 的线程数

  - 为了获得最好的速度，将其设置为 **real CPU cores（真实 CPU 内核）** 数量，而不是线程数（大多数 CPU 使用 `hyper-threading`_ 来为每个 CPU core 生成 2 个线程）

  - 对于并行学习，不应该使用全部的 CPU cores ，因为这会导致网络性能不佳

- ``max_depth``, 默认=\ ``-1``, 类型=int

  - 树模型最大深度的限制。
    当 ``#data`` 很小的时候，这被用来处理 overfit（过拟合）。
    树仍然通过 leaf-wise 生长

  - ``< 0`` 意味着没有限制

- ``min_data_in_leaf``, 默认=\ ``20``, 类型=int, 别名=\ ``min_data_per_leaf`` , ``min_data``, ``min_child_samples``

  - 一个叶子中的最小数据量。可以用这个来处理过拟合。

- ``min_sum_hessian_in_leaf``, 默认=\ ``1e-3``, 类型=double,
  别名=\ ``min_sum_hessian_per_leaf``, ``min_sum_hessian``, ``min_hessian``, ``min_child_weight``

  - 一个叶子节点中最小的 sum hessian 。类似于 ``min_data_in_leaf`` ，它可以用来处理过拟合。

想要了解全部的参数， 请参阅 `Parameters（参数） <./Parameters.rst>`__.

运行 LightGBM
------------------------------------

对于 Windows:

::

    lightgbm.exe config=your_config_file other_args ...

对于 Unix:

::

    ./lightgbm config=your_config_file other_args ...

参数既可以在配置文件中，也可以在命令行中，命令行中的参数优先于配置文件。例如下面的命令行会保留 ``num_trees=10`` ，并忽略配置文件中的相同参数。

::

    ./lightgbm config=train.conf num_trees=10

示例
--------------------

-  `Binary Classification（二元分类） <https://github.com/Microsoft/LightGBM/tree/master/examples/binary_classification>`__

-  `Regression（回归） <https://github.com/Microsoft/LightGBM/tree/master/examples/regression>`__

-  `Lambdarank <https://github.com/Microsoft/LightGBM/tree/master/examples/lambdarank>`__

-  `Parallel Learning（并行学习） <https://github.com/Microsoft/LightGBM/tree/master/examples/parallel_learning>`__

.. _CSV: https://en.wikipedia.org/wiki/Comma-separated_values

.. _TSV: https://en.wikipedia.org/wiki/Tab-separated_values

.. _LibSVM: https://www.csie.ntu.edu.tw/~cjlin/libsvm/

.. _Expo data: http://stat-computing.org/dataexpo/2009/

.. _Huber loss: https://en.wikipedia.org/wiki/Huber_loss

.. _Fair loss: https://www.kaggle.com/c/allstate-claims-severity/discussion/24520

.. _Poisson regression: https://en.wikipedia.org/wiki/Poisson_regression

.. _Quantile regression: https://en.wikipedia.org/wiki/Quantile_regression

.. _log loss: https://www.kaggle.com/wiki/LogLoss

.. _softmax: https://en.wikipedia.org/wiki/Softmax_function

.. _One-vs-All: https://en.wikipedia.org/wiki/Multiclass_classification#One-vs.-rest

.. _lambdarank: https://papers.nips.cc/paper/2971-learning-to-rank-with-nonsmooth-cost-functions.pdf

.. _Dropouts meet Multiple Additive Regression Trees: https://arxiv.org/abs/1505.01866

.. _hyper-threading: https://en.wikipedia.org/wiki/Hyper-threading
