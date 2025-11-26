# Arquitectura del Sistema - Pipeline MLdP

## Visión General

Este documento describe la arquitectura conceptual del pipeline completo de Machine Learning de Producción (MLdP) implementado en el Synthetic Market Engine.

---

## Pipeline de Procesamiento

### Etapa 1: Ingestión y Validación

**Qué hace:**
- Recibe archivos CSV con datos históricos OHLCV
- Valida estructura y formato de columnas
- Normaliza nombres de columnas (soporta múltiples variantes)
- Limpia datos inválidos o faltantes

**Componentes:**
- Módulo de parsing flexible
- Normalizador de columnas
- Validador de integridad

**Salida:** DataFrame normalizado y validado

---

### Etapa 2: Preprocesamiento y Análisis

**Qué hace:**
- Convierte precios a log-returns (estacionariedad)
- Extrae ratios relativos OHLC (preservación de estructura de velas)
- Calcula métricas estadísticas base del dataset original
- Determina parámetros óptimos (block size) según longitud de datos

**Componentes:**
- Transformador de precios a returns
- Extractor de ratios OHLC
- Calculador de métricas base
- Optimizador de parámetros

**Salida:** Estructuras preprocesadas + métricas de referencia + configuración

---

### Etapa 3: Generación de Candidatos

**Qué hace:**
- Aplica Hybrid Block Bootstrap para crear series sintéticas
- Muestrea bloques contiguos aleatorios de los datos originales
- Ensambla bloques en nuevas series temporales
- Reconstruye paths de precio y estructura OHLCV completa

**Componentes:**
- Motor de sampling de bloques
- Ensamblador de series
- Reconstructor de OHLCV

**Salida:** Series candidatas sintéticas

---

### Etapa 4: Validación Estadística

**Qué hace:**
- Calcula 15+ métricas estadísticas para cada candidato
- Compara métricas con las del dataset original
- Calcula score de fidelidad mediante comparación ponderada
- Evalúa si el candidato cumple umbral mínimo

**Métricas calculadas:**
- Momentos: media, volatilidad, skewness, kurtosis
- Estructura temporal: ACF, Ljung-Box, Hurst
- Volatilidad: clustering, ARCH effects, persistencia
- Coherencia: correlaciones retorno-volumen, RSI, leverage effect

**Componentes:**
- Calculador de métricas estadísticas
- Comparador ponderado
- Evaluador de umbrales

**Salida:** Score de fidelidad (0-100%) + decisión (aceptar/rechazar)

---

### Etapa 5: Rejection Sampling

**Qué hace:**
- Filtra candidatos que no cumplen umbral mínimo
- Reintenta generación hasta obtener candidatos válidos
- Limita número máximo de intentos por muestra
- Acumula muestras válidas hasta alcanzar cantidad solicitada

**Componentes:**
- Filtro por umbral
- Controlador de reintentos
- Acumulador de muestras válidas

**Salida:** Lista de N series sintéticas validadas

---

### Etapa 6: Post-procesamiento y Exportación

**Qué hace:**
- Formatea resultados para exportación
- Genera archivos CSV y Excel individuales
- Crea archivos ZIP con batches completos
- Prepara metadatos y reportes de validación

**Componentes:**
- Formateador de salida
- Generador de archivos
- Agregador de batches

**Salida:** Archivos listos para descarga

---

## Flujo de Datos Completo

```
[CSV Input]
    │
    ▼
[Validación y Normalización]
    │
    ▼
[Preprocesamiento]
    ├─► Log Returns
    ├─► Ratios OHLC
    ├─► Métricas Originales
    └─► Configuración
    │
    ▼
┌─────────────────────────────────────┐
│   LOOP: Generación + Validación    │
│                                     │
│   [Generar Candidato]              │
│         │                           │
│         ▼                           │
│   [Calcular Métricas]              │
│         │                           │
│         ▼                           │
│   [Comparar con Original]          │
│         │                           │
│         ▼                           │
│   [Score >= Umbral?]               │
│         │                           │
│    ┌────┴────┐                     │
│    │         │                     │
│   SÍ        NO                     │
│    │         │                     │
│    ▼         ▼                     │
│ [Aceptar] [Rechazar y Reintentar] │
│    │                                │
│    └─────────┐                      │
│              │                      │
└──────────────┼──────────────────────┘
               │
               ▼
        [N Muestras Válidas]
               │
               ▼
        [Exportación]
               │
               ▼
        [CSV/Excel/ZIP]
```

---

## Componentes del Sistema

### Backend API (FastAPI)

**Responsabilidades:**
- Endpoints REST para carga de archivos
- Gestión de jobs asíncronos
- Cálculo de métricas en tiempo real
- Servicio de descarga de resultados

**Endpoints principales:**
- `POST /upload` - Carga de CSV
- `POST /generate` - Inicio de generación
- `GET /job-status/{job_id}` - Estado de progreso
- `GET /download/{filename}` - Descarga de resultados

---

### Core Engine (SyntheticDataFactory)

**Responsabilidades:**
- Orquestación del pipeline completo
- Gestión de estado interno
- Coordinación entre módulos
- Control de calidad

**Métodos principales:**
- `__init__()` - Inicialización y preprocesamiento
- `generate()` - Generación con validación
- `_generate_candidate()` - Creación de candidatos
- `_calculate_metrics()` - Cálculo de métricas
- `_calculate_similarity()` - Scoring de fidelidad

---

### Frontend (React)

**Responsabilidades:**
- Interfaz de usuario para carga de archivos
- Visualización de datos originales y sintéticos
- Monitoreo de progreso de generación
- Descarga de resultados

**Componentes principales:**
- Dashboard - Interfaz principal
- Upload Zone - Carga de archivos
- Visualization - Gráficos de series
- Progress Tracker - Seguimiento de jobs
- Metrics Display - Tabla de métricas

---

## Consideraciones de Diseño

### Escalabilidad

- Procesamiento asíncrono para no bloquear API
- Generación progresiva (muestra por muestra)
- Optimización de memoria para datasets grandes

### Robustez

- Manejo de errores en cada etapa
- Validación exhaustiva de inputs
- Fallbacks para casos edge

### Calidad

- Rejection sampling estricto
- Múltiples métricas de validación
- Scoring ponderado por importancia

---

## Integración con MLdP

Este pipeline sigue principios de Machine Learning de Producción:

1. **Versionado de Datos**: Tracking de datasets de entrada
2. **Validación Automática**: Tests estadísticos integrados
3. **Monitoreo**: Métricas en tiempo real durante generación
4. **Reproducibilidad**: Parámetros configurables y documentados
5. **Escalabilidad**: Diseño para procesamiento en background

---

## Notas de Implementación

- El sistema está diseñado para ser modular y extensible
- Cada etapa puede optimizarse independientemente
- La validación estadística es configurable por métricas
- El rejection sampling permite control fino de calidad

