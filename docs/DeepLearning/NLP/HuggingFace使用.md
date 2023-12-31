---
title: HuggingFace使用
---
## HuggingFace概述
HuggingFace是一个开源社区，提供了先进的NLP模型（Models - Hugging Face）、数据集（Datasets - Hugging Face）以及其他便利的工具

HuggingFace主干库：
- Transformer模型库
- Datasets数据集库：下载/预处理
- Tokenizer分词库：将sequence转变为一个id序列

主要的模型：
- 自回归：GPT2、Transformer-XL、XLNet
- 自编码：BERT、ALBERT、RoBERTa、ELECTRA
- Seq2Seq：BART、Pegasus、T5

### 安装
```python
pip install transformers --trusted-host pypi.tuna.tsinghua.edu.cn
```
安装完成后，输入：
```python
import transformers
transformers.__version__   
```
### 本文目录

1. 使用字典和分词工具：`tokenizer`
2. 数据集的操作：`datasets`
3. 使用评价函数：`metrics`
4. 使用管道函数：`pipeline`（提供了一些不需要训练就可以执行一些NLP任务的模型）
5. 实战任务：中文情感二分类问题

## 使用字典和分词工具：tokenizer
### 加载tokenizer，准备语料
在加载tokenizer的时候要传一个name，这个name与模型的name相一致，所以一个模型对应一个tokenizer
```python
from transformers import BertTokenizer

#加载预训练字典和分词方法
tokenizer = BertTokenizer.from_pretrained(
    pretrained_model_name_or_path='bert-base-chinese',  # 可选，huggingface 中的预训练模型名称或路径，默认为 bert-base-chinese
    cache_dir=None,  # 将数据保存到的本地位置，使用cache_dir 可以指定文件下载位置
    force_download=False,   
)

sents = [
    '选择珠江花园的原因就是方便。',
    '笔记本的键盘确实爽。',
    '房间太小。其他的都一般。',
    '今天才知道这书还有第6卷,真有点郁闷.',
    '机器背面似乎被撕了张什么标签，残胶还在。',
]

tokenizer, sents
```
运行后，会自动下载 vocab.txt、tokenizer_config.json 和 config.json 三个文件

### 编码
1. **简单的编码函数 tokenizer.encode()**
```python
#编码两个句子
out = tokenizer.encode(
	# 一次编码两个句子，若没有text_pair这个参数，就一次编码一个句子
    text=sents[0],
    text_pair=sents[1], 

    #当句子长度大于max_length时,截断
    truncation=True,

    #一律补pad到max_length长度
    padding='max_length',   # 少于max_length时就padding
    add_special_tokens=True,
    max_length=30,
    return_tensors=None,  # None表示不指定数据类型，默认返回list
)

print(out)

tokenizer.decode(out)
```
bert-base-chinese是以一个字作为一个词，开头是特殊符号 [CLS]，两个句子中间用 [SEP] 分隔，句子末尾也是 [SEP]，最后用 [PAD] 将句子填充到 max_length 长度

2. **增强的编码函数 tokenizer.encode_plus()**
```python
# 增强的编码函数
out = tokenizer.encode_plus(
    text=sents[0],
    text_pair=sents[1],

    # 当句子长度大于max_length时,截断
    truncation=True,

    # 一律补零到max_length长度
    padding='max_length',
    max_length=30,
    add_special_tokens=True,

    # 可取值tensorflow,pytorch,numpy,默认值None为返回list
    return_tensors=None,

    # 返回token_type_ids
    return_token_type_ids=True,

    # 返回attention_mask
    return_attention_mask=True,

    # 返回special_tokens_mask 特殊符号标识
    return_special_tokens_mask=True,

    # 返回offset_mapping 标识每个词的起止位置,这个参数只能BertTokenizerFast使用
    # return_offsets_mapping=True,

    # 返回length 标识长度
    return_length=True,
)

print(out)   # 字典
```

```python
for k, v in out.items():
    print(k, ':', v)

tokenizer.decode(out['input_ids'])
```
![[Pasted image 20231125172852.png]]
- **input_ids** 就是编码后的词，即将句子里的一个一个词变为一个一个数字
- **token_type_ids** 第一个句子和特殊符号的位置是0，第二个句子的位置是1（含第二个句子末尾的 [SEP]）
- **special_tokens_mask** 特殊符号的位置是1，其他位置是0
- **attention_mask** pad的位置是0，其他位置是1
- **length** 返回句子长度
---
上述方式是一次编码一个或者一对句子，但是**实际操作中需要批量编码句子**


3. **批量编码单个句子 tokenizer.batch_encode_plus()**
这里编码的是一个一个的句子，而不是一对一对的句子
```python
#批量编码一个一个的句子
out = tokenizer.batch_encode_plus(
	# 批量编码，一次编码了两个句子(与增强的编码函数相比，就此处不同)
    batch_text_or_text_pairs=[sents[0], sents[1]],  

    #当句子长度大于max_length时,截断
    truncation=True,

    #一律补零到max_length长度
    padding='max_length',
    max_length=15,
    add_special_tokens=True,
    
    #可取值tf,pt,np,默认为返回list
    return_tensors=None,

    #返回token_type_ids
    return_token_type_ids=True,

    #返回attention_mask
    return_attention_mask=True,

    #返回special_tokens_mask 特殊符号标识
    return_special_tokens_mask=True,

    #返回offset_mapping 标识每个词的起止位置,这个参数只能BertTokenizerFast使用
    #return_offsets_mapping=True,

    #返回length 标识长度
    return_length=True,
)

# 字典
print(out)   
```

```python
for k, v in out.items():
    print(k, ':', v)

tokenizer.decode(out['input_ids'][0]), tokenizer.decode(out['input_ids'][1])
```
![1700904741797.png](http://pic.moguw.top/i/2023/11/25/6561bf2716cb8.png)

4. **批量编码成对句子 tokenizer.batch_encode_plus()**
传入的list中是一个一个的tuple，tuple中是一对句子
```python
#批量编码成对的句子
out = tokenizer.batch_encode_plus(
	# tuple
    batch_text_or_text_pairs=[(sents[0], sents[1]), (sents[2], sents[3])],   

    #当句子长度大于max_length时,截断
    truncation=True,

    #一律补零到max_length长度
    padding='max_length',
    max_length=30,
    add_special_tokens=True,

    #可取值tf,pt,np,默认为返回list
    return_tensors=None,

    #返回token_type_ids
    return_token_type_ids=True,

    #返回attention_mask
    return_attention_mask=True,

    #返回special_tokens_mask 特殊符号标识
    return_special_tokens_mask=True,

    #返回offset_mapping 标识每个词的起止位置,这个参数只能BertTokenizerFast使用
    #return_offsets_mapping=True,

    #返回length 标识长度
    return_length=True,
)

# 字典
print(out)   
```

```python
for k, v in out.items():
    print(k, ':', v)

tokenizer.decode(out['input_ids'][0])
```
![1700904842094.png](http://pic.moguw.top/i/2023/11/25/6561bf8bb61c0.png)

### 字典操作并编码新词
1. **操作 tokenizer 中的字典：**
```python
# 获取字典
zidian = tokenizer.get_vocab()
type(zidian), len(zidian), '月光' in zidian,   # (dict, 21128, False)
```
因为 `bert-base-chinese` 是以一个字为一个词，所以“月光”这个词（而不是单个字）是不存在的，返回 `False`

```python
# 添加新词
tokenizer.add_tokens(new_tokens=['月光', '希望'])

# 添加新符号
tokenizer.add_special_tokens({'eos_token': '[EOS]'})   # End Of Sentence
zidian = tokenizer.get_vocab()
# (dict, 21131, 21128, 21130)
type(zidian), len(zidian), zidian['月光'], zidian['[EOS]']   
```

2. **编码新词：**
```python
# 编码新添加的词
out = tokenizer.encode(
    text='月光的新希望[EOS]',
    text_pair=None,

    # 当句子长度大于max_length时,截断
    truncation=True,

    # 一律补pad到max_length长度
    padding='max_length',
    add_special_tokens=True,
    max_length=8,
    
    return_tensors=None,
)

print(out)
tokenizer.decode(out)
```
![1700905084504.png](http://pic.moguw.top/i/2023/11/25/6561c07ddba5a.png)

## 数据集的操作：datasets
### 加载数据
**方法1：远程下载到本地，并保存到磁盘**
```python
from datasets import load_dataset

# 加载数据
dataset = load_dataset(path='lansinuote/ChnSentiCorp')
dataset
```

> 注意运行这段代码时，要关闭代理服务器，否则无法下载数据集

```python
# 保存数据集到磁盘
dataset.save_to_disk(dataset_dict_path='./data/ChnSentiCorp')
```

**方法2：直接从本地磁盘读取**

```python
# 从磁盘加载数据
from datasets import load_from_disk

dataset = load_from_disk('./data/ChnSentiCorp')
dataset
```

ChnSentiCorp是一个中文情感分析数据集，包含酒店、笔记本电脑和书籍的网购评论，长下面这样：
![1700905191616.png](http://pic.moguw.top/i/2023/11/25/6561c0e929aa7.png)
训练集有9600个句子，每个句子有两个特征：text 和 label
```python
# 取出训练集
dataset = dataset['train']
# 查看一个数据
dataset[0]
```

### 操作数据
- **排序sort** 和 **打乱shuffle**
- **选择select** 和 **过滤filter**
- **切分**训练集和测试集**train_test_split** 和 **分桶shard**（把数据切分到n个桶中，均匀分配）
- 列操作（包括**列重命名rename_column** 和 **列移除remove_columns**）和 **转换类型set_format**
- **map函数**：对数据集中的每一条数据都做 函数f 操作
- **保存到磁盘**（可以是 csv格式 或 json格式）**.to_csv 或 .to_json** 和 **加载到内存load_dataset**

## 使用评价函数：metrics
### 查看可用的评价指标
```python
from datasets import list_metrics

# 列出评价指标
metrics_list = list_metrics()
len(metrics_list), metrics_list
```

### 使用某个评价指标，并查看其说明文档
```python
from datasets import load_metric

#加载一个评价指标
# MRPC(The Microsoft Research Paraphrase Corpus，微软研究院释义语料库)
metric = load_metric('glue', 'mrpc')   
print(metric.inputs_description)
```

### 计算评价指标
```python
# 计算一个评价指标
predictions = [0, 1, 0]
references = [0, 1, 1]

final_score = metric.compute(predictions=predictions, references=references)
final_score
```

## 使用管道函数：pipeline

pipeline提供了一些不需要训练就可以执行一些nlp任务的模型，实用价值不高

```python
from transformers import pipeline
```

- **情感分类**：判断是 positive 还是 negative  
    classifier = pipeline("sentiment-analysis")  
    
- **阅读理解**：输入一段文本，再问一个问题，答案一定在文本中  
    question_answerer = pipeline("question-answering")  
    
- **完形填空**：把一段文本中的某个词变成mask，预测这个mask对应的词  
    unmasker = pipeline("fill-mask")  
    
- **文本生成**：输入一个开头，让模型接着往后书写  
    text_generator = pipeline("text-generation")  
    
- **命名实体识别**：识别出这段文本当中的一些城市名/公司名/人名等等  
    ner_pipe = pipeline("ner")  
    
- **文本摘要**：给一段长文本，输出一段短文本，两个文本表达的核心思想相同  
    summarizer = pipeline("summarization")  
    
- **翻译**：如英文翻译德语  
    translator = pipeline("translation_en_to_de")
