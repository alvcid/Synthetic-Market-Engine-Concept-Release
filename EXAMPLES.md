# Usage Examples - Input and Output

This document describes the system's input and output formats, as well as typical use cases.

---

## Input Format

### Data Requirements

The system expects CSV files with the following minimum structure:

**Required columns:**
- `Open` - Opening price
- `High` - Maximum price
- `Low` - Minimum price  
- `Close` - Closing price
- `Volume` - Traded volume

**Optional columns:**
- `Date`, `Time`, `DateTime`, `Timestamp` - Temporal information

### Input CSV Example

```csv
Date,Open,High,Low,Close,Volume
2024-01-01,1.2500,1.2550,1.2480,1.2520,1500000
2024-01-02,1.2520,1.2580,1.2500,1.2560,1800000
2024-01-03,1.2560,1.2600,1.2540,1.2580,1650000
2024-01-04,1.2580,1.2620,1.2560,1.2600,1900000
2024-01-05,1.2600,1.2650,1.2580,1.2630,1750000
```

### Supported Name Variants

The system automatically normalizes column names:

- **Open**: `Open`, `Op`, `O`, `OPEN`
- **High**: `High`, `Hi`, `H`, `HIGH`
- **Low**: `Low`, `Lo`, `L`, `LOW`
- **Close**: `Close`, `Cl`, `C`, `CLOSE`
- **Volume**: `Volume`, `Vol`, `V`, `Tick_Volume`, `TickVol`

---

## Output Format

### Individual Files

For each generated synthetic series, two files are created:

1. **CSV**: `synthetic_{sample_id}_{batch_id}.csv`
2. **Excel**: `synthetic_{sample_id}_{batch_id}.xlsx`

**Output CSV structure:**
```csv
Open,High,Low,Close,Volume
1.2500,1.2552,1.2478,1.2518,1520000
1.2518,1.2581,1.2498,1.2562,1810000
1.2562,1.2601,1.2541,1.2581,1640000
...
```

### Aggregated Files (ZIP)

To facilitate download of multiple series:

1. **CSV ZIP**: `synthetic_batch_{batch_id}.zip`
   - Contains all CSVs from the generation

2. **Excel ZIP**: `synthetic_batch_{batch_id}_xlsx.zip`
   - Contains all XLSX from the generation

---

## Complete Flow Example

### Scenario: Generation of 5 Synthetic Series

**Input:**
- File: `EURUSD_Daily_2023.csv` (500 daily bars)
- Parameters: `n_samples=5`, `min_fidelity=95%`

**Process:**
1. System loads and validates CSV
2. Calculates original metrics (volatility, skewness, etc.)
3. Generates 5 candidates with validation
4. Each candidate is validated against original metrics
5. Only candidates with fidelity ≥ 95% are accepted

**Output:**
```
outputs/
├── synthetic_1_batch_abc123.csv
├── synthetic_1_batch_abc123.xlsx
├── synthetic_2_batch_abc123.csv
├── synthetic_2_batch_abc123.xlsx
├── synthetic_3_batch_abc123.csv
├── synthetic_3_batch_abc123.xlsx
├── synthetic_4_batch_abc123.csv
├── synthetic_4_batch_abc123.xlsx
├── synthetic_5_batch_abc123.csv
├── synthetic_5_batch_abc123.xlsx
├── synthetic_batch_abc123.zip (all CSVs)
└── synthetic_batch_abc123_xlsx.zip (all XLSX)
```

---

## Use Cases

### Case 1: Strategy Stress-Testing

**Objective:** Validate trading strategy robustness

**Input:**
- Historical dataset: 2 years of daily SPY data
- Generation: 100 synthetic series
- Minimum fidelity: 90%

**Process:**
1. Generate 100 synthetic variations
2. Run strategy backtest on each series
3. Analyze distribution of results (Sharpe, drawdown, etc.)

**Expected output:**
- 100 validated synthetic series
- Distribution of performance metrics
- Identification of extreme scenarios

---

### Case 2: Data Augmentation for ML

**Objective:** Increase training dataset for ML model

**Input:**
- Historical dataset: 1 year of 1-minute BTC data
- Generation: 1000 synthetic series
- Minimum fidelity: 92%

**Process:**
1. Generate 1000 synthetic series
2. Combine with original data
3. Train model on augmented dataset
4. Validate on real data (hold-out)

**Expected output:**
- 1000x augmented dataset
- Improved model generalization
- Reduced overfitting

---

### Case 3: Tail Risk Analysis

**Objective:** Estimate maximum drawdown in extreme scenarios

**Input:**
- Historical dataset: 10 years of daily portfolio data
- Generation: 10000 synthetic series
- Minimum fidelity: 88%

**Process:**
1. Generate 10000 alternative scenarios
2. Calculate maximum drawdown of each series
3. Analyze 1% percentile (worst case)

**Expected output:**
- Drawdown distribution
- VaR (Value at Risk) estimation
- Stress scenario identification

---

## Included Validation Metrics

Each generated synthetic series includes a comparative metrics report:

**Example of calculated metrics:**
```json
{
  "original": {
    "volatility": 0.0156,
    "skewness": -0.32,
    "kurtosis": 5.8,
    "max_drawdown": -0.18,
    "hurst": 0.52
  },
  "synthetic": {
    "volatility": 0.0154,
    "skewness": -0.31,
    "kurtosis": 5.7,
    "max_drawdown": -0.17,
    "hurst": 0.51
  },
  "fidelity_score": 96.2
}
```

---

## Performance Considerations

### Estimated Times

- **Daily data (1000 bars)**: 1-3 seconds per series
- **Hourly data (10000 bars)**: 5-15 seconds per series
- **1-minute data (100000 bars)**: 30-120 seconds per series

### Factors Affecting Performance

- Original dataset length
- Data frequency
- Required fidelity threshold (higher = more attempts)
- Number of requested series

---

## Limitations and Recommendations

### Limitations

- Requires continuous data (no large gaps)
- Works better with stationary data
- Performance degrades with very large datasets (>500k bars)

### Recommendations

- Use clean and validated data
- For large datasets, consider prior sampling
- Adjust `min_fidelity` according to need (90-95% typical)
- For production, use background processing

