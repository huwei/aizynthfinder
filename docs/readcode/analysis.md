# aizynthfinder/analysis
##  aizynthfinder/analysis/tree_analysis.py
  这个文件是分析模块的核心，定义了 TreeAnalysis 类，用于对搜索算法（如MCTS）产生的搜索树进行深入分析。

* `TreeAnalysis` 类:
    * 初始化: 接收一个搜索树对象（MctsSearchTree 或 AndOrSearchTreeBase）和一个或多个评分器（Scorer）作为输入。评分器用于评估合成路线的优劣。
    * 单目标与多目标分析: 能够处理单一评分标准（单目标）和多个评分标准（多目标）的场景。
    * `best()` 方法: 在单目标分析中，返回得分最高的那条合成路线。
    * `pareto_front()` 方法: 在多目标分析中，计算并返回帕累托前沿（Pareto Front）。帕累托前沿是指一组“没有其他任何解在所有目标上都比它好”的解，是多目标优化中的最优解集。
    * `sort()` 方法: 根据评分对所有找到的路线进行排序和筛选。它支持复杂的筛选逻辑，比如只返回已解决的路线，或者返回得分最高的N条路线。
    * `tree_statistics()` 方法: 计算并返回关于整个搜索树的统计信息，例如节点总数、路线总数、已解决路线数、搜索深度、最优路线的步数和起始物料等。这对于评估搜索算法的性能和结果质量非常有用。

##  aizynthfinder/analysis/routes.py
  这个文件定义了 RouteCollection 类，用于管理一组合成路线。它可以看作是 TreeAnalysis 筛选出的一系列“优秀”路线的集合。
* `RouteCollection` 类:
    * 数据持有: 封装了一组 ReactionTree 对象，以及与它们相关的节点、分数、元数据等信息。
    * 数据生成: 提供了便捷的方法来生成这组路线的多种表示形式，如：
        * dicts: 字典格式，用于数据交换。
        * jsons: JSON字符串，用于存储和传输。
        * images: 图片格式，用于可视化展示。
    * 评分与重排:
        * compute_scores(): 使用新的评分器为集合中的所有路线计算额外的分数。
        * rescore(): 使用新的评分器重新对路线进行评分，并根据新分数对集合进行排序。
    * 聚类分析 (`cluster` 方法): 这是一个高级功能。它能计算路线之间的相似性（距离），并根据距离将相似的路线聚集成簇（cluster）。这有助于发现不同类型的合成策略。
    * 合并与可视化 (`combined_reaction_trees` 方法): 能够将集合中的所有路线合并成一个大的、共享节点的反应网络图，并可以生成一个交互式的 vis.js 页面来可视化这个网络。

##  aizynthfinder/analysis/utils.py
  这个文件包含了一些辅助类和函数，供 analysis 包中的其他模块使用，以保持主模块代码的整洁。

* `RouteSelectionArguments` (Dataclass): 一个简单的数据类，用于封装从 TreeAnalysis 中筛选路线时的参数，如返回路线的最小数量（nmin）、最大数量（nmax）等。
* `CombinedReactionTrees` 类: 实现了将多个 ReactionTree 合并成一个单一图的算法。
    * 它通过识别不同路线中相同的反应步骤，将它们在图中合并，从而创建一个紧凑的反应网络视图。
    * 提供了 to_dict 和 to_visjs_page 方法，用于将合并后的图导出为数据或交互式可视化页面。


总结来说，aizynthfinder/analysis 目录提供了一套完整的工具链，用于处理和理解 AiZynthFinder 的搜索结果：
1. tree_analysis.py 从原始的搜索树中提取、排序和筛选出有价值的合成路线。
2. routes.py 对这些筛选出的路线进行集合管理，支持进一步的评分、聚类和批量转换。
3. utils.py 提供了底层的合并与可视化算法支持。