# 关于模型的分析

## 这个项目主要在以下三个方面使用AI模型：

1. Expansion Policy (扩展策略): 这是最核心的AI应用。它用于预测一个给定的目标分子可以由哪些反应物通过哪种反应得到。这本质上是一个逆合成反应预测模型。
2. Filter Policy (筛选策略): 在扩展策略提出可能的反应后，筛选模型会快速评估这个反应是否可行或有价值，过滤掉一些明显不合理的步骤。这是一个反应可行性分类模型。
3. Stock (库存查询): 虽然不完全是传统意义上的AI预测模型，但它使用数据驱动的方式（如基于HDF5的快速查询）来判断一个分子是否是可购买的起始物料。

## 模型格式和框架
根据 pyproject.toml 和 config_sample.yml 的分析，项目主要支持并使用了以下AI模型格式和框架：

* ONNX (Open Neural Network Exchange): 这是项目首选和默认的模型格式。从 config_sample.yml 中可以看到，无论是扩展模型（uspto_model.onnx）还是筛选模型（uspto_filter_model.onnx），都使用了 .onnx
  文件。pyproject.toml 中的 onnxruntime 依赖证实了项目具备运行ONNX模型的能力。
* TensorFlow/Keras: 项目也支持TensorFlow模型，特别是 Keras 保存的 .hdf5 格式。pyproject.toml 的 [project.optional-dependencies] 部分明确列出了 tensorflow，表明它是一个可选的依赖项。源代码
  aizynthfinder/utils/models.py 中也存在用于加载和处理 TensorFlow/Keras 模型的逻辑（尽管在我刚才的搜索中没有直接显示 LocalTensorflowModel，但 LocalKerasModel
  的存在暗示了这一点，并且该项目历史版本中明确支持）。

##  模型在代码中的实现

* 模型加载和推理的底层逻辑主要封装在 aizynthfinder/utils/models.py 文件中。这个模块定义了如 LocalOnnxModel 和 LocalKerasModel (或 LocalTensorflowModel)
  这样的类，它们负责从磁盘加载模型文件并提供一个统一的 predict 接口。
* aizynthfinder/context/policy/expansion_strategies.py 和 aizynthfinder/context/policy/filter_strategies.py 这两个模块则调用 models.py
  中加载的模型，将分子数据输入模型，并解释模型的输出以用于实际的搜索决策。

##  总结

  AiZynthFinder项目采用了一种灵活的、可配置的方式来使用AI模型。
* 核心用途: 通过神经网络预测逆合成反应，指导蒙特卡洛树搜索。
* 主要格式: ONNX 是其主要使用的模型格式，具有良好的跨平台和性能优势。
* 可选格式: 同时兼容 TensorFlow/Keras 模型，用户可以根据自己训练的模型格式进行配置。
* 配置方式: 用户通过 config.yml 文件来指定使用哪个模型、模型的路径以及模型类型，使得替换或添加新模型变得非常方便。
* 模型存放: aiz_models 目录是官方推荐的存放模型文件的位置。