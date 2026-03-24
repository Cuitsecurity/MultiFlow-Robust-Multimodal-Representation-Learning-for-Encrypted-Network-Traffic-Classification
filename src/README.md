# MultiFlow 源代码说明文档

本目录包含了MultiFlow项目的核心源代码，用于加密网络流量的多模态表示学习和分类。项目采用模块化设计，分为数据处理、预处理和模型三个主要部分。

## 目录结构

```
src/
├── data/                          # 数据处理模块
│   ├── build_vocab.py            # 构建属性值词表和嵌入矩阵
│   ├── extract_stats_features.py  # 提取网络流统计特征
│   └── extract_stats_payload.py   # 提取网络包Payload特征
├── preprocessing/                 # 数据预处理模块
│   └── split_dataset.py           # 网络流的拆分和处理
└── models/                        # 模型训练模块
    └── train.py                   # 模型训练脚本
```

---

## 模块详解

### 1. 数据处理模块 (`data/`)

#### `build_vocab.py` - 属性值词表与嵌入矩阵构建

**功能:**
- 从CSV格式的网络流属性数据构建Word2Vec词表
- 生成属性值的嵌入向量矩阵
- 输出词典和嵌入数据供后续使用

**主要类:**

```python
class AttributeEmbedding
```

**核心方法:**
- `__init__()`: 初始化，加载语料、训练Word2Vec模型、生成嵌入矩阵
- `load_corpus()`: 从CSV文件加载语料，每个序列最多保留40个包的属性值
- `get_vector()`: 查询指定token的嵌入向量

**输入:**
- CSV文件：包含网络包属性值的序列（逗号分隔）

**输出:**
- `{prefix}_dict.pkl`: Word2Vec词典（token→index的映射）
- `{prefix}_embedding.npy`: 嵌入矩阵（numpy数组格式）
- `{prefix}_word2vec.model`: Word2Vec模型文件

**配置参数:**
- `embedding_dim`: 嵌入维度（默认300）
- `window`: Word2Vec上下文窗口大小（默认5）
- `max_packets`: 每个流最多提取的包数（默认40）

---

#### `extract_stats_features.py` - 网络流统计特征提取

**功能:**
- 从PCAP文件解析网络流
- 提取网络流的统计特征：包长度、包方向、包间时间间隔(IAT)
- 批量处理多个PCAP文件

**主要函数:**

| 函数名 | 功能描述 |
|------|--------|
| `get_flow_features(pcap_path)` | 从PCAP文件提取单个流的特征 |
| `append_csv(path, rows)` | 批量追加行数据到CSV文件 |
| `mark_processed()` | 标记已处理的PCAP文件 |
| `get_processed_pcapfile()` | 读取已处理文件的日志 |

**提取的特征:**
- **Lengths**: 每个包的字节长度（绝对值）
- **Directions**: 包传输方向（1表示上行，-1表示下行）
- **IATs**: 包间到达时间间隔（毫秒×100，便于离散化）

**输入:**
- PCAP文件（网络包捕获文件）

**输出:**
- CSV格式：包含lengths、directions、iats的流特征

**配置参数:**
- `max_packets`: 每个流最多提取的包数（默认40）
- `BATCH_SIZE`: 文件批处理大小（默认100）
- `NUM_WORKERS`: 多进程工作线程数

---

#### `extract_stats_payload.py` - 网络包Payload特征提取

**功能:**
- 从PCAP文件提取网络包的实际Payload数据
- 使用Bigram特征表示包内容
- 同时提取统计特征（长度、方向、IAT）

**主要函数:**

| 函数名 | 功能描述 |
|------|--------|
| `get_flow_features(pcap_path)` | 提取流的完整特征（统计+Payload） |
| `bigram_generation()` | 生成包数据的Bigram特征 |
| `mark_processed()` | 标记已处理的文件 |
| `cut()` | 分块处理数据 |

**提取的特征:**
- **Lengths**: 包长度
- **Directions**: 传输方向
- **IATs**: 包间时间间隔
- **Payloads**: 包内容的Bigram特征序列

**输入:**
- PCAP文件

**输出:**
- CSV格式：label, lengths, directions, iats, payloads

**配置参数:**
- `max_packets`: 每个流最多包数（默认40）
- `payload_len`: 单个包payload字符串长度（默认64字符）
- `payload_pac`: 每个流提取的包个数（默认10）
- `MAX_FLOWS_PER_LABEL`: 每个标签最多流数（默认500）
- `NUM_WORKERS`: 多线程数（默认64）

---

### 2. 预处理模块 (`preprocessing/`)

#### `split_dataset.py` - 网络流的拆分和处理

**功能:**
- 从PCAP文件中解析和拆分网络TCP连接
- 实现流级别的分割，处理多连接场景
- 生成规范化的五元组标识

**主要函数:**

| 函数名 | 功能描述 |
|------|--------|
| `get_normalized_five_tuple(pkt)` | 生成归一化的五元组(src_ip, src_port, dst_ip, dst_port, protocol) |
| `split_multiple_connections_in_session()` | 拆分同五元组下的多个独立TCP连接 |
| `is_timestamp_interval_gt()` | 检查时间戳间隔是否超过阈值 |
| `create_folder_if_not_exists()` | 创建输出目录 |
| `split_complete_flows()` | 完整的流拆分主流程 |

**关键特性:**

1. **流的规范化**: 
   - 双向流量（A→B和B→A）归为同一个流
   - 按字典序排序IP+Port的组合

2. **TCP连接拆分**:
   - 按TCP三次握手识别连接开始（纯SYN包）
   - 按FIN/RST包识别连接结束
   - 过滤掉包数少于3个的无效连接

3. **时间间隔分割**:
   - 根据包间时间戳差值（`time_step`参数）进行流的进一步拆分
   - 避免将不同时间段的数据混在一个流中

**输入:**
- PCAP文件

**输出:**
- 拆分后的PCAP文件，存放在指定输出目录
- 文件名格式：`{prefix}-{src_ip}_{src_port}_{dst_ip}_{dst_port}_{protocol}_{flow_count}.pcap`

**配置参数:**
- `time_step`: 时间戳间隔阈值（默认15秒）
- `output_dir`: 输出目录路径

---

### 3. 模型训练模块 (`models/`)

#### `train.py` - 模型训练脚本

**功能:**
- 训练多模态网络流量分类模型
- 支持多种输入特征（统计特征、Payload特征）
- 实现模型优化和性能评估

**预期功能:**
- 使用前面生成的词表和嵌入矩阵
- 融合多个特征模态（统计特征+Payload特征）
- 训练分类器区分不同的网络应用/协议

---

## 数据流水线

```
PCAP文件 (原始网络包)
    ↓
[preprocessing/split_dataset.py] ← 流拆分和规范化
    ↓
拆分后的PCAP文件
    ↓
[data/extract_stats_features.py] ← 提取统计特征
[data/extract_stats_payload.py]  ← 提取Payload特征
    ↓
CSV特征文件
    ↓
[data/build_vocab.py] ← 构建词表和嵌入
    ↓
词典文件(.pkl) + 嵌入矩阵(.npy)
    ↓
[models/train.py] ← 训练分类模型
    ↓
训练完成的模型
```

---

## 依赖库

```
pandas
numpy
gensim       # Word2Vec词表构建
scapy         # 网络包解析
flowcontainer # 网络流提取
scikit-learn  # 可能用于模型训练
```

---

## 使用说明

### 步骤1: PCAP流拆分
```bash
python src/preprocessing/split_dataset.py
```
输入：原始PCAP文件
输出：拆分后的流级PCAP文件

### 步骤2: 提取网络特征
```bash
# 方式A：仅提取统计特征
python src/data/extract_stats_features.py

# 方式B：提取统计特征+Payload特征
python src/data/extract_stats_payload.py
```
输入：拆分后的PCAP文件
输出：CSV格式特征文件

### 步骤3: 构建词表和嵌入
```bash
python src/data/build_vocab.py
```
输入：特征CSV文件
输出：词典.pkl、嵌入矩阵.npy、Word2Vec模型

### 步骤4: 训练模型
```bash
python src/models/train.py
```
输入：特征数据、词表、嵌入矩阵
输出：训练完毕的分类模型

---

## 技术特点

1. **多模态特征融合**:
   - 统计特征：包长度、方向、时间间隔
   - 内容特征：包Payload的Bigram表示

2. **鲁棒的流拆分**:
   - 处理双向流
   - 正确拆分多连接场景
   - 时间间隔自适应分割

3. **高效的特征表示**:
   - 使用Word2Vec嵌入降低维度
   - 支持批量处理和多进程并行

4. **加密流量友好**:
   - 不依赖Payload解密
   - 使用加密前的网络元数据
   - 支持应用级别的分类

---

## 注意事项

- 修改输入/输出路径时，需要在各脚本中配置对应的参数
- 多进程并行处理时注意系统资源限制，适当调整`NUM_WORKERS`
- PCAP文件需要至少5个包才能被进一步处理
- IAT特征乘以100倍便于离散化处理，使用时注意单位转换


