# 总结

## 解决的问题：

大语言模型在执行用户要求的能力上不足



## 主要内容：

设计了ODSL(Office Domain Specific Language)。通过向大模型输入用户要求，输出ODSL程序的方式，执行用户的要求。

## 实验：

### 编辑prompt

提供给模型的信息包括：总体介绍，语法示例，额外规则提示，用户要求，和上下文

#### 总体介绍

总体介绍是指ODSL语言的介绍

#### 是否提供上下文

使用大模型进行判断是否需要上下文；若需要，将上下文以树状形式（JSON）提供

#### 语法示例

使用大模型判断用户要求相关的实体，并提供实体相关的语法

#### 程序示例

向量化存储正则化的问题和答案示例，其中最接近当前问题的会被选取；同时，对于需要上下文的问题，不同上下文条件下的不同问答对会被存储，其中问题和上下文都接近当前情况的会被选取。

#### 额外规则提示

根据用户要求相关的实体提供额外的提示

### 获取ODSL程序的后处理

编译时确认语法错误，减少运行过程中的报错

自动纠错

### ODSL语言设计

#### 统一性

因为是few-shot学习，函数设计的统一性可以提高模型学习效率

#### 紧凑性

为了节约token的数量，以在prompt提供更多信息，函数设计让同一个目标能够以更短的方式达成

#### 减少冗余

减少能够以多种方式达成同一目标的情况，提高模型学习效率

#### 控制语言的表达能力

保证语言不能表达无法撤销的操作，例如关闭文件，使得模型输出在任何时候都可以由用户撤销

## 实验发现：

实验中，用户要求仅包括ODSL能够表达的范围，不包括生成表格和评论处理等

实验中，生成的程序会经过归一化，包括重命名变量，删除一些冗余参数等

实验测量的数据：pass rate

pass：生成的程序，经过归一化，包含了一段子程序和标准答案相同

最好的结果是在提供5个程序示例时达到96%pass rate。

# 评论

本文主要讲了ODSL的设计思路，以及prompt engineering的过程。设计一个对大模型友好的语言，相比于直接让大模型学习Office API，可以给我们更多的控制。比如说，我们可以从语言上保证操作可撤销，从理论上避免了错误的发生。同时，也提高了模型能够正确操作的可能性，和我们对模型打字错误的纠正能力。

实验中，使用的用户要求范围包括创建ppt，添加页面，添加文字，修改已有内容，添加图片，更改格式等，可能是因为设计语言的工程量，并不是PPT的全部功能。而如果我们想要设计一个功能范围更广的语言，在使用同样方法的情况下，pass rate会不会变低，是我会想要探索的问题。