# MultiFlow: Robust Multimodal Representation Learning for Encrypted Network Traffic Classification

MultiFlow is a framework for multimodal representation learning and classification of encrypted network traffic. This repository contains the core source code, including data processing, preprocessing, and model training modules.

---

## Table of Contents

- [Overview](#overview)
- [Directory Structure](#directory-structure)
- [Modules](#modules)
  - [Data Processing (`data/`)](#1-data-processing-data)
  - [Preprocessing (`preprocessing/`)](#2-preprocessing-preprocessing)
  - [Model Training (`models/`)](#3-model-training-models)
- [Data Pipeline](#data-pipeline)
- [Dependencies](#dependencies)
- [Usage](#usage)
- [Key Features](#key-features)
- [Notes](#notes)
- [Citation](#citation)

---

## Overview

MultiFlow is designed to extract multimodal features from encrypted network traffic and train models for traffic classification. It combines statistical flow features and payload content features, using Word2Vec embeddings and Bigram representations to achieve robust classification even on encrypted flows.

---

## Directory Structure
src/
├── data/                          # Data processing module
│   ├── build_vocab.py             # Build attribute-value vocabulary and embedding matrix
│   ├── extract_stats_features.py  # Extract statistical features of network flows
│   └── extract_stats_payload.py   # Extract network packet payload features
├── preprocessing/                 # Data preprocessing module
│   └── split_dataset.py           # Split and process network flows
└── models/                        # Model training module
└── train.py                   # Model training script
---

## Modules

### 1. Data Processing (`data/`)

#### `build_vocab.py` — Build Attribute-Value Vocabulary and Embedding Matrix

**Functionality:**
- Build a Word2Vec vocabulary from CSV network flow attribute data
- Generate embedding vectors for attribute values
- Output vocabulary and embedding data for downstream use

**Main Class:** `AttributeEmbedding`  

**Key Methods:** `__init__()`, `load_corpus()`, `get_vector()`  

**Input:** CSV file containing network flow sequences  
**Output:** Vocabulary (`.pkl`), embedding matrix (`.npy`), Word2Vec model

---

#### `extract_stats_features.py` — Extract Statistical Features

**Functionality:**  
- Parse PCAP files and extract flow-level statistical features: lengths, directions, inter-arrival times (IAT)  
- Batch processing of multiple files

**Input:** PCAP file  
**Output:** CSV containing lengths, directions, IATs

---

#### `extract_stats_payload.py` — Extract Payload Features

**Functionality:**  
- Extract payload content as Bigram features  
- Combine with statistical features

**Input:** PCAP file  
**Output:** CSV file with labels, lengths, directions, IATs, payloads

---

### 2. Preprocessing (`preprocessing/`)

#### `split_dataset.py` — Split and Process Network Flows

**Functionality:**  
- Parse TCP connections from PCAP files  
- Normalize flows and split multi-connection sessions  
- Generate normalized five-tuples  

**Input:** PCAP file  
**Output:** Split PCAP files  

---

### 3. Model Training (`models/`)

#### `train.py` — Model Training Script

**Functionality:**  
- Train multimodal network traffic classification model  
- Fuse statistical and payload features  
- Evaluate model performance

---

## Data Pipeline
PCAP files (raw network packets)
↓
[preprocessing/split_dataset.py] → Split and normalize flows
↓
Split PCAP files
↓
[data/extract_stats_features.py] → Statistical features
[data/extract_stats_payload.py]  → Payload features
↓
CSV feature files
↓
[data/build_vocab.py] → Vocabulary + embeddings
↓
[models/train.py] → Train classification model
↓
Trained model
---

## Dependencies

```bash
pandas
numpy
gensim        # Word2Vec embedding
scapy         # Packet parsing
flowcontainer # Flow extraction
scikit-learn  # Model training
Usage
	1.	Split PCAP flows
python src/preprocessing/split_dataset.py
	2.	Extract network features
# Option A: Statistical features only
python src/data/extract_stats_features.py

# Option B: Statistical + Payload features
python src/data/extract_stats_payload.py
	3.	Build vocabulary and embeddings
python src/data/build_vocab.py
    4.	Train the model
python src/models/train.py

Key Features
	•	Multimodal feature fusion: statistical + payload features
	•	Robust flow splitting: supports bi-directional flows and multi-connection sessions
	•	Efficient representation: Word2Vec embeddings + Bigram payloads
	•	Encrypted traffic friendly: no payload decryption required

⸻

Notes
	•	Update input/output paths in scripts as needed
	•	Adjust NUM_WORKERS for system resources
	•	Minimum 5 packets per flow for processing
	•	IAT features are multiplied by 100 for discretization
