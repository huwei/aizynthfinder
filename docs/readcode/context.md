# aizynthfinder/context 目录代码分析
context 目录是 AiZynthFinder
的“大脑”和“工具箱”的集合，它定义了整个逆合成路线搜索过程中的所有策略、设置和可用资源。可以说，它为搜索算法提供了决策所需的所有“上下文”信息。

它主要包含三个核心子目录：policy、scoring 和 stock，以及一个顶层配置文件 config.py 和一个基础集合类 collection.py。

##  aizynthfinder/context/collection.py

* `ContextCollection` 类: 这是一个抽象基类，为 policy, scoring, stock 这三个核心模块提供了一个统一的管理框架。它实现了：
    * 条目管理: 将不同的策略或资源（如不同的扩展模型、不同的评分函数）作为条目（item）进行存储，并可以通过唯一的键（key）来访问、添加或删除。
    * 选择机制: 提供了 select 和 deselect
      方法，允许用户在加载的多个条目中选择一个或多个来激活使用。例如，你可以加载多个扩展模型，但只选择其中一个用于当前的搜索。
    * 配置加载: 定义了 load_from_config 的抽象方法，强制子类实现从配置文件加载和初始化条目的逻辑。

##  aizynthfinder/context/config.py

* `Configuration` 类: 这是整个 AiZynthFinder 搜索的中央控制室。它封装了所有与搜索相关的设置。
    * 结构: 它聚合了 ExpansionPolicy, FilterPolicy, Stock, 和 ScorerCollection 的实例，将所有上下文管理类集中到一处。
    * 参数管理: 定义了搜索算法的参数（如迭代次数、时间限制）、后处理参数等。
    * 加载与初始化: 提供了 from_file 和 from_dict 类方法，使得可以非常方便地从一个 YAML 文件或 Python
      字典中加载所有配置，并自动初始化所有的策略和资源。这是启动一个定制化搜索的入口点。

  ---

##  aizynthfinder/context/policy/** (策略)

  这个子目录定义了在搜索树中如何扩展和筛选节点的策略。

* `policy/policies.py`:
    * `ExpansionPolicy` 类: 继承自 ContextCollection，专门用于管理扩展策略。它负责调用选中的扩展模型，为给定的分子生成可能的逆合成反应（actions）。
    * `FilterPolicy` 类: 同样继承自
      ContextCollection，用于管理筛选策略。在扩展策略生成反应后，它会调用选中的筛选器来判断这个反应是否应该被保留，如果一个反应不合理，筛选器会抛出
      RejectionException 来拒绝它。

* `policy/expansion_strategies.py`:
    * `ExpansionStrategy` (抽象基类): 定义了所有扩展策略的通用接口，核心是 get_actions 方法。
    * `TemplateBasedExpansionStrategy`: 基于反应模板的扩展策略。它加载一个AI模型（如ONNX或TensorFlow模型）和一个模板库。当给定一个分子时，模型会预测哪些
      模板最有可能适用，然后生成相应的逆合成反应。这是项目中最核心的AI应用之一。
    * `MultiExpansionStrategy`:
      一个组合策略，允许将多个不同的扩展策略（例如，一个用于常规反应，一个专门用于开环反应）组合在一起，并可以为它们分配不同的权重。

* `policy/filter_strategies.py`:
    * `FilterStrategy` (抽象基类): 定义了所有筛选策略的通用接口，核心是 apply 方法。
    * `QuickKerasFilter`: 使用一个快速的AI模型（分类器）来判断一个生成的反应是否“可行”，这可以过滤掉许多化学上不合理的步骤。
    * `BondFilter` / `FrozenSubstructureFilter`: 基于规则的筛选器。例如，BondFilter 可以确保用户指定要“冻结”的化学键在反应中不被破坏。
    * `ReactantsCountFilter`: 检查生成的反应物数量是否与模板预期的数量相符。

---

##  aizynthfinder/context/scoring/** (评分)

这个子目录定义了如何评估一个合成路线或搜索树中的一个状态（节点）的优劣。

* `scoring/collection.py`:
    * `ScorerCollection` 类: 继承自 ContextCollection，用于管理各种评分函数 (Scorer)。用户可以加载多个评分函数，并选择其中一个或多个来评估路线。

* `scoring/scorers_base.py`:
    * `Scorer` (抽象基类): 定义了所有评分函数的通用接口。一个评分函数可以接收一个 MctsNode 或 ReactionTree 对象，并返回一个数值分数。它还定义了排序逻辑。
    * Scalers: 提供了如 SquashScaler, MinMaxScaler 等工具，用于将原始的评分值归一化或缩放到一个更合适的范围。

* `scoring/scorers.py`, `scorers_mols.py`, `scorers_reactions.py`: 这些文件包含了几十个具体的评分函数实现，涵盖了从不同维度评估路线质量的方方面面：
    * 基于状态/复杂度: StateScorer (一个综合评分), NumberOfReactionsScorer (路线长度), NumberOfPrecursorsScorer (起始物料数量)。
    * 基于库存: FractionInStockScorer (起始物料在库存中的比例), PriceSumScorer (起始物料总价)。
    * 基于化学信息: DeltaSyntheticComplexityScorer (反应前后分子的合成复杂度变化), BrokenBondsScorer (是否打断了关键化学键)。
    * 基于AI模型: RouteSimilarityScorer (路线与参考路线的相似度), DeepSetScorer (使用深度学习模型评估整条路线)。

  ---

##  aizynthfinder/context/stock/** (库存)

这个子目录定义了可购买化合物的库存。库存是逆合成搜索的终点，判断一个分子是否“已解决”的关键就是看它是否在库存中。

* `stock/stock.py`:
    * `Stock` 类: 继承自 ContextCollection，用于管理一个或多个库存数据源。它可以同时查询多个库存。
    * 核心功能: 提供 __contains__ 方法（即 mol in stock），用于快速判断一个分子是否存在于选定的库存中。
    * 附加信息: 还能提供分子的价格 (price)、可用量 (amount) 和来源 (availability_string) 等信息。
    * 停止条件: 允许设置基于分子属性（如价格、分子量、原子数量）的“停止条件”，即使一个分子不在库存中，如果它满足这些简单条件，也可以被视为“已解决”。

* `stock/queries.py`:
    * `StockQueryMixin` (接口): 定义了所有库存查询类的通用接口。
    * `InMemoryInchiKeyQuery`: 最常用的查询类。它从一个文件（如 HDF5, CSV）中加载所有分子的 InChI Key 到内存中，形成一个集合，从而实现极快的查询。
    * `MongoDbInchiKeyQuery`: 支持从 MongoDB 数据库中查询分子。
    * `MolbloomFilterQuery`: 使用布隆过滤器（Bloom Filter）进行查询，这是一种空间效率极高的概率性数据结构，适用于超大规模的库存。

## 总结
context 目录是 AiZynthFinder 框架的支柱，它将搜索的“策略” (Policy)、“评价标准” (Scoring) 和 “目标” (Stock) 高度模块化和可配置化。通过 config.py
这个中央枢纽，用户可以像搭积木一样组合不同的AI模型、评分函数和库存数据，来构建一个完全定制化的逆合成路线搜索工具。