# System Architecture - MLdP Pipeline

## Overview

This document describes the conceptual architecture of the complete Production Machine Learning (MLdP) pipeline implemented in the Synthetic Market Engine.

---

## Processing Pipeline

### Stage 1: Ingestion and Validation

**What it does:**
- Receives CSV files with historical OHLCV data
- Validates structure and column format
- Normalizes column names (supports multiple variants)
- Cleans invalid or missing data

**Components:**
- Flexible parsing module
- Column normalizer
- Integrity validator

**Output:** Normalized and validated DataFrame

---

### Stage 2: Preprocessing and Analysis

**What it does:**
- Converts prices to log-returns (stationarity)
- Extracts relative OHLC ratios (candle structure preservation)
- Calculates base statistical metrics of original dataset
- Determines optimal parameters (block size) according to data length

**Components:**
- Price to returns transformer
- OHLC ratio extractor
- Base metrics calculator
- Parameter optimizer

**Output:** Preprocessed structures + reference metrics + configuration

---

### Stage 3: Candidate Generation

**What it does:**
- Applies Hybrid Block Bootstrap to create synthetic series
- Samples random contiguous blocks from original data
- Assembles blocks into new time series
- Reconstructs price paths and complete OHLCV structure

**Components:**
- Block sampling engine
- Series assembler
- OHLCV reconstructor

**Output:** Synthetic candidate series

---

### Stage 4: Statistical Validation

**What it does:**
- Calculates 15+ statistical metrics for each candidate
- Compares metrics with those of original dataset
- Calculates fidelity score through weighted comparison
- Evaluates if candidate meets minimum threshold

**Calculated metrics:**
- Moments: mean, volatility, skewness, kurtosis
- Temporal structure: ACF, Ljung-Box, Hurst
- Volatility: clustering, ARCH effects, persistence
- Coherence: return-volume correlations, RSI, leverage effect

**Components:**
- Statistical metrics calculator
- Weighted comparator
- Threshold evaluator

**Output:** Fidelity score (0-100%) + decision (accept/reject)

---

### Stage 5: Rejection Sampling

**What it does:**
- Filters candidates that do not meet minimum threshold
- Retries generation until valid candidates are obtained
- Limits maximum number of attempts per sample
- Accumulates valid samples until requested quantity is reached

**Components:**
- Threshold filter
- Retry controller
- Valid sample accumulator

**Output:** List of N validated synthetic series

---

### Stage 6: Post-processing and Export

**What it does:**
- Formats results for export
- Generates individual CSV and Excel files
- Creates ZIP files with complete batches
- Prepares metadata and validation reports

**Components:**
- Output formatter
- File generator
- Batch aggregator

**Output:** Files ready for download

---

## Complete Data Flow

```
[CSV Input]
    │
    ▼
[Validation and Normalization]
    │
    ▼
[Preprocessing]
    ├─► Log Returns
    ├─► OHLC Ratios
    ├─► Original Metrics
    └─► Configuration
    │
    ▼
┌─────────────────────────────────────┐
│   LOOP: Generation + Validation    │
│                                     │
│   [Generate Candidate]              │
│         │                           │
│         ▼                           │
│   [Calculate Metrics]              │
│         │                           │
│         ▼                           │
│   [Compare with Original]          │
│         │                           │
│         ▼                           │
│   [Score >= Threshold?]            │
│         │                           │
│    ┌────┴────┐                     │
│    │         │                     │
│   YES       NO                     │
│    │         │                     │
│    ▼         ▼                     │
│ [Accept] [Reject and Retry]        │
│    │                                │
│    └─────────┐                      │
│              │                      │
└──────────────┼──────────────────────┘
               │
               ▼
        [N Valid Samples]
               │
               ▼
        [Export]
               │
               ▼
        [CSV/Excel/ZIP]
```

---

## System Components

### Backend API (FastAPI)

**Responsibilities:**
- REST endpoints for file upload
- Asynchronous job management
- Real-time metrics calculation
- Results download service

**Main endpoints:**
- `POST /upload` - CSV upload
- `POST /generate` - Generation start
- `GET /job-status/{job_id}` - Progress status
- `GET /download/{filename}` - Results download

---

### Core Engine (SyntheticDataFactory)

**Responsibilities:**
- Complete pipeline orchestration
- Internal state management
- Module coordination
- Quality control

**Main methods:**
- `__init__()` - Initialization and preprocessing
- `generate()` - Generation with validation
- `_generate_candidate()` - Candidate creation
- `_calculate_metrics()` - Metrics calculation
- `_calculate_similarity()` - Fidelity scoring

---

### Frontend (React)

**Responsibilities:**
- User interface for file upload
- Visualization of original and synthetic data
- Generation progress monitoring
- Results download

**Main components:**
- Dashboard - Main interface
- Upload Zone - File upload
- Visualization - Series charts
- Progress Tracker - Job tracking
- Metrics Display - Metrics table

---

## Design Considerations

### Scalability

- Asynchronous processing to avoid blocking API
- Progressive generation (sample by sample)
- Memory optimization for large datasets

### Robustness

- Error handling at each stage
- Exhaustive input validation
- Fallbacks for edge cases

### Quality

- Strict rejection sampling
- Multiple validation metrics
- Weighted scoring by importance

---

## MLdP Integration

This pipeline follows Production Machine Learning principles:

1. **Data Versioning**: Tracking of input datasets
2. **Automatic Validation**: Integrated statistical tests
3. **Monitoring**: Real-time metrics during generation
4. **Reproducibility**: Configurable and documented parameters
5. **Scalability**: Design for background processing

---

## Implementation Notes

- The system is designed to be modular and extensible
- Each stage can be optimized independently
- Statistical validation is configurable by metrics
- Rejection sampling allows fine quality control

