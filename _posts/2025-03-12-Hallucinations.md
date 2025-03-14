---
layout: post
title: 大模型的“幻觉”：是创意的火花还是认知的陷阱？
date: 2025-03-12 21:30:13
description: 探讨大语言模型（LLM）中的“幻觉”现象——它既能激发无限的创造力，也可能带来误导性的风险。本文深入分析幻觉的成因、影响以及如何在技术发展中找到平衡。
tags: LLM
categories: Tech

toc:
  sidebar: left
---


> 幻觉通常是指模型生成不真实、虚构、不一致或无意义的内容。但这个现象也可以用另一个名称来理解 —— "温度"（temperature）。

## **幻觉的类型**

1. **上下文幻觉** (In-context Hallucination)
    
    指模型生成的输出与当前对话上下文（用户提供的输入或历史信息）存在直接矛盾或不一致。
    
2. **外部幻觉** (Extrinsic Hallucination)
    
    指模型生成的内容违背预训练数据中的世界知识或客观事实
    

为了避免幻觉，大模型需要做到以下两点：

1. 实事求是，不编造不存在的信息。
2. 在适当的时候承认自己不知道答案。

## Why cause **Hallucinations**

大模型的基本原理是基于上下文的预测，也就是在给定前文的情况下通过概率模型预测下文。预测的准确性取决于大模型在训练时如何处理和消化这些信息。大模型不是数据库，它压缩消化的是知识体系，包括常识或百科知识，但天然排除缺乏信息冗余度的长尾事实及细节。
从统计上来看，“长尾”事实与噪音无异。只有冗余信息和常识性内容最终嵌入模型参数中。当模型需要预测下一个词时，如果此处需要模型没有“记住”的长尾事实，它只能“编造”细节继续生成，这就是幻觉的来源。

### **预训练数据问题 Pre-training Data Issues**

大模型通过网络内容进行训练，而这些网络内容本身往往包含错误信息。人类在传播信息时会带有偏差，这些错误信息会被纳入训练集，进而影响模型输出

### 微调阶段引入新知识 Fine-tuning New Knowledge

在微调阶段引入新知识是不可避免的。但是由于Fine-tune的计算量相对较小，所以会出现以下问题：

1. 模型学习新知识示例是慢于具有与模型先前知识一致的知识的其他示例
2. 一旦最终学习了具有新知识的示例，它们就会增加模型产生幻觉的倾向

### **模型自身特点**

大语言模型的灵感来源于我们的脑神经。那么请问，人脑一般情况下最记不住什么？答案是具体实体（如人名、地名、书名、标题、时间、地点等）。所以大模型的本质是试图从大量数据中找出各种规律，而不是记录所有细节。
但人类记不住事实的时候，通常会说自己忘了或者添加“好像、可能”等不确定的语气（习惯性说谎者除外）。而现在的语言大模型与此不同，它“记不住”事实的时候，就会编造读起来似乎最顺畅的细节。

## 如何对抗幻觉

<aside>

这些方法的核心思想是：

1. **让模型自己验证和修正**，减少错误。
2. **引入外部知识**，确保内容的准确性。
3. **调整模型的行为**，让它更关注事实。
</aside>

### 1. RAG （知识检索增强）

RAG是一种结合检索（Retrieval）和生成（Generation）的方法，旨在通过引入外部知识来增强大语言模型的准确性和可靠性。其核心思想是：在生成答案之前，先从外部知识库中检索与问题相关的文档，然后将这些文档作为额外的上下文信息提供给模型，帮助其生成更准确、更可靠的回答。

**RAG的工作原理**

1. **检索阶段（Retrieval Phase）**
    - **问题理解**：当用户提出一个问题时，模型首先需要理解问题的核心内容和关键词。
    - **文档检索**：模型会从一个预先构建的外部知识库（如维基百科、新闻文章、专业数据库等）中检索与问题最相关的文档片段。检索过程通常使用向量相似度计算（如BERT Embeddings）来找到与问题最匹配的文档。
    - **文档筛选**：检索到的文档可能包含大量信息，模型会进一步筛选出最相关、最有用的部分，作为后续生成的上下文。

2. **生成阶段（Generation Phase）**
    - **上下文融合**：将检索到的文档片段与用户的问题结合，形成一个增强的上下文。这个上下文包含了问题的背景信息和相关知识，帮助模型更好地理解问题。
    - **答案生成**：模型基于增强的上下文生成答案。由于引入了外部知识，生成的答案更有可能是准确的、基于事实的，而不是模型自身的“幻觉”。
    - **优化与调整**：生成的答案可能会经过进一步的优化和调整，以确保其连贯性和准确性。

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Hall_RAG.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### 2. 行动链 - 验证链

验证链（CoVe）包含核心四个步骤：

1. **Baseline Response**
    
    模型生成一个初始结果，称之为 “Baseline”
    
2. **Plan Verification**
    
    基于Baseline，模型设计非模版化的验证问题用于事实核查。比如，如果模型说“太阳从西边升起”，它可能会设计一个问题：“太阳是从哪个方向升起的？”
    
3. **Execute Verification**
    
    模型独立回答这些问题。这里有几种不同的方式：
    
    - **联合验证**：验证问题和回答一起进行，但这样可能会导致模型重复之前的错误。
    - **两步验证**：把验证问题的设计和回答分开，避免初始回答影响验证结果。
    - **分解验证**：每个验证问题单独回答，避免长回答中的错误相互影响。
    - **分解+修订**：在分解验证的基础上，增加一个“交叉检查”步骤，确保回答和验证结果一致。

4. **Final output**
    
    如果发现不一致，模型会修正回答，生成最终的正确回答。
    

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Hall_CoVe.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### **3. Fine-tuning for Factuality（针对事实性的微调）**

通过微调模型，让它更擅长生成准确的内容。具体方法：

- **主题前缀（TopicPrefix）**
    
    在训练时，给每个句子加上主题（如维基百科标题），帮助模型更好地理解事实。
    
- **句子补全损失（Sentence Completion Loss）**
    
    训练时更关注句子的后半部分，因为后半部分通常包含更多事实性信息。
    

### **4. Sampling Methods**

在生成文本时，模型会从多个可能的词中选择。不同的采样方法会影响生成内容的准确性：

- **核采样（Nucleus Sampling）**
    
    这种方法会增加随机性，虽然能生成更多样化的内容，但可能导致更多错误。
    
- **事实核采样（Factual-Nucleus Sampling）**
    
    改进版的核采样，动态调整随机性，减少句子后半部分的错误。
    

**通俗理解**：就像写句子时，前半部分尽量准确，后半部分稍微灵活一些，但不要瞎编。

## **幻觉的双面性：创造与谎言**

大模型的幻觉现象是一个复杂而多面的存在，不能简单地用“好”或“坏”来一概而论。在某些场景下，幻觉可能是一种误导性的“谎言”，而在另一些场景中，它却可能成为激发创造力和想象力的源泉。这种双面性使得我们对大模型的使用必须更加审慎和有选择性。

**幻觉作为“谎言”：误导与风险**

在事实性问题、新闻信息传播以及专业领域中，幻觉往往是一种需要极力避免的现象。例如，当模型在回答关于科学原理、历史事件或法律条款的问题时，生成与事实不符的内容，这不仅会误导用户，还可能引发严重的后果。在新闻报道中，幻觉可能导致虚假信息的传播，从而破坏社会信任；在医疗、法律等专业领域，错误的建议甚至可能危及生命或导致法律纠纷。

随着生成技术的不断发展，模型生成内容的成本越来越低，但人工检查和验证的成本却依然高昂。这意味着在关键领域，我们仍然需要依赖专业人士的校正和审核。因此，我们不能对模型的输出过度信任，尤其是在涉及重大决策和专业知识的场景中。

**幻觉作为“创造”：激发想象力与创造力**

然而，幻觉并非总是负面的。在文学创作、艺术设计和娱乐游戏等领域，幻觉可能成为一种独特的工具，激发人类的想象力和创造力。大模型能够生成虚构的故事、独特的艺术风格或新颖的游戏情节，这些内容虽然并非基于现实，但却能为创作者提供灵感，推动艺术和文化的创新。

这种“创造性幻觉”与人类智慧中的一种关键能力——虚构能力——有着相似之处。正如尤瓦尔·赫拉利在《人类简史》中所指出的，人类文明的发展正是依赖于“讲故事”的能力。我们通过虚构神话、宗教、理想和情怀，组织起庞大的群体合作，从而战胜其他动物，成为地球的主宰。从这个意义上说，幻觉也可以被视为一种人类创造力的延伸。

**专家角色的演变与挑战**

尽管大模型在某些领域降低了知识获取的门槛，但这也带来了新的挑战。它实际上**提高了真正成为专家的门槛，加剧了普通人之间的竞争**，同时使得真正的专业知识竞争相对减弱。这种现象可能导致那些具有表面知识而缺乏深入理解的人在某些领域获得更多影响力和话语权。

许多人因此认为**专家的作用在大模型出现后变得不再重要**，但这种观点潜藏着巨大的危险。随着技术的发展，专家的权威虽然在逐步下降，但他们在验证事实、提供深度分析和指导决策方面仍然不可或缺。专家的存在不仅是知识的守护者，更是社会信任和进步的重要保障。

## Related Article

1. [**Extrinsic Hallucinations in LLMs**](https://lilianweng.github.io/posts/2024-07-07-hallucination/)
2. [**万字解构“幻觉陷阱”：大模型犯的错，会摧毁互联网吗？**](https://mp.weixin.qq.com/s/zsK0XmDIZOsmBXLHnYJZrQ)
