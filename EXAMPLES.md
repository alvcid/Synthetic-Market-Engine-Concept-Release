# Ejemplos de Uso - Entrada y Salida

Este documento describe los formatos de entrada y salida del sistema, así como casos de uso típicos.

---

## Formato de Entrada

### Requisitos de Datos

El sistema espera archivos CSV con la siguiente estructura mínima:

**Columnas requeridas:**
- `Open` - Precio de apertura
- `High` - Precio máximo
- `Low` - Precio mínimo  
- `Close` - Precio de cierre
- `Volume` - Volumen transaccionado

**Columnas opcionales:**
- `Date`, `Time`, `DateTime`, `Timestamp` - Información temporal

### Ejemplo de CSV de Entrada

```csv
Date,Open,High,Low,Close,Volume
2024-01-01,1.2500,1.2550,1.2480,1.2520,1500000
2024-01-02,1.2520,1.2580,1.2500,1.2560,1800000
2024-01-03,1.2560,1.2600,1.2540,1.2580,1650000
2024-01-04,1.2580,1.2620,1.2560,1.2600,1900000
2024-01-05,1.2600,1.2650,1.2580,1.2630,1750000
```

### Variantes de Nombres Soportadas

El sistema normaliza automáticamente nombres de columnas:

- **Open**: `Open`, `Op`, `O`, `OPEN`
- **High**: `High`, `Hi`, `H`, `HIGH`
- **Low**: `Low`, `Lo`, `L`, `LOW`
- **Close**: `Close`, `Cl`, `C`, `CLOSE`
- **Volume**: `Volume`, `Vol`, `V`, `Tick_Volume`, `TickVol`

---

## Formato de Salida

### Archivos Individuales

Por cada serie sintética generada, se crean dos archivos:

1. **CSV**: `synthetic_{sample_id}_{batch_id}.csv`
2. **Excel**: `synthetic_{sample_id}_{batch_id}.xlsx`

**Estructura del CSV de salida:**
```csv
Open,High,Low,Close,Volume
1.2500,1.2552,1.2478,1.2518,1520000
1.2518,1.2581,1.2498,1.2562,1810000
1.2562,1.2601,1.2541,1.2581,1640000
...
```

### Archivos Agregados (ZIP)

Para facilitar descarga de múltiples series:

1. **ZIP CSV**: `synthetic_batch_{batch_id}.zip`
   - Contiene todos los CSV de la generación

2. **ZIP Excel**: `synthetic_batch_{batch_id}_xlsx.zip`
   - Contiene todos los XLSX de la generación

---

## Ejemplo de Flujo Completo

### Escenario: Generación de 5 Series Sintéticas

**Input:**
- Archivo: `EURUSD_Daily_2023.csv` (500 barras diarias)
- Parámetros: `n_samples=5`, `min_fidelity=95%`

**Proceso:**
1. Sistema carga y valida CSV
2. Calcula métricas originales (volatilidad, skewness, etc.)
3. Genera 5 candidatos con validación
4. Cada candidato es validado contra métricas originales
5. Solo se aceptan candidatos con fidelidad ≥ 95%

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
├── synthetic_batch_abc123.zip (todos los CSV)
└── synthetic_batch_abc123_xlsx.zip (todos los XLSX)
```

---

## Casos de Uso

### Caso 1: Stress-Testing de Estrategia

**Objetivo:** Validar robustez de estrategia de trading

**Input:**
- Dataset histórico: 2 años de datos diarios de SPY
- Generación: 100 series sintéticas
- Fidelidad mínima: 90%

**Proceso:**
1. Generar 100 variaciones sintéticas
2. Ejecutar backtest de estrategia en cada serie
3. Analizar distribución de resultados (Sharpe, drawdown, etc.)

**Output esperado:**
- 100 series sintéticas validadas
- Distribución de métricas de performance
- Identificación de escenarios extremos

---

### Caso 2: Data Augmentation para ML

**Objetivo:** Aumentar dataset de entrenamiento para modelo de ML

**Input:**
- Dataset histórico: 1 año de datos de 1-minuto de BTC
- Generación: 1000 series sintéticas
- Fidelidad mínima: 92%

**Proceso:**
1. Generar 1000 series sintéticas
2. Combinar con datos originales
3. Entrenar modelo en dataset aumentado
4. Validar en datos reales (hold-out)

**Output esperado:**
- Dataset aumentado 1000x
- Mejora en generalización del modelo
- Reducción de overfitting

---

### Caso 3: Análisis de Riesgo de Cola

**Objetivo:** Estimar drawdown máximo en escenarios extremos

**Input:**
- Dataset histórico: 10 años de datos diarios de portafolio
- Generación: 10000 series sintéticas
- Fidelidad mínima: 88%

**Proceso:**
1. Generar 10000 escenarios alternativos
2. Calcular drawdown máximo de cada serie
3. Analizar percentil 1% (peor caso)

**Output esperado:**
- Distribución de drawdowns
- Estimación de VaR (Value at Risk)
- Identificación de escenarios de stress

---

## Métricas de Validación Incluidas

Cada serie sintética generada incluye un reporte de métricas comparativas:

**Ejemplo de métricas calculadas:**
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

## Consideraciones de Rendimiento

### Tiempos Estimados

- **Datos diarios (1000 barras)**: 1-3 segundos por serie
- **Datos horarios (10000 barras)**: 5-15 segundos por serie
- **Datos de 1-minuto (100000 barras)**: 30-120 segundos por serie

### Factores que Afectan Rendimiento

- Longitud del dataset original
- Frecuencia de los datos
- Umbral de fidelidad requerido (más alto = más intentos)
- Número de series solicitadas

---

## Limitaciones y Recomendaciones

### Limitaciones

- Requiere datos continuos (sin gaps grandes)
- Funciona mejor con datos estacionarios
- Rendimiento degrada con datasets muy grandes (>500k barras)

### Recomendaciones

- Usar datos limpios y validados
- Para datasets grandes, considerar muestreo previo
- Ajustar `min_fidelity` según necesidad (90-95% típico)
- Para producción, usar procesamiento en background

