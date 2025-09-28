# aizynthfinder/chem 代码分析
##  aizynthfinder/chem/mol.py
这个文件定义了如何表示和操作分子。它主要是对 RDKit 分子对象的封装，提供了更便捷的接口。
* `Molecule` 类: 这是最基础的分子类。
    * 可以从 SMILES 字符串或 RDKit 的 Mol 对象创建。
    * 提供了方便的属性来获取分子的关键标识，如 smiles (SMILES字符串), inchi (国际化学标识符), 和 inchi_key (InChI的哈希值)。inchi_key 通常用来唯一地比较和识别分子。
    * 能够计算分子指纹（fingerprint 方法），这对于机器学习模型和相似性比较非常重要。
    * 提供了处理原子映射（atom mapping）的工具，这在追踪反应过程中原子的变化时至关重要。
* `TreeMolecule` 类: 继承自 Molecule，是为逆合成树（retrosynthesis tree）设计的特殊分子类。
    * 它额外记录了分子在树中的“父节点”（parent）以及它是由哪一步转换（transform）生成的。
    * 它特别关注原子映射的继承，确保从一个分子（产物）到其前体（反应物）的原子对应关系能够被正确追踪。
* `UniqueMolecule` 类: 一个特殊的 Molecule 子类，它的哈希值是基于对象的ID，因此每个实例都是独一无二的，即使它们代表相同的化学结构。

##  aizynthfinder/chem/reaction.py
  这个文件定义了如何表示和操作化学反应，特别是逆合成反应。
  * `RetroReaction` (抽象基类): 定义了所有逆合成反应的通用接口。
      * 一个逆合成反应作用于一个分子（mol），并产生一组或多组可能的反应物（reactants）。
      * 它拥有 reactants 属性，当首次访问时会触发反应的应用（_apply 方法）来计算反应物。
  * `TemplatedRetroReaction` 类: RetroReaction 的一个具体实现，它使用一个化学反应模板（以 SMARTS 字符串的形式）来预测反应物。
      * 它利用 RDChiral 或 RDKit 内置的功能来执行逆合成反应。这是实现基于模板的逆合成预测的核心。
  * `SmilesBasedRetroReaction` 类: 另一个 RetroReaction 的实现，它不是通过模板计算反应物，而是在创建时就直接通过SMILES字符串指定了反应物。
  * `FixedRetroReaction` 类: 一个固定的、不可变的反应表示，通常用于存储或展示已经确定的反应步骤。它的反应物是固定的，不能动态计算。

###  aizynthfinder/chem/serialization.py
  这个文件负责序列化和反序列化 Molecule 和 Reaction 对象。序列化是将复杂的 Python 对象转换成可以轻松存储（如存为JSON文件）或传输的简单数据格式（如字典）的过程。
  * `MoleculeSerializer` 和 `MoleculeDeserializer`: 这两个类协同工作，用于将 Molecule 对象（及其子类 TreeMolecule）转换成字典，以及从字典恢复成原始的 Molecule
    对象。它们通过给每个独特的分子对象分配一个ID来处理对象间的引用关系（比如 TreeMolecule 的 parent 属性）。
  * `serialize_action` 和 `deserialize_action`: 这两个函数用于序列化和反序列化 RetroReaction 对象。它们利用 MoleculeSerializer
    来处理反应中涉及的分子，从而将整个反应步骤保存为可存储的格式。

  总结来说，aizynthfinder/chem 目录构建了 AiZynthFinder 的化学知识基础：
  1. mol.py 定义了“物质”（分子）。
  2. reaction.py 定义了“变化”（反应）。
  3. serialization.py 定义了如何“记录和读取”这些物质和变化。