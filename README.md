# Synthetic Market Engine – Concept Release

**Synthetic data generation engine for Production Machine Learning in financial markets**

OHLCV synthetic time series generation system that preserves real statistical properties through advanced bootstrapping techniques. Designed for quantitative strategy validation, ML model training, and risk analysis.

---

## Description

Production tool to generate alternative market scenarios that maintain the complex statistical dependencies of historical data. Allows creating thousands of synthetic variations for stress-testing, data augmentation, and tail risk estimation without exposing proprietary logic.

---

## Main Features

- **High-fidelity synthetic data generation**: Produces OHLCV series statistically indistinguishable from real data
- **Stylized properties preservation**: Maintains autocorrelation, volatility clustering, fat tails, and temporal structure
- **Automatic validation pipeline**: Scoring system that guarantees minimum quality through rejection sampling
- **REST API and web frontend**: Complete interface for loading, generation, and dataset download
- **Integrated statistical metrics**: Automatic validation with 15+ metrics (Hurst, ARCH, Ljung-Box, etc.)
- **Background processing**: Asynchronous generation with real-time progress tracking

---

## Architecture / Conceptual Design

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND (React)                         │
│  - Visualization dashboard                                 │
│  - CSV file upload                                         │
│  - Progress monitoring                                      │
│  - Results download                                        │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTP/REST
┌──────────────────────▼──────────────────────────────────────┐
│                 BACKEND API (FastAPI)                       │
│  - Upload and generation endpoints                          │
│  - Asynchronous job management                             │
│  - Metrics calculation                                     │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│            CORE ENGINE (SyntheticDataFactory)               │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ 1. PREPROCESSING MODULE                             │  │
│  │    - Column normalization                           │  │
│  │    - Conversion to log-returns                       │  │
│  │    - OHLC ratio extraction                          │  │
│  │    - Optimal block size calculation                 │  │
│  └─────────────────────────────────────────────────────┘  │
│                       │                                     │
│  ┌─────────────────────▼─────────────────────────────────┐ │
│  │ 2. GENERATION ENGINE                                  │ │
│  │    - Hybrid Block Bootstrap                          │ │
│  │    - Random block sampling                           │ │
│  │    - Price path reconstruction                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                       │                                     │
│  ┌─────────────────────▼─────────────────────────────────┐ │
│  │ 3. VALIDATION MODULE                                  │ │
│  │    - Statistical metrics calculation                 │ │
│  │    - Comparison with original data                   │ │
│  │    - Fidelity scoring                                │ │
│  └──────────────────────────────────────────────────────┘ │
│                       │                                     │
│  ┌─────────────────────▼─────────────────────────────────┐ │
│  │ 4. REJECTION SAMPLING                                 │ │
│  │    - Filtering by minimum threshold                  │ │
│  │    - Automatic retries                               │ │
│  │    - Return of valid candidates                      │ │
│  └──────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Processing Flow (MLdP Pipeline)

1. **Data Ingestion**
   - CSV loading with format validation
   - Automatic column normalization
   - Cleaning and preprocessing

2. **Analysis and Configuration**
   - Base metrics calculation of original dataset
   - Optimal parameter determination (block size)
   - Data structure preparation

3. **Candidate Generation**
   - Contiguous block sampling
   - Synthetic series assembly
   - OHLCV reconstruction

4. **Statistical Validation**
   - 15+ metrics calculation per candidate
   - Comparison with original metrics
   - Weighted similarity scoring

5. **Filtering and Output**
   - Rejection sampling by threshold
   - Export to CSV/Excel
   - Aggregation in ZIP files

---

## Input/Output Examples (I/O)

### Expected Input

**CSV format with columns:**
- `Open`, `High`, `Low`, `Close`, `Volume`
- Optional: `Date`, `Time`, `DateTime`

**Input data example:**
```
Date,Open,High,Low,Close,Volume
2024-01-01,1.2500,1.2550,1.2480,1.2520,1500000
2024-01-02,1.2520,1.2580,1.2500,1.2560,1800000
...
```

### Generated Output

**Multiple synthetic series in format:**
- Individual CSV per series
- Excel (XLSX) per series
- ZIP files with complete batches

**Output structure:**
```
synthetic_1_batch_xxx.csv
synthetic_2_batch_xxx.csv
...
synthetic_batch_xxx.zip (all CSVs)
synthetic_batch_xxx_xlsx.zip (all XLSX)
```

### Usage Pseudocode

```python
# Conceptual pseudocode - NOT real code

# 1. Initialization
factory = SyntheticDataFactory(input_dataframe)
# System automatically calculates optimal parameters

# 2. Generation with validation
synthetic_series = factory.generate(
    n_samples=10,           # Number of series to generate
    min_fidelity=95.0      # Minimum quality threshold (%)
)

# 3. Internal process (simplified)
for each requested sample:
    attempts = 0
    while attempts < max_attempts:
        candidate = generate_candidate_series()
        metrics = calculate_statistical_metrics(candidate)
        fidelity_score = compare_with_original(metrics)
        
        if fidelity_score >= min_fidelity:
            accept_candidate()
            break
        else:
            attempts += 1
            continue
    
    if no_valid_candidate_found:
        raise_error()

# 4. Result
# synthetic_series contains N valid DataFrames
```

---

## Safe Pipeline Pseudocode

### Preprocessing Module

```python
def preprocess_data(raw_dataframe):
    # Column normalization
    normalized_df = normalize_column_names(raw_dataframe)
    
    # Conversion to stationary format
    log_returns = calculate_log_returns(normalized_df['Close'])
    relative_ratios = extract_ohlc_ratios(normalized_df)
    
    # Parameter configuration
    optimal_block_size = calculate_optimal_block_size(data_length)
    
    return preprocessed_components, optimal_block_size
```

### Generation Engine

```python
def generate_candidate(preprocessed_data, block_size):
    # Block sampling
    number_of_blocks = calculate_required_blocks(data_length, block_size)
    random_start_indices = sample_random_indices(number_of_blocks)
    
    # Block assembly
    synthetic_returns = []
    synthetic_ratios = []
    
    for start_index in random_start_indices:
        block = extract_block(preprocessed_data, start_index, block_size)
        synthetic_returns.append(block.returns)
        synthetic_ratios.append(block.ratios)
    
    # Reconstruction
    synthetic_series = reconstruct_ohlcv(
        synthetic_returns, 
        synthetic_ratios, 
        initial_price
    )
    
    return synthetic_series
```

### Validation Module

```python
def validate_candidate(candidate_series, original_metrics):
    # Candidate metrics calculation
    candidate_metrics = calculate_metrics(candidate_series)
    
    # Weighted comparison
    fidelity_score = weighted_similarity_score(
        original_metrics, 
        candidate_metrics
    )
    
    # Decision
    if fidelity_score >= threshold:
        return ACCEPT, fidelity_score
    else:
        return REJECT, fidelity_score
```

### Main Pipeline

```python
def generate_synthetic_data(input_data, n_samples, min_fidelity):
    # Phase 1: Preprocessing
    preprocessed, config = preprocess_data(input_data)
    original_metrics = calculate_metrics(input_data)
    
    # Phase 2: Generation with validation
    valid_samples = []
    
    while len(valid_samples) < n_samples:
        candidate = generate_candidate(preprocessed, config.block_size)
        decision, score = validate_candidate(candidate, original_metrics)
        
        if decision == ACCEPT:
            valid_samples.append(candidate)
        else:
            # Rejection sampling: discard and retry
            continue
    
    return valid_samples
```

---

## Final Metrics

The system calculates and validates the following statistical metrics:

| Metric | Description | Weight in Scoring |
|--------|-------------|-------------------|
| **Volatility** | Standard deviation of returns | 3.0x |
| **Skewness** | Distribution asymmetry | 2.0x |
| **Kurtosis** | Distribution tails | 2.0x |
| **Max Drawdown** | Maximum drop from peak | 1.5x |
| **ACF Lag-1** | First-order autocorrelation | 1.0x |
| **ACF Sum (10)** | Autocorrelation structure | 2.0x |
| **Vol Clustering** | Volatility clustering | 3.0x |
| **Hurst Exponent** | Long-term memory | 2.0x |
| **Ljung-Box (p)** | White noise test | 1.0x |
| **ARCH LM (p)** | Heteroscedasticity test | 1.0x |
| **RSI Correlation** | Coherence with indicators | 2.0x |
| **Vol Persistence** | Regime persistence | 2.0x |

**Typical Fidelity**: 90-98% on quality datasets  
**Generation Time**: 1-5 seconds per series (daily data), 1-3 minutes (high-frequency data)

---

## Roadmap

### Phase 1: Current Optimizations
- [ ] Parallelization of candidate generation
- [ ] Precalculated metrics cache
- [ ] Support for multiple simultaneous timeframes

### Phase 2: Modeling Extensions
- [ ] Incorporation of GARCH/ARCH models
- [ ] Multi-asset data support (correlations)
- [ ] Market regime-conditioned generation

### Phase 3: ML Integration
- [ ] API for automated ML pipelines
- [ ] Integration with backtesting frameworks
- [ ] Export to ML formats (TFRecord, Parquet)

### Phase 4: Scalability
- [ ] Cluster distribution (Spark/Dask)
- [ ] Streaming generation for massive datasets
- [ ] Memory optimization for ultra-long series

---

## Recommended Folder Structure for GitHub

```
project/
├── README.md                 # This file
├── LICENSE                   # License (MIT/Apache)
├── .gitignore               # Standard exclusions
│
├── src/                     # Source code (NOT included in public repo)
│   ├── core/               # Generation engine
│   │   ├── preprocessing.py
│   │   ├── generator.py
│   │   └── validator.py
│   ├── api/                # REST endpoints
│   └── utils/              # Utilities
│
├── notebooks/              # Example Jupyter notebooks
│   ├── 01_basic_usage.ipynb
│   ├── 02_validation_metrics.ipynb
│   └── 03_ml_integration.ipynb
│
├── examples/               # Example scripts (pseudocode)
│   ├── basic_generation.py
│   └── batch_processing.py
│
├── tests/                  # Unit tests (without private logic)
│   ├── test_preprocessing.py
│   └── test_metrics.py
│
├── docs/                   # Additional documentation
│   ├── architecture.md
│   ├── api_reference.md
│   └── methodology.md
│
└── data/                    # Example datasets (synthetic)
    └── sample_ohlcv.csv
```

**Note**: Files in `src/` contain proprietary implementation and should NOT be included in the public repository.

---

## Legal Section

**Note: This repository contains conceptual components only. Full implementation is proprietary to TradeAndRoll.**

This repository presents the architecture, methodology, and conceptual documentation of the system. The complete implementation, specific algorithms, heuristics, and proprietary formulas are the exclusive property of TradeAndRoll and are not included in this public repository.

---

## Technical Executive Summary

**Synthetic Market Engine** is a production system for generating synthetic financial data that implements a complete Production Machine Learning (MLdP) pipeline. The system combines advanced bootstrapping techniques with rigorous statistical validation to produce synthetic time series that preserve the stylized properties of real markets.

**Main applications:**
- Stress-testing of quantitative strategies
- Data augmentation for ML models
- Tail risk estimation through Monte Carlo
- Parameter robustness validation

**Technical advantages:**
- Model-free: Does not assume specific distributions
- Preserves complex temporal dependencies
- Automatic validation with multiple metrics
- Scalable to high-frequency datasets

---

## Contact and Contributions

For methodology inquiries or collaborations, contact through the project's official channels.

**Version**: Concept Release v1.0  
**Last update**: 2024

