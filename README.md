# Synthetic Market Engine – Concept Release

**Motor de generación de datos sintéticos para Machine Learning de Producción en mercados financieros**

Sistema de generación de series temporales sintéticas OHLCV que preserva propiedades estadísticas reales mediante técnicas avanzadas de bootstrapping. Diseñado para validación de estrategias cuantitativas, entrenamiento de modelos ML y análisis de riesgo.

---

## Descripción

Herramienta de producción para generar escenarios alternativos de mercado que mantienen las dependencias estadísticas complejas de los datos históricos. Permite crear miles de variaciones sintéticas para stress-testing, data augmentation y estimación de riesgos de cola sin exponer lógica propietaria.

---

## Características Principales

- **Generación de datos sintéticos de alta fidelidad**: Produce series OHLCV indistinguibles estadísticamente de datos reales
- **Preservación de propiedades estilizadas**: Mantiene autocorrelación, clustering de volatilidad, fat tails y estructura temporal
- **Pipeline de validación automática**: Sistema de scoring que garantiza calidad mínima mediante rejection sampling
- **API REST y frontend web**: Interfaz completa para carga, generación y descarga de datasets
- **Métricas estadísticas integradas**: Validación automática con 15+ métricas (Hurst, ARCH, Ljung-Box, etc.)
- **Procesamiento en background**: Generación asíncrona con seguimiento de progreso en tiempo real

---

## Arquitectura / Diseño Conceptual

### Componentes del Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND (React)                         │
│  - Dashboard de visualización                              │
│  - Carga de archivos CSV                                   │
│  - Monitoreo de progreso                                   │
│  - Descarga de resultados                                  │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTP/REST
┌──────────────────────▼──────────────────────────────────────┐
│                 BACKEND API (FastAPI)                       │
│  - Endpoints de carga y generación                         │
│  - Gestión de jobs asíncronos                              │
│  - Cálculo de métricas                                     │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│            CORE ENGINE (SyntheticDataFactory)               │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ 1. PREPROCESSING MODULE                             │  │
│  │    - Normalización de columnas                      │  │
│  │    - Conversión a log-returns                       │  │
│  │    - Extracción de ratios OHLC                     │  │
│  │    - Cálculo de block size óptimo                  │  │
│  └─────────────────────────────────────────────────────┘  │
│                       │                                     │
│  ┌─────────────────────▼─────────────────────────────────┐ │
│  │ 2. GENERATION ENGINE                                  │ │
│  │    - Hybrid Block Bootstrap                          │ │
│  │    - Sampling de bloques aleatorios                 │ │
│  │    - Reconstrucción de paths de precio              │ │
│  └──────────────────────────────────────────────────────┘ │
│                       │                                     │
│  ┌─────────────────────▼─────────────────────────────────┐ │
│  │ 3. VALIDATION MODULE                                  │ │
│  │    - Cálculo de métricas estadísticas                │ │
│  │    - Comparación con datos originales               │ │
│  │    - Scoring de fidelidad                           │ │
│  └──────────────────────────────────────────────────────┘ │
│                       │                                     │
│  ┌─────────────────────▼─────────────────────────────────┐ │
│  │ 4. REJECTION SAMPLING                                 │ │
│  │    - Filtrado por umbral mínimo                     │ │
│  │    - Reintentos automáticos                          │ │
│  │    - Retorno de candidatos válidos                  │ │
│  └──────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Flujo de Procesamiento (Pipeline MLdP)

1. **Ingestión de Datos**
   - Carga de CSV con validación de formato
   - Normalización automática de columnas
   - Limpieza y preprocesamiento

2. **Análisis y Configuración**
   - Cálculo de métricas base del dataset original
   - Determinación de parámetros óptimos (block size)
   - Preparación de estructuras de datos

3. **Generación de Candidatos**
   - Sampling de bloques contiguos
   - Ensamblado de series sintéticas
   - Reconstrucción de OHLCV

4. **Validación Estadística**
   - Cálculo de 15+ métricas por candidato
   - Comparación con métricas originales
   - Scoring de similitud ponderado

5. **Filtrado y Salida**
   - Rejection sampling por umbral
   - Exportación a CSV/Excel
   - Agregación en archivos ZIP

---

## Ejemplos de Entrada/Salida (I/O)

### Entrada Esperada

**Formato CSV con columnas:**
- `Open`, `High`, `Low`, `Close`, `Volume`
- Opcional: `Date`, `Time`, `DateTime`

**Ejemplo de datos de entrada:**
```
Date,Open,High,Low,Close,Volume
2024-01-01,1.2500,1.2550,1.2480,1.2520,1500000
2024-01-02,1.2520,1.2580,1.2500,1.2560,1800000
...
```

### Salida Generada

**Múltiples series sintéticas en formato:**
- CSV individual por serie
- Excel (XLSX) por serie
- Archivos ZIP con batches completos

**Estructura de salida:**
```
synthetic_1_batch_xxx.csv
synthetic_2_batch_xxx.csv
...
synthetic_batch_xxx.zip (todos los CSV)
synthetic_batch_xxx_xlsx.zip (todos los XLSX)
```

### Pseudocódigo de Uso

```python
# Pseudocódigo conceptual - NO código real

# 1. Inicialización
factory = SyntheticDataFactory(input_dataframe)
# Sistema calcula automáticamente parámetros óptimos

# 2. Generación con validación
synthetic_series = factory.generate(
    n_samples=10,           # Número de series a generar
    min_fidelity=95.0      # Umbral de calidad mínimo (%)
)

# 3. Proceso interno (simplificado)
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

# 4. Resultado
# synthetic_series contiene N DataFrames válidos
```

---

## Pseudocódigo Seguro del Pipeline

### Módulo de Preprocesamiento

```python
def preprocess_data(raw_dataframe):
    # Normalización de columnas
    normalized_df = normalize_column_names(raw_dataframe)
    
    # Conversión a formato estacionario
    log_returns = calculate_log_returns(normalized_df['Close'])
    relative_ratios = extract_ohlc_ratios(normalized_df)
    
    # Configuración de parámetros
    optimal_block_size = calculate_optimal_block_size(data_length)
    
    return preprocessed_components, optimal_block_size
```

### Motor de Generación

```python
def generate_candidate(preprocessed_data, block_size):
    # Sampling de bloques
    number_of_blocks = calculate_required_blocks(data_length, block_size)
    random_start_indices = sample_random_indices(number_of_blocks)
    
    # Ensamblado de bloques
    synthetic_returns = []
    synthetic_ratios = []
    
    for start_index in random_start_indices:
        block = extract_block(preprocessed_data, start_index, block_size)
        synthetic_returns.append(block.returns)
        synthetic_ratios.append(block.ratios)
    
    # Reconstrucción
    synthetic_series = reconstruct_ohlcv(
        synthetic_returns, 
        synthetic_ratios, 
        initial_price
    )
    
    return synthetic_series
```

### Módulo de Validación

```python
def validate_candidate(candidate_series, original_metrics):
    # Cálculo de métricas del candidato
    candidate_metrics = calculate_metrics(candidate_series)
    
    # Comparación ponderada
    fidelity_score = weighted_similarity_score(
        original_metrics, 
        candidate_metrics
    )
    
    # Decisión
    if fidelity_score >= threshold:
        return ACCEPT, fidelity_score
    else:
        return REJECT, fidelity_score
```

### Pipeline Principal

```python
def generate_synthetic_data(input_data, n_samples, min_fidelity):
    # Fase 1: Preprocesamiento
    preprocessed, config = preprocess_data(input_data)
    original_metrics = calculate_metrics(input_data)
    
    # Fase 2: Generación con validación
    valid_samples = []
    
    while len(valid_samples) < n_samples:
        candidate = generate_candidate(preprocessed, config.block_size)
        decision, score = validate_candidate(candidate, original_metrics)
        
        if decision == ACCEPT:
            valid_samples.append(candidate)
        else:
            # Rejection sampling: descartar y reintentar
            continue
    
    return valid_samples
```

---

## Métricas Finales

El sistema calcula y valida las siguientes métricas estadísticas:

| Métrica | Descripción | Peso en Scoring |
|---------|-------------|-----------------|
| **Volatilidad** | Desviación estándar de returns | 3.0x |
| **Skewness** | Asimetría de distribución | 2.0x |
| **Kurtosis** | Colas de distribución | 2.0x |
| **Max Drawdown** | Máxima caída desde peak | 1.5x |
| **ACF Lag-1** | Autocorrelación de primer orden | 1.0x |
| **ACF Sum (10)** | Estructura autocorrelacional | 2.0x |
| **Vol Clustering** | Clustering de volatilidad | 3.0x |
| **Hurst Exponent** | Memoria de largo plazo | 2.0x |
| **Ljung-Box (p)** | Test de ruido blanco | 1.0x |
| **ARCH LM (p)** | Test de heteroscedasticidad | 1.0x |
| **RSI Correlation** | Coherencia con indicadores | 2.0x |
| **Vol Persistence** | Persistencia de regímenes | 2.0x |

**Fidelidad Típica**: 90-98% en datasets de calidad
**Tiempo de Generación**: 1-5 segundos por serie (datos diarios), 1-3 minutos (datos de alta frecuencia)

---

## Roadmap

### Fase 1: Optimizaciones Actuales
- [ ] Paralelización de generación de candidatos
- [ ] Caché de métricas precalculadas
- [ ] Soporte para múltiples timeframes simultáneos

### Fase 2: Extensiones de Modelado
- [ ] Incorporación de modelos GARCH/ARCH
- [ ] Soporte para datos multi-activo (correlaciones)
- [ ] Generación condicionada a regímenes de mercado

### Fase 3: Integración ML
- [ ] API para pipelines de ML automatizados
- [ ] Integración con frameworks de backtesting
- [ ] Exportación a formatos de ML (TFRecord, Parquet)

### Fase 4: Escalabilidad
- [ ] Distribución en cluster (Spark/Dask)
- [ ] Generación en streaming para datasets masivos
- [ ] Optimización de memoria para series ultra-largas

---

## Estructura de Carpetas Recomendada para GitHub

```
project/
├── README.md                 # Este archivo
├── LICENSE                   # Licencia (MIT/Apache)
├── .gitignore               # Exclusiones estándar
│
├── src/                     # Código fuente (NO incluido en repo público)
│   ├── core/               # Motor de generación
│   │   ├── preprocessing.py
│   │   ├── generator.py
│   │   └── validator.py
│   ├── api/                # Endpoints REST
│   └── utils/              # Utilidades
│
├── notebooks/              # Jupyter notebooks de ejemplo
│   ├── 01_basic_usage.ipynb
│   ├── 02_validation_metrics.ipynb
│   └── 03_ml_integration.ipynb
│
├── examples/               # Scripts de ejemplo (pseudocódigo)
│   ├── basic_generation.py
│   └── batch_processing.py
│
├── tests/                  # Tests unitarios (sin lógica privada)
│   ├── test_preprocessing.py
│   └── test_metrics.py
│
├── docs/                   # Documentación adicional
│   ├── architecture.md
│   ├── api_reference.md
│   └── methodology.md
│
└── data/                    # Datasets de ejemplo (sintéticos)
    └── sample_ohlcv.csv
```

**Nota**: Los archivos en `src/` contienen implementación propietaria y NO deben incluirse en el repositorio público.

---

## Sección Legal

**Note: This repository contains conceptual components only. Full implementation is proprietary to TradeAndRoll.**

Este repositorio presenta la arquitectura, metodología y documentación conceptual del sistema. La implementación completa, algoritmos específicos, heurísticas y fórmulas propietarias son propiedad exclusiva de TradeAndRoll y no están incluidas en este repositorio público.

---

## Resumen Ejecutivo Técnico

**Synthetic Market Engine** es un sistema de producción para generación de datos sintéticos financieros que implementa un pipeline completo de Machine Learning de Producción (MLdP). El sistema combina técnicas de bootstrapping avanzadas con validación estadística rigurosa para producir series temporales sintéticas que preservan las propiedades estilizadas de los mercados reales.

**Aplicaciones principales:**
- Stress-testing de estrategias cuantitativas
- Data augmentation para modelos de ML
- Estimación de riesgos de cola mediante Monte Carlo
- Validación de robustez de parámetros

**Ventajas técnicas:**
- Model-free: No asume distribuciones específicas
- Preserva dependencias temporales complejas
- Validación automática con múltiples métricas
- Escalable a datasets de alta frecuencia

---

## Contacto y Contribuciones

Para consultas sobre metodología o colaboraciones, contactar a través de los canales oficiales del proyecto.

**Versión**: Concept Release v1.0
**Última actualización**: 2024

