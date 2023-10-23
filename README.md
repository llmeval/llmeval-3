# LLMEval-3

<img src=".\pic\llmeval-logo.png" alt="llmeval-logo" style="zoom:50%;" />

## 目录说明

```
├─code 评测代码
├─data
│  ├─data_process 数据处理过程的代码文件和数据文件
│  ├─logs 处理日志
│  └─raw_data 原始的尚未变形的题目
├─pic 图片
└─release 可以发布的1000题
```



## 题目内容与形式

LLMEval-3聚焦于专业知识能力评测，涵盖哲学、经济学、法学、教育学、文学、历史学、理学、工学、农学、医学、军事学、管理学、艺术学等教育部划定的13个学科门类、50余个二级学科，共计约20W道标准生成式问答题目（后续我们将继续收集题目将总题库扩充至100W）。

<img src=".\pic\subjects.PNG" alt="subjects" style="zoom: 80%;" />

题目来源主要包括**大学本科课后作业、大学本科期中期末考试、研究生入学考试**等。为了尽可能的防止参与评测的大模型在预训练阶段引入大比例原始评测数据，LLMEval-3评测题目来源尽可能为非互联网公开渠道，数据格式为PDF和Word文件，经过一定的OCR识别与数据清洗之后，将题目进行格式化处理。针对于不同的题型，提供给待测试模型标准接口，实现全流程自动化。

与其他知识评测所采用的选择题模式不同，LLMEval-3中所有问题将统一处理为**生成式知识问答**形式，并尽可能包含多种题型，包括简答，计算、判断、辨析、写作等。相较于具有标准格式的选择题，LLMEval-3所采用的生成式知识问答，能够更好地反映用户实际需求以及模型语言能力。



## 评测流程与指标

防止作弊是LLMEval-3考虑的重要因素。现有公开评测基准存在测试题库泄露的问题，因此可能出现“刷榜”、“刷分”等不公平现象，在LLMEval-3中，每个参与评测的系统需要完成从总题库中随机抽样的1000题，**针对同一机构的模型，确保每次评测题目不重复**。评测过程将采用在线方式，一轮评测中题目的发送串行进行，即下一题的发送将会视上一道题目的回答情况而定，避免恶意爬取行为。 

本轮评测采用GPT-4自动评测方法打分，每道题得分的范围为0-3分。评分聚焦于回答的核心正确性和解释正确性，其中核心正确性作为衡量分数的主要指标。采用评测prompt如下所示：

```text
Please evaluate the following response from the LLM regarding a discipline-specific question based  on the following criteria. You must score it on a scale of 0, 1, 2 or 3 stars:

Overall Rating:
0 star indicates wrong answer with a wrong explanation
1 stars indicate wrong answer but a partially reasonable explanation
2 stars indicate a correct answer with a partially reasonable explanation
3 stars indicate a correct answer with a reasonable explanation

User: {question}

LLM:{answer_from_llm}

The correct answer to user’s question is: {correct_answer}

You must provide your feedback in the following format:
{"Overall Rating":numbers of its stars(int)}
```

为了规避由随机抽样1000题引入的系统偏差，LLMEval-3使用**相对分数**和**绝对分数**两个指标。

模型的相对分数定义为其绝对分数相比于GPT-3.5-turbo以及GPT-4在相同题目上取得的绝对分数的分位并映射到 $[0, 100]$ 区间，使用 $$\rho_{\text{GPT-3.5}}^{model}$$ 和 $$\rho_{\text{GPT-4}}^{model}$$ 表示，具体计算公式如下：
$$
\rho^{model}_{\text{GPT-3.5}}=\frac{S_{model}}{S_\text{GPT-3.5}} \times 100
$$

$$
\rho^{model}_{\text{GPT-4}}=\frac{S_{model}}{S_\text{GPT-4}} \times 100
$$



模型的绝对分数是指模型在 $$N=1000$$ 道题目的单题得分 $$s_{i}$$ (单题总分 $$s_{max}=3$$  )之和并映射到 $$[0, 100]$$ 区间，使用 $$S_{model}$$ 表示，具体计算公式如下：
$$
S_{model}=\frac{\sum_{i=1}^Ns_i}{N\times s_{max}} \times100
$$

## 评测结果

LLMEVAL-3项目已经对多个开源模型进行评测，结果如下：

| 模型名称                   | 访问方式 | 发布机构                          | 评测日期  | 相对分数-GPT4  | 相对分数-GPT3.5 | 绝对分数 |
| -------------------------- | -------- | --------------------------------- | --------- | -------------- | --------------- | -------- |
| GPT-4                      | API      | OpenAI                            | 2023.9.29 | __**100.00**__ | __**116.81**__  | 74.60    |
| Baichuan2-13B-Chat         | 权重     | Baichuan                          | 2023.9.29 | __**92.94**__  | __**108.56**__  | 69.33    |
| Qwen-14B-Chat              | 权重     | Alibaba Cloud                     | 2023.10.1 | __**86.10**__  | __**100.57**__  | 64.23    |
| GPT-3.5-turbo              | API      | OpenAI                            | 2023.9.29 | **85.61**      | **100.00**      | 63.87    |
| ChatGLM2-6B                | 权重     | Tsinghua&Zhipu.AI                 | 2023.9.29 | **75.56**      | **88.26**       | 56.37    |
| ziya_v1.1-13b              | 权重     | IDEA研究院                        | 2023.9.29 | **70.33**      | **82.15**       | 52.47    |
| BELLE-Llama2-13B-chat-0.4M | 权重     | LianjiaTech                       | 2023.10.1 | **68.86**      | **80.43**       | 51.37    |
| Linly-Chinese-LLaMA2-13B   | 权重     | 大数据系统计算技术国家工程实验室  | 2023.10.3 | **67.69**      | **79.07**       | 50.50    |
| InternLM-Chat-7B           | 权重     | Shanghai AI Laboratory&Sense Time | 2023.9.29 | **61.08**      | **71.35**       | 45.57    |
| Llama-2-7b-chat-hf         | 权重     | Meta                              | 2023.9.29 | **51.30**      | **59.92**       | 38.27    |

各学科分数如下：

| 模型名称                   | 绝对分数 | 工学 | 经济学 | 教育学 | 法学 | 文学 | 管理学 | 理学 | 历史学 | 医学 | 军事学 |
| -------------------------- | -------- | ---- | ------ | ------ | ---- | ---- | ------ | ---- | ------ | ---- | ------ |
| GPT-4                      | 74.60    | 7.23 | 7.80   | 7.73   | 8.40 | 6.73 | 7.67   | 7.73 | 7.07   | 6.20 | 8.03   |
| Baichuan2-13B-Chat         | 69.33    | 6.00 | 7.53   | 8.63   | 8.13 | 6.23 | 6.33   | 5.63 | 8.20   | 5.43 | 7.20   |
| Qwen-14B-Chat              | 64.23    | 5.77 | 7.07   | 7.07   | 7.37 | 5.70 | 6.20   | 5.93 | 6.97   | 5.40 | 6.77   |
| GPT-3.5-turbo              | 63.87    | 6.27 | 6.87   | 7.23   | 7.40 | 5.40 | 6.30   | 6.37 | 6.00   | 5.17 | 6.87   |
| ChatGLM2-6B                | 56.37    | 4.03 | 6.33   | 7.00   | 7.57 | 4.77 | 5.93   | 4.23 | 5.87   | 5.07 | 5.57   |
| ziya_v1.1-13b              | 52.47    | 4.67 | 5.77   | 6.07   | 6.53 | 4.53 | 5.33   | 3.70 | 5.00   | 4.63 | 6.23   |
| BELLE-Llama2-13B-chat-0.4M | 51.37    | 4.47 | 5.93   | 6.20   | 6.77 | 4.33 | 4.97   | 4.10 | 5.07   | 3.77 | 5.77   |
| Linly-Chinese-LLaMA2-13B   | 50.50    | 3.87 | 5.80   | 5.83   | 6.57 | 3.93 | 5.37   | 4.07 | 5.43   | 3.93 | 5.70   |
| InternLM-Chat-7B           | 45.57    | 3.83 | 5.13   | 5.27   | 6.57 | 3.90 | 4.83   | 3.10 | 4.87   | 3.67 | 4.40   |
| Llama-2-7b-chat-hf         | 38.27    | 3.33 | 4.77   | 3.77   | 5.03 | 3.07 | 3.77   | 3.93 | 4.00   | 2.40 | 4.20   |



其中，模型在各学科上绝对分数的分布如下：

<img src=".\pic\sub_results.png" alt="sub_results" style="zoom:80%;" />

可以看到，在本科程度的知识问答上，即便是GPT-4也只能达到74.60的绝对分数，仍有非常大的提升空间。国内开源模型在最近3个月有大幅度提升，Baichuan2-13B-Chat以及Qwen-14B-Chat都高于GPT-3.5-turbo的效果。由于LLaMA-2-7B-Chat在中文上的训练数量较少，对于中文的问题回答能力较弱。

为了验证使用相对分数的可靠性，LLMEVAL-3对现阶段所有评测的模型进行了两次完整的评测流程，对两次抽样评测的结果进行了分析，结果如下：

模型两次相对于GPT-3.5-turbo的相对分数

<img src=".\pic\related_scores_3.5.png" alt="related_scores_3.5" style="zoom:80%;" />

模型两次相对于GPT-4的相对分数

<img src=".\pic\related_scores_4.png" alt="related_scores_4" style="zoom:80%;" />

根据实验结果，可以发现以GPT-4作为计算相对分数的基准模型比GPT-3.5-turbo的稳定性更好，结果更为客观。大部分模型相对GPT-4分数的二次相对偏差小于5%，说明了随机抽样的均匀性与多次抽样之间分布的一致性，相对分数更高的模型两次评测的相对分数偏差越小。相较于直接使用绝对分数来说，采用相对分数可以更有效地减少抽样带来的系统偏差。



## 联系我们

本项目已经向公众开放，欢迎参与我们的评测。http://llmeval.com/

机构评测需要进行认证，注册完账户以后，请联系管理员认证并申请评测权限。

如无特殊情况，在评测完成之后，相关结果都会添加在排行榜上。

Email: mingzhang23@m.fudan.edu.cn

Wechat: zanyingluan
