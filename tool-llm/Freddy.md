# 总结

## 解决的问题：

现阶段的开源大语言模型不能够使用外部工具

## 主要内容：

利用已有的闭源模型ChatGPT，创建了针对API使用的功能微调大模型的数据库，并训练了能够使用API的大模型ToolLLaMA

## 实验：

### 数据集建设 - [ToolBench](https://github.com/OpenBMB/ToolBench)

[数据集文件在Google Drive](https://drive.google.com/drive/folders/1yBUQ732mPu-KclJnuQELEhtKakdXFc3J)

数据集包含API，问题，和解决路径

- 从[Rapid API Hub](https://rapidapi.com/hub)选取了质量较高的3,451工具（16,464 API)
  - 爬取了API文档的细节内容，包括功能描述，必需参数等
- 选取工具的结果
  - 包含49个粗分类种类
  - 包含细分集
- 对于选取的工具预处理
  - 对于API响应内容过长的，使用ChatGPT舍弃不重要的部分
- 根据以下选取策略选取多个API($S_i$)，使用ChatGPT根据API生成问题($I_i$)，问题可能需求一个或多个API解答
  - $I_1$ 对于一个工具选取它的全部API
  - $I_2$ 选取同一个种类的2-5个工具，每个工具最多3个API
  - $I_3$ 选取同一个集的2-5个工具，每个工具最多3个API
- 根据问题使用ChatGPT生成解决路径
  - 提供的API集合为之前选取的集合 $S_i$
  - 使用DFSDT（深度优先决策树）辅助ChatGPT

### 模型训练和评测 - ToolLLaMA

在ToolBench上微调了LLaMA模型，得到ToolLLaMA

测试时，提供的可选API是之前同样策略选取的API集合 $S_k$

## 实验发现：

![image](https://github.com/rd-wei/llm-papers/assets/64512950/4d61ea90-f837-43c8-a93d-bb310e32af29)

表格中的后缀表示API的选取范围不同
  - Inst表示测试API是训练API
  - Tool表示测试API和训练API相同种类
  - Cat表示测试API和训练API不同种类

ToolLLaMA模型结合DFSDT可以（200步以内）解决68%的单工具问题，超过了LLaMA模型的其他针对指令遵循微调的变种。老师模型ChatGPT则可以解决78%的单工具问题

对于多工具问题，根据工具跨越的领域范围不同，ToolLLaMA+DFSDT分别能够解决47%和40%，而ChatGPT+DFSDT可以解决51%和57%。

[但是怎么没见过的工具解决率更高呢？？](https://github.com/OpenBMB/ToolBench/issues/131)

# 评论

这个数据集构建的方式让ChatGPT发挥了它的工具使用能力，并记录了下来，使得后续开源模型能够获取一部分ChatGPT的工具使用能力。整个过程不需要任何人工标注或者问题收集，节约了很多人力成本。其中，DFSDT是一个很关键的用时间换取正确率的方式，使得大模型解决了许多原本不能解决的问题。

然而，数据集构建的方式（ChatGPT根据API集合生成问题）导致数据集中的问题从直觉上来看和人类的问题有一些区别，数据上分布的差异还需要更多研究。这些问题有不少都是“我需要A，另外，我需要B”的形式，这使得解决问题的方式相比于树状的“选择了A，然后根据A的结果选择了B”，更加像是平行的“选择了A，同时选择了B”。因此，我对基于此方法训练的模型的综合问题解决能力还是存在疑问，比如“和北京气温一样的城市有哪些？”，这样的问题需要越来越深入的探索（先查询北京气温，然后在查询同样气温的城市），而不是平行的查询时，ToolLLaMA会不会不那么容易解决呢？
