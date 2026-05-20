# 第二章：HuggingFace Transformers 库

## 一、这个库解决什么问题？

模型每天都在出新的，每个模型用法都不一样。HuggingFace Transformers 做的事是当「统一翻译层」——不管底层是什么模型，外面这套接口完全一样。

三个核心特点：
- **易用**：两行代码加载模型、喂数据、拿结果。
- **灵活**：底层就是标准 PyTorch 类，任何 PyTorch 工具都能用。
- **一个文件一个模型**：每个模型的所有代码都在自己的文件里，互不干扰。想看 BERT？打开 bert.py。

## 二、pipeline 背后的三步流水线

```
你的文字 → ① 分词器 → ② 模型本体 → ③ 后处理 → 最终答案
```

**① 分词器**：文字 → 数字。做三件事：切 token、映射成整数 ID、生成 attention_mask。

**② 模型本体**：数字 → 高维向量（hidden states）。输出形状是 [批次大小, 序列长度, 隐藏维度]，比如 [2, 16, 768]。这是「理解状态」，还不是答案。

**③ 模型头 + 后处理**：高维向量 → 最终结果。模型头把向量压缩成 logits（原始分），再经 Softmax 转成概率。同一个 Transformer 身体，接不同的头就能做不同任务：
- 分类头 → 情感分析
- 语言模型头 → 文字生成
- 问答头 → 阅读理解

## 三、模型的两个文件

`model.save_pretrained("文件夹")` 生成两个文件：
- **config.json**：蓝图。记录层数、隐藏维度、注意力头数等架构参数。体积小，几 KB。
- **model.safetensors**：砖瓦。存着训练出来的所有权重参数。体积大，几百 MB 到几 GB。

两者缺一不可。`from_pretrained("文件夹路径")` 自动读取两者，复原完整模型。

**AutoModel**：写 `AutoModel.from_pretrained("bert-base-cased")`，不用手动指定用哪个类，它自动根据 checkpoint 判断。换模型时只改名字，代码不动。

## 四、分词的三种流派

| 流派 | 做法 | 问题 |
|------|------|------|
| 按词切 | 每个词一个 ID | 词表爆炸；dog/dogs 被当成陌生人；生僻词变 [UNK] |
| 按字符切 | 每个字母一个 ID | 词表极小，但序列极长，字母本身没语义 |
| **子词分词** | 高频词整用，生僻词拆碎 | 现代大模型都用这个 |

子词分词示例：`annoyingly` → `annoying` + `ly`；`tokenization` → `token` + `ization`。

三种子词算法：
- **BPE**（GPT-2）：从字符开始，反复合并最高频的相邻符号对。
- **WordPiece**（BERT）：合并时看哪对合并后让训练数据整体概率最大。
- **Unigram**（T5）：反过来，先造大词表，不停删最不重要的子词。

> **铁律**：预训练时怎么切，推理时就必须怎么切。必须用配套的分词器，不能混用。

**`##` 标记**：子词接续符号，表示「我粘在前一个 token 上」。`Transformer` → `transform` + `##er`。decode 时自动拼回完整词。

## 五、批处理：Padding + Attention Mask

批处理时，不同句子长度不同，但张量必须是矩形——这是核心矛盾。

**Padding（填充）**：短句后面补特殊 token（编号通常为 0），补到与最长句子一样长。

**问题**：Padding 会污染注意力计算。注意力机制会关注那些填充的 0，导致同一句话单独推理和放进 batch 推理结果不一致。

**Attention Mask（注意力遮罩）**：和 input_ids 形状相同，**1 = 真文字请关注，0 = padding 请无视**。

```
input_ids:      [200, 200, 200, 200, 200, 200,   0,   0,   0,   0]
attention_mask: [  1,   1,   1,   1,   1,   1,   0,   0,   0,   0]
```

> Padding 和 attention_mask 必须配对使用，这不是可选项。

三种 Padding 模式：
- **`padding="longest"`**：补到本批最长。最省算力，日常首选。
- **`padding="max_length"`**：补到模型上限（如 BERT 的 512）。浪费严重。
- **`padding="max_length", max_length=N`**：补到指定长度 N。

## 六、一行 tokenizer() 干了七件事

```
分词 → 加特殊标记 → 转 ID → padding → 截断 → 生成 attention_mask → 转成张量
```

关于特殊 Token 的坑：直接调用 `tokenizer("I love this")` 会自动加 `[CLS]` 和 `[SEP]`；但手动走 `tokenize()` + `convert_tokens_to_ids()` 不会加。原因是手动走时分词器不知道你要喂给模型，直接调用时它知道。

处理单句和批量的 API 完全一致，不需要切换函数。

### 完整流程

1. 加载分词器
2. 加载模型
3. `tokenizer(文字)` —— 一行干七件事
4. 推理得到 logits
5. Softmax 转概率，查标签表得出结论
