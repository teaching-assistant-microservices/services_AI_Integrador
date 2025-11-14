# Plataforma Educativa Inteligente

Solución integral basada en inteligencia artificial para la gestión, búsqueda y generación automática de recursos educativos. La plataforma funciona como una biblioteca digital inteligente con asistencia conversacional, organización automática de contenidos y generación de materiales pedagógicos personalizados.

---

## Contenidos

- [Descripción General](#descripción-general)
- [Características Principales](#características-principales)
- [Arquitectura del Sistema](#arquitectura-del-sistema)
- [Core IA - Núcleo Inteligente](#core-ia---núcleo-inteligente)
- [Servicios Principales](#servicios-principales)
- [Especificación de API](#especificación-de-api)
- [Integración con Files Service](#integración-con-files-service)
- [Requisitos Técnicos](#requisitos-técnicos)
- [Seguridad](#seguridad)
- [Monitoreo y Observabilidad](#monitoreo-y-observabilidad)

---

## Descripción General

La Plataforma Educativa Inteligente es un sistema que permite a docentes y educadores:

1. Gestionar una biblioteca centralizada de recursos educativos (guías, libros, investigaciones, técnicas pedagógicas)
2. Realizar consultas en lenguaje natural mediante un asistente IA conversacional
3. Obtener recursos recomendados con referencias exactas a documentos y páginas
4. Generar automáticamente guías de estudio y planes de clase personalizados
5. Descargar materiales pedagógicos completos en formato DOCX/PDF
6. Explorar organización automática de contenidos mediante clustering
7. Acceder a búsqueda web automática cuando los recursos locales son insuficientes

---

## Características Principales

### Biblioteca Educativa Digital

- Almacenamiento centralizado de recursos educativos
- Clasificación automática por temas, niveles y tipos de documento
- Búsqueda semántica mediante embeddings
- Gestión de múltiples formatos (PDF, DOCX, imágenes con OCR)
- Soporte para documentos extensos con chunking inteligente

### Asistente Pedagógico Conversacional

- Consultas en lenguaje natural
- Respuestas contextualizadas con referencias exactas a documentos
- Historial conversacional mantenido en sesión
- Preguntas aclaratorias para refinar búsquedas
- Integración de DeepSeek como LLM conversacional

### Generación Automática de Materiales

- Guías de estudio personalizadas por nivel educativo
- Ejercicios y evaluaciones adaptadas al contexto
- Planes de clase completos con objetivos y metodología
- Recomendaciones de técnicas pedagógicas
- Cronogramas de aprendizaje estructurados

### Exportación de Planes Educativos

- Generación automática de documentos profesionales
- Adaptación a diferentes niveles (primaria, secundaria, universidad)
- Inclusión de objetivos, metodología, recursos y evaluación
- Exportación a DOCX y PDF
- Documentos listos para usar inmediatamente en clase

### Organización Inteligente mediante ML

- Clustering automático de documentos similares
- Descubrimiento de patrones temáticos
- Mapas conceptuales de relaciones entre contenidos
- Recomendaciones basadas en contexto educativo
- Identificación de gaps de conocimiento

### Enriquecimiento Web Automático

- Búsqueda externa cuando recursos locales son insuficientes
- Integración con motores de búsqueda educativos (DuckDuckGo, SearXNG, ERIC)
- Scoring automático de confiabilidad de fuentes
- Descarga controlada de documentos educativos
- Reingesta automática de nuevos materiales

---

## Arquitectura del Sistema

```
Cliente (Interfaz Web)
    |
    v
API Gateway (FastAPI)
    |
    +---> Core IA (Orquestador Principal)
    |        |
    |        +---> Ingesta de Documentos
    |        +---> Verificación y Deduplicación
    |        +---> Clasificación y Etiquetado
    |        +---> Chat y Búsqueda Semántica
    |        +---> Enriquecimiento Web
    |
    +---> Files Service (HTTP REST)
    |        - Gestión de almacenamiento S3
    |        - Validación de archivos
    |        - Acceso a documentos
    |        - Arquitectura hexagonal (NestJS)
    |
    +---> PostgreSQL + pgvector
             - Almacenamiento de embeddings
             - Búsqueda vectorial HNSW
             - Metadatos de documentos
```

---

## Core IA - Núcleo Inteligente

El Core IA es el componente central que orquesta el procesamiento de documentos y la interacción conversacional. Está compuesto por cinco microservicios especializados que se comunican mediante TCP/IP:

### Arquitectura de Servicios

```
CORE IA (FastAPI Orquestador)
|
+---> Servicio de Ingesta
|     - Extracción de texto
|     - Chunking inteligente (512 tokens, overlap 100)
|     - Generación de embeddings (MiniLM-L6-v2, 384 dimensiones)
|     - Comunicación TCP con Files Service
|
+---> Servicio de Verificación
|     - Hash SHA-256 de documentos
|     - Búsqueda de similitud semántica (umbral 0.95)
|     - Análisis de metadatos
|     - Prevención de duplicados
|
+---> Servicio de Clasificación
|     - KMeans clustering
|     - BERTopic para detección de tópicos
|     - KeyBERT para extracción de keywords
|     - Asignación de categorías y etiquetas
|
+---> Servicio de Chat
|     - Búsqueda semántica en pgvector
|     - Generación de contexto
|     - Integración con DeepSeek LLM
|     - Gestión de historial conversacional
|
+---> Servicio de Enriquecimiento Web
     - Búsqueda en múltiples motores
     - Scoring de confiabilidad
     - Descarga controlada de documentos
     - Reingesta automática
```

---

## Servicios Principales

### 1. Servicio de Ingesta de Documentos

Responsable del procesamiento automático de documentos subidos por usuarios.

**Entrada:** Documento PDF, DOCX o imagen (vía Files Service)

**Proceso:**
1. Extracción de texto mediante PyMuPDF, pdfminer.six o Tesseract OCR
2. Limpieza y normalización del texto
3. División en chunks de 512 tokens con overlap de 100 tokens
4. Generación de embeddings usando sentence-transformers/all-MiniLM-L6-v2
5. Comunicación con Servicio de Verificación para detectar duplicados
6. Indexación en pgvector con índice HNSW
7. Comunicación con Servicio de Clasificación

**Salida:** Documento procesado, indexado y clasificado

**Endpoint:** `POST /api/v1/ingestion/process`

### 2. Servicio de Verificación y Deduplicación

Previene ingesta de documentos duplicados mediante análisis multilayer.

**Verificación:**
1. Cálculo de hash SHA-256 del documento
2. Búsqueda exacta en índice de hashes
3. Si no coincide exacto: búsqueda de similitud coseno (umbral 0.95)
4. Análisis de metadatos (título, autor, fecha)
5. Decisión: aceptar, rechazar o marcar para revisión manual

**Endpoint:** `POST /api/v1/verification/check-duplicate`

### 3. Servicio de Clasificación y Etiquetado

Asigna automáticamente categorías, etiquetas y keywords a documentos mediante técnicas avanzadas de Machine Learning. Este servicio es crítico para la organización inteligente de la plataforma.

#### 3.1 Arquitectura de Clasificación

El servicio implementa un pipeline de machine learning que combina tres técnicas complementarias:

```
Documento Procesado (chunks + embeddings)
    |
    ├─> Generación de embeddings (MiniLM-L6-v2)
    |   - 384 dimensiones
    |   - Representación semántica del documento
    |
    ├─> KMeans Clustering
    |   - Agrupa documentos por similitud semántica
    |   - Genera clusters temáticos
    |
    ├─> BERTopic (UMAP + HDBSCAN)
    |   - Detecta tópicos latentes automáticamente
    |   - Genera descripciones interpretables
    |
    └─> KeyBERT
        - Extrae keywords más relevantes
        - Calcula importancia de cada término
```

#### 3.2 KMeans: Clustering Temático

**Propósito:** Agrupar documentos similares automáticamente para descubrir patrones temáticos.

**Problema que resuelve:**
- Un docente sube 100 documentos PDF sin organización
- KMeans descubre automáticamente que 25 hablan de "Multiplicación", 18 de "Fracciones", 32 de "Álgebra", etc.
- Organiza la biblioteca de forma inteligente sin intervención manual

**Funcionamiento:**

```
Entrada: 100 documentos con sus embeddings (100 x 384 matriz)

Paso 1: Inicialización
- Define 10 clusters iniciales
- Asigna centros (puntos medios) aleatoriamente

Paso 2: Iteración
- Calcula distancia euclidiana de cada documento al centroide
- Asigna cada documento al cluster más cercano
- Recalcula centros basado en documentos asignados
- Repite hasta convergencia

Salida: 
Cluster 1: [doc_001, doc_034, doc_067] <- "Matemáticas Básica"
Cluster 2: [doc_012, doc_045, doc_089] <- "Pedagogía Avanzada"
Cluster 3: [doc_002, doc_056, doc_123] <- "Ciencias Naturales"
...
```

**Parámetros clave:**

| Parámetro | Valor | Justificación |
|-----------|-------|---------------|
| n_clusters | 10 | Balance entre granularidad y cobertura |
| max_iter | 300 | Suficientes iteraciones para convergencia |
| random_state | 42 | Reproducibilidad de resultados |
| n_init | 10 | Múltiples inicializaciones para evitar óptimos locales |
| metric | 'euclidean' | Distancia estándar en espacios de embedding |

**Ventajas de KMeans para este proyecto:**

1. **Escalabilidad**: Puede procesar miles de documentos eficientemente
2. **Interpretabilidad**: Los clusters representan temas coherentes
3. **Velocidad**: Convergencia rápida en espacios de alta dimensión
4. **Determinismo**: Resultados reproducibles

**Limitaciones y soluciones:**

| Limitación | Impacto | Solución |
|-----------|--------|----------|
| Requiere especificar k | Número de clusters fijo | Usar BERTopic que lo determina automáticamente |
| Sensible a valores atípicos | Clusters imbalanceados | Normalizar embeddings, usar StandardScaler |
| Geometría esférica | Asume clusters esféricos | Complementar con HDBSCAN |
| No maneja clusters vacíos | Ineficiencia | Re-inicializar centros |

**Caso de uso típico:**

```
Entrada: 50 documentos de matemáticas sin clasificar
         - 15 sobre multiplicación
         - 18 sobre fracciones
         - 12 sobre álgebra
         - 5 sobre geometría

KMeans con k=4:
Cluster 0: [doc_001, doc_015, doc_032, doc_041, ...] (15 docs)
           Centroide: [0.12, -0.34, 0.56, ...] (promedio de embeddings)
           Tema inferido: Multiplicación

Cluster 1: [doc_002, doc_018, doc_045, doc_067, ...] (18 docs)
           Centroide: [0.45, 0.23, -0.12, ...] 
           Tema inferido: Fracciones

Cluster 2: [doc_003, doc_025, doc_056, ...] (12 docs)
           Tema inferido: Álgebra

Cluster 3: [doc_005, doc_019, ...] (5 docs)
           Tema inferido: Geometría

Resultado: Organización automática sin intervención
```

#### 3.3 BERTopic: Detección de Tópicos Latentes

**Propósito:** Descubrir tópicos implícitos en los documentos de forma automática.

**Diferencia fundamental con KMeans:**
- KMeans: "¿A qué grupo conocido pertenece este documento?"
- BERTopic: "¿Cuáles son los tópicos ocultos en estos documentos?"

**Problema que resuelve:**

Un docente carga documentos sobre "Educación" pero los tópicos reales son más específicos:
- "Metodologías de enseñanza activa"
- "Evaluación formativa"
- "Inteligencia emocional en el aula"

BERTopic los descubre automáticamente sin necesidad de definir categorías previas.

**Arquitectura de BERTopic:**

```
Paso 1: Embeddings (sentence-transformers)
   Documentos -> Vectores de 384 dimensiones
   
Paso 2: Reducción de dimensionalidad (UMAP)
   384 dimensiones -> 2-10 dimensiones
   Razón: Mejorar clustering, visualización, eficiencia
   
Paso 3: Clustering (HDBSCAN)
   Agrupa documentos en el espacio reducido
   Genera clusters de densidad variable
   Identifica outliers automáticamente
   
Paso 4: Etiquetado de Tópicos (c-TF-IDF)
   Extrae palabras más representativas por cluster
   Genera descripción automática: "Técnicas de enseñanza"
```

**¿Por qué esta arquitectura es superior a KMeans?**

| Aspecto | KMeans | BERTopic |
|--------|--------|----------|
| Números de clusters | Debe especificarse | Se detectan automáticamente |
| Forma de clusters | Esférica | Arbirtaria (densidad variable) |
| Clusters vacios | Posibles | Eliminados |
| Outliers | Fuerza a asignar | Detecta automáticamente |
| Interpretabilidad | Requiere validación | Genera descripciones automáticas |

**Algoritmo HDBSCAN (Hierarchical DBSCAN):**

```
Problema: "Necesitamos agrupar documentos pero no sabemos cuántos tópicos hay"

Solución HDBSCAN:

1. Construir árbol jerárquico de densidades
   - Comienza con cada documento como cluster individual
   - Fusiona clusters cercanos iterativamente
   
2. Extraer clustering óptimo del árbol
   - No necesita especificar k
   - Automáticamente encuentra número de clusters
   
3. Identificar outliers
   - Documentos demasiado alejados son "ruido"
   
Ventaja: Descubre estructura natural en los datos

Ejemplo:
   Documentos [doc_1, doc_2, doc_3, ... doc_100]
   
   HDBSCAN detecta:
   - Cluster densidad-alta (35 docs): Tópico "Multiplicación"
   - Cluster densidad-media (28 docs): Tópico "Fracciones"  
   - Cluster densidad-baja (20 docs): Tópico "Evaluación"
   - Outliers (17 docs): Sin tópico claro, requieren revisión
```

**Ejemplo de salida de BERTopic:**

```
Tópico 0 (35 documentos):
  Palabras clave: ['multiplicación', 'tablas', 'operación', 'aritmética', 'producto']
  Descripción generada: "Operaciones de multiplicación"
  Documentos ejemplo: [doc_001, doc_015, doc_032, ...]

Tópico 1 (28 documentos):
  Palabras clave: ['fracciones', 'numerador', 'denominador', 'proporciones', 'división']
  Descripción generada: "Conceptos de fracciones"
  Documentos ejemplo: [doc_008, doc_042, doc_071, ...]

Tópico 2 (20 documentos):
  Palabras clave: ['evaluación', 'rúbrica', 'calificación', 'desempeño', 'criterios']
  Descripción generada: "Métodos de evaluación"
  Documentos ejemplo: [doc_005, doc_119, doc_156, ...]

Outliers (-1): [doc_088, doc_093, doc_105, ...] 
  Sin tópico claro, requieren revisión manual
```

#### 3.4 KeyBERT: Extracción de Keywords

**Propósito:** Extraer automáticamente las palabras más importantes de cada documento.

**Funcionamiento:**

```
Documento: "La multiplicación es una operación fundamental en matemáticas..."

KeyBERT extrae:
1. Genera embeddings de la oración completa
2. Genera embeddings de palabras individuales (candidatos)
3. Calcula similitud coseno entre documento y palabras
4. Ordena por similitud (palabras más representativas primero)

Resultado:
[('multiplicación', 0.95),
 ('operación', 0.87),
 ('matemáticas', 0.84),
 ('fundamental', 0.76),
 ('números', 0.71)]
```

**Ventajas:**

- Basado en embeddings semánticos (entiende significado)
- No requiere diccionarios o stop-words personalizados
- Captura contexto del documento
- Múltiples idiomas soportados

**Caso de uso:**

```
Docente busca: "multiplicación"

Sistema busca documentos con keywords ["multiplicación", "operación", "producto", ...]
Encuentra 45 documentos relevantes
Los ordena por relevancia (score de similitud)
```

#### 3.5 Pipeline Completo de Clasificación

**Flujo de procesamiento de un documento:**

```
1. ENTRADA: Documento PDF "Guía de Multiplicación para Primaria"

2. EXTRACCIÓN DE EMBEDDINGS
   - Documento completo -> vector de 384 dimensiones
   - Ejemplo: [0.23, -0.45, 0.12, ..., 0.56]

3. KMEANS CLUSTERING
   - Calcula distancia a 10 centroides
   - Asigna a cluster más cercano: Cluster 2
   - Score de distancia: 0.34 (qué tan cerca está)

4. BERTOPIC ANALYSIS
   - Reduce 384 dims a 5 dims (UMAP)
   - Detecta densidad (HDBSCAN)
   - Asigna tópico: "Operaciones Básicas de Matemáticas"
   - Confianza: 0.92

5. KEYBERT EXTRACTION
   - Extrae top 8 keywords:
     1. multiplicación (0.98)
     2. tablas (0.91)
     3. operación (0.87)
     4. aritmética (0.84)
     5. producto (0.81)
     6. repetición (0.78)
     7. número (0.76)
     8. cálculo (0.73)

6. CONSOLIDACIÓN DE RESULTADOS
   Primary Category: "Matemáticas" (91% confianza)
   Secondary: ["Pedagogía" (0.78), "Educación Primaria" (0.72)]
   Keywords: [multiplicación, tablas, operación, ...]
   Topics: ["Operaciones Básicas", "Enseñanza de Aritmética"]
   Cluster: 2 (con 156 documentos similares)

7. SALIDA JSON
{
  "document_id": "doc_789",
  "primary_category": "Matemáticas",
  "confidence": 0.91,
  "secondary_categories": [
    {"name": "Pedagogía", "confidence": 0.78},
    {"name": "Educación Primaria", "confidence": 0.72}
  ],
  "keywords": [
    {"term": "multiplicación", "score": 0.98},
    {"term": "tablas", "score": 0.91},
    ...
  ],
  "topics": ["Operaciones Básicas", "Enseñanza de Aritmética"],
  "cluster_id": 2,
  "cluster_size": 156,
  "cluster_coherence": 0.87
}
```

#### 3.6 Evaluación de Calidad de Clustering

**Métrica 1: Silhouette Score**

```
Mide qué tan bien separados están los clusters:
- Rango: -1 a 1
- Valor alto (0.7+): Clusters bien definidos
- Valor bajo (<0.3): Clusters débiles

Fórmula para cada documento:
  silhouette = (b - a) / max(a, b)
  
  donde:
  a = distancia promedio a documentos del mismo cluster
  b = distancia promedio a documentos del cluster más cercano

Interpretación:
  0.8-1.0: Clustering excelente
  0.5-0.8: Clustering bueno
  0.2-0.5: Clustering débil
  <0.2: Clustering muy débil
```

**Métrica 2: Calinski-Harabasz Index**

```
Relación entre dispersión entre-clusters y dentro-clusters:

CalinskiHarabasz = (SS_between / SS_within) * ((n - k) / (k - 1))

Interpretación:
- Valores altos (>100): Clustering fuerte
- Valores medios (50-100): Clustering moderado
- Valores bajos (<50): Clustering débil
```

**Monitoreo continuo:**

```
Cada vez que se ingesta un nuevo documento:

1. Calcular silhouette score
2. Si score < 0.5: Alertar que clustering puede mejorar
3. Recalcular clustering si:
   - 20% nuevos documentos añadidos
   - Silhouette score cae < 0.4
   - Cambios significativos en datos

Logs:
[INFO] Ingestion complete. Silhouette: 0.76 (GOOD)
[WARN] Ingestion complete. Silhouette: 0.42 (NEEDS RECALCULATION)
[INFO] Clustering recalculated. New silhouette: 0.81
```

#### 3.7 Endpoint de Clasificación

**Endpoint:** `POST /api/v1/classification/classify`

**Request:**
```json
{
  "document_id": "doc_789xyz",
  "embedding": [0.123, -0.456, ...],  // 384 dimensiones
  "text_sample": "La multiplicación es una operación...",
  "apply_kmeans": true,
  "apply_bertopic": true,
  "apply_keybert": true
}
```

**Response:**
```json
{
  "document_id": "doc_789xyz",
  "primary_category": "Matemáticas",
  "primary_confidence": 0.92,
  "secondary_categories": [
    {"name": "Pedagogía", "confidence": 0.78},
    {"name": "Educación Primaria", "confidence": 0.72}
  ],
  "keywords": [
    {"keyword": "multiplicación", "score": 0.98},
    {"keyword": "tablas", "score": 0.91},
    {"keyword": "aritmética", "score": 0.87},
    {"keyword": "operación", "score": 0.84},
    {"keyword": "producto", "score": 0.81},
    {"keyword": "repetición", "score": 0.78},
    {"keyword": "número", "score": 0.76},
    {"keyword": "cálculo", "score": 0.73}
  ],
  "topics": [
    {
      "id": 0,
      "name": "Operaciones Básicas",
      "probability": 0.88
    },
    {
      "id": 1,
      "name": "Enseñanza de Aritmética",
      "probability": 0.76}
  ],
  "clustering": {
    "cluster_id": 2,
    "cluster_label": "Matemáticas Primaria",
    "cluster_size": 156,
    "cluster_coherence": 0.87,
    "silhouette_score": 0.76
  },
  "processing_time_ms": 245
}
```

#### 3.8 Métricas y Monitoreo de ML

**Métricas importantes:**

1. **Precisión de Clasificación**: % documentos bien categorizados
   - Objetivo: >85%
   - Validación: Manual de muestra estratificada

2. **Cohesión de Clusters**: ¿Qué tan similares son los documentos en cada cluster?
   - Métrica: Silhouette Score
   - Objetivo: >0.7

3. **Separación entre Clusters**: ¿Qué tan diferentes son los clusters?
   - Métrica: Calinski-Harabasz Index
   - Objetivo: >100

4. **Estabilidad**: ¿Los resultados son reproducibles?
   - Métrica: Concordancia entre ejecuciones
   - Objetivo: >95%

5. **Relevancia de Keywords**: ¿Los keywords son realmente importantes?
   - Métrica: Validación manual
   - Objetivo: >80% de keywords relevantes

**Dashboard de monitoreo:**

```
=== CLUSTERING QUALITY METRICS ===
Silhouette Score:        0.76 [GOOD]
Calinski-Harabasz:       142.3 [GOOD]
Davies-Bouldin Index:    0.89 [GOOD]

=== CLASSIFICATION ACCURACY ===
Primary Category:        92% [EXCELLENT]
Secondary Categories:    87% [GOOD]
Keyword Relevance:       94% [EXCELLENT]

=== CLUSTERING STATS ===
Total Documents:         1,847
Total Clusters:          12
Avg Cluster Size:        154
Min Cluster Size:        23
Max Cluster Size:        412
Outliers (Unassigned):   47 (2.5%)

=== RECENT PERFORMANCE ===
Last 100 Documents:      Silhouette 0.78
Last 7 Days:            Silhouette 0.75
Last 30 Days:           Silhouette 0.73

=== RECOMMENDATIONS ===
None - Clustering performance is optimal
```

### 4. Servicio de Chat y Búsqueda Semántica

Centro de interacción conversacional con la plataforma.

**Flujo:**
1. Recepción de consulta en lenguaje natural
2. Extracción de filtros (nivel, tema, tipo de documento)
3. Generación de embedding de la consulta
4. Búsqueda en pgvector (similitud coseno, top-5)
5. Construcción de contexto (máximo 2000 tokens)
6. Envío a DeepSeek para generación de respuesta
7. Respuesta con referencias exactas a documentos y páginas
8. Almacenamiento de historial conversacional

**Endpoints:**
- `POST /api/v1/chat/start`: Iniciar sesión
- `POST /api/v1/chat/query`: Enviar consulta
- `GET /api/v1/chat/history/{session_id}`: Obtener historial
- `POST /api/v1/chat/end/{session_id}`: Cerrar sesión

### 5. Servicio de Enriquecimiento Web

Búsqueda automática de recursos externos cuando recursos locales son insuficientes.

**Activación:** Cuando similitud máxima < 0.60

**Proceso:**
1. Búsqueda paralela en DuckDuckGo, SearXNG, ERIC
2. Filtrado por dominio educativo (.edu, .org), autoría, recencia, licencia
3. Scoring de confiabilidad (mínimo 60 puntos)
4. Descarga de top-3 documentos (máximo 50MB cada uno)
5. Validación de formato y escaneo básico
6. Reingesta mediante Servicio de Ingesta
7. Notificación al usuario

**Límites:**
- Máximo 5 búsquedas por usuario por día
- Cooldown de 5 minutos entre búsquedas
- Máximo 10 documentos nuevos por búsqueda

**Endpoint:** `POST /api/v1/enrichment/search-external`

---

## Especificación de API

### Autenticación

```
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
```

### Gestión de Documentos

```
POST /api/v1/documents/upload
GET /api/v1/documents
GET /api/v1/documents/{document_id}
PATCH /api/v1/documents/{document_id}
DELETE /api/v1/documents/{document_id}
GET /api/v1/documents/{document_id}/similar
```

### Chat y Búsqueda

```
POST /api/v1/chat/start
POST /api/v1/chat/query
GET /api/v1/chat/history/{session_id}
POST /api/v1/chat/end/{session_id}
GET /api/v1/search?q=<query>&limit=<n>
GET /api/v1/search/suggestions?q=<partial>
```

### Generación de Materiales

```
POST /api/v1/generation/study-guide
POST /api/v1/generation/lesson-plan
POST /api/v1/generation/export
GET /api/v1/generation/{generation_id}/status
```

### Organización y Recomendaciones

```
GET /api/v1/organization/clusters
GET /api/v1/organization/concept-map
GET /api/v1/organization/recommendations
```

### Administración

```
GET /api/v1/admin/statistics
GET /api/v1/admin/documents/pending-review
POST /api/v1/admin/documents/{document_id}/approve
POST /api/v1/admin/documents/{document_id}/reject
GET /api/v1/admin/audit-logs
```

---

## Integración con Files Service

El almacenamiento de archivos se gestiona mediante el Files Service de la organización, un microservicio implementado en NestJS con arquitectura hexagonal.

### Arquitectura del Files Service

El Files Service implementa Clean Architecture con:
- Puertos de entrada (IFilesService): Define las operaciones
- Puertos de salida: IFilesRepository (persistencia) e IFilesStorage (S3)
- Casos de uso independientes para cada operación
- Inyección de dependencias vía NestJS

### Operaciones Disponibles

```
uploadFile(userId: string, file: Express.Multer.File): Promise<File>
getFileById(id: string): Promise<File | null>
getAllFiles(): Promise<File[]>
getFilesByUserId(userId: string): Promise<File[]>
deleteFile(id: string): Promise<void>
```

### Modelo de Datos (File Entity)

```
{
  id: string                                    // UUID único
  filename: string                              // Nombre original del archivo
  s3Key: string                                 // Clave/ruta en S3
  userId: string                                // Usuario propietario
  status: 'PENDING' | 'PROCESSING' | 'COMPLETED' | 'FAILED'
  createdAt: Date                               // Timestamp de creación
}
```

### Protocolo de Comunicación

**Subida de archivo (REST HTTP):**
```
POST /files/upload
Content-Type: multipart/form-data
Headers: Authorization: Bearer {jwt_token}

{
  file: <binary>,
  userId: string
}

Response:
{
  id: "uuid-xxx",
  filename: "documento.pdf",
  s3Key: "uploads/user-123/documento-abc.pdf",
  userId: "user-123",
  status: "PENDING",
  createdAt: "2024-01-15T10:30:00Z"
}
```

**Obtener archivo por ID:**
```
GET /files/{file_id}
Headers: Authorization: Bearer {jwt_token}

Response:
{
  id: "uuid-xxx",
  filename: "documento.pdf",
  s3Key: "uploads/user-123/documento-abc.pdf",
  userId: "user-123",
  status: "COMPLETED",
  createdAt: "2024-01-15T10:30:00Z"
}
```

**Obtener archivos de usuario:**
```
GET /files/user/{user_id}
Headers: Authorization: Bearer {jwt_token}

Response: File[]
```

**Eliminar archivo:**
```
DELETE /files/{file_id}
Headers: Authorization: Bearer {jwt_token}

Response: 204 No Content
```

### Repositorio

Files Service: https://github.com/teaching-assistant-microservices/files-service

Implementación: NestJS + TypeScript + Arquitectura Hexagonal

### Flujo de Ingesta con Files Service

1. Usuario sube archivo mediante interfaz web
2. Archivo enviado a Files Service REST API
3. Files Service valida tipo y tamaño
4. Archivo almacenado en S3, metadatos en BD
5. Files Service retorna File con id y s3Key
6. Core IA recibe notificación (webhook o polling)
7. Core IA obtiene archivo via GET /files/{file_id}
8. Extracción de texto (PyMuPDF/Tesseract)
9. Limpieza, chunking, generación de embeddings
10. Indexación en pgvector
11. Metadatos vinculados al file_id del Files Service
12. Archivo permanece en S3 para descargas posteriores

### Integración en el Core IA

**En el Servicio de Ingesta:**

```python
# Obtener archivo desde Files Service
file_metadata = await files_service_client.get_file_by_id(file_id)

# Descargar contenido desde S3
file_content = await s3_client.get_object(file_metadata.s3Key)

# Procesar documento
extracted_text = extract_text(file_content)
chunks = chunk_text(extracted_text)
embeddings = generate_embeddings(chunks)

# Indexar en pgvector
index_in_pgvector(file_id, chunks, embeddings)

# Guardar referencia en BD
save_document_metadata(
    file_id=file_metadata.id,
    filename=file_metadata.filename,
    user_id=file_metadata.userId,
    s3_key=file_metadata.s3Key,
    chunks_count=len(chunks),
    status='PROCESSED'
)
```

### Transiciones de Estado

**Flujo de estados del documento:**

```
PENDING (archivo subido, esperando procesamiento)
   |
   v
PROCESSING (Core IA procesando)
   |
   ├─> COMPLETED (éxito, indexado)
   |
   └─> FAILED (error en procesamiento)
```

### Manejo de Errores

**Errores esperados:**

- 404 Not Found: Archivo no existe
- 403 Forbidden: Usuario no tiene permisos
- 413 Payload Too Large: Archivo excede límite
- 415 Unsupported Media Type: Tipo de archivo no permitido
- 500 Internal Server Error: Error del servidor

**Core IA debe manejar estos errores y notificar al usuario**

---

## Requisitos Técnicos

### Hardware Mínimo

- CPU: 4 cores (8 recomendados)
- RAM: 8GB (16GB recomendados)
- Disco: 50GB SSD
- Bandwidth: 100 Mbps

### Software Requerido

- Python 3.11+
- PostgreSQL 15+
- Tesseract OCR 5.0+
- Redis 7.0+ (opcional, para caching)
- Docker 24.0+ (opcional)

### Dependencias Principales

| Componente | Tecnología | Versión |
|-----------|-----------|---------|
| Framework API | FastAPI | 0.104+ |
| Base de datos | PostgreSQL | 15+ |
| Vectores | pgvector | 0.1.8+ |
| Embeddings | sentence-transformers | 2.2+ |
| Clustering | scikit-learn | 1.3+ |
| Modelado de tópicos | BERTopic | 0.15+ |
| Extracción keywords | KeyBERT | 0.8+ |
| LLM | DeepSeek API | - |
| OCR | Tesseract | 5.0+ |
| ORM | SQLAlchemy | 2.0+ |
| Validación | Pydantic | 2.4+ |

---

## Seguridad

### Autenticación y Autorización

- Autenticación mediante JWT con tokens de corta duración (24 horas)
- Refresh tokens con expiración extendida (7 días)
- Roles basados en acceso (profesor, administrador, auditor)
- Validación de permisos en cada endpoint

### Protección de Datos

- Validación de inputs conforme a OWASP
- Sanitización de documentos antes del procesamiento
- Comunicación TCP encriptada entre servicios internos
- Rate limiting: 100 requests/minuto por usuario
- Límites de tamaño en uploads (máximo 50MB)
- Validación de tipos de archivo permitidos

### Auditoría

- Logs de auditoría para operaciones críticas (upload, eliminación, generación)
- Trazabilidad mediante request IDs únicos
- Registro de acceso de usuarios
- Almacenamiento de logs con retención configurable

### Cumplimiento

- Protección CORS configurable
- Encriptación en tránsito (TLS 1.3)
- Cumplimiento básico de GDPR (derecho a olvido, portabilidad)
- Protección contra inyección SQL mediante ORM

---

## Monitoreo y Observabilidad

### Métricas Clave

**Rendimiento:**
- Latencia de búsqueda semántica: objetivo <500ms (p95)
- Tiempo de respuesta del chat: objetivo <2 segundos
- Tiempo de procesamiento de documentos: 1-5 minutos según tamaño

**Calidad:**
- Precisión de clasificación: objetivo >85%
- Tasa de duplicados evitados: objetivo >95%
- Relevancia de búsquedas: objetivo >80%

**Disponibilidad:**
- Uptime del servicio: objetivo >99.5%
- Recuperación ante fallos: objetivo <5 minutos

### Monitoreo

- Logs estructurados con niveles DEBUG, INFO, WARNING, ERROR
- Métricas de Prometheus para monitoreo en tiempo real
- Health checks en endpoints `/api/v1/health`
- Alertas automáticas ante anomalías críticas
- Dashboard de estadísticas de uso

### Health Checks

```
GET /api/v1/health
GET /api/v1/health/db
GET /api/v1/health/external-services
```

---

## Modelo de Datos

### Documento

```
id: UUID
title: String
author: String
file_id: String (referencia a Files Service)
content_hash: String (SHA-256)
embedding: Vector (384 dimensiones)
chunks: Integer
primary_category: String
secondary_categories: List[String]
keywords: List[String]
created_at: DateTime
updated_at: DateTime
status: Enum (processed, pending, rejected)
```

### Sesión de Chat

```
id: UUID
user_id: UUID
title: String
created_at: DateTime
updated_at: DateTime
status: Enum (active, closed)
```

### Mensaje de Chat

```
id: UUID
session_id: UUID
turn: Integer
role: Enum (user, assistant)
message: String
embedding: Vector (384 dimensiones, opcional)
sources: List[DocumentSource]
confidence: Float
timestamp: DateTime
```

### Generación de Material

```
id: UUID
user_id: UUID
type: Enum (study_guide, lesson_plan)
topic: String
level: String
status: Enum (generating, completed, failed)
content: String
metadata: Dict
created_at: DateTime
completed_at: DateTime
```

---

## Flujos de Trabajo Típicos

### Flujo 1: Ingesta y Procesamiento

1. Usuario sube documento a través de interfaz
2. Archivo enviado a Files Service (TCP)
3. Core IA inicia ingesta con file_id
4. Extracción de texto (PyMuPDF/OCR)
5. Limpieza y chunking
6. Generación de embeddings
7. Verificación de duplicados
8. Indexación en pgvector
9. Clasificación automática
10. Documento disponible para búsqueda

### Flujo 2: Consulta Conversacional

1. Usuario inicia sesión de chat
2. Usuario envía consulta: "Ejercicios de multiplicación"
3. Búsqueda semántica en pgvector
4. Si score > 0.60: generar respuesta
5. Si score < 0.60: activar búsqueda web
6. DeepSeek genera respuesta contextualizada
7. Respuesta con referencias exactas
8. Historial almacenado en sesión

### Flujo 3: Generación de Plan de Clase

1. Usuario solicita plan: tema, nivel, duración
2. Búsqueda de recursos relacionados
3. Selección de documentos más relevantes
4. Extracción de información pedagógica
5. Generación de estructura (objetivos, contenidos, evaluación)
6. Compilación de documento
7. Exportación a DOCX/PDF
8. Usuario descarga archivo listo para usar

### Flujo 4: Enriquecimiento Web

1. Usuario realiza consulta con score bajo
2. Activación de búsqueda web externa
3. Búsqueda en DuckDuckGo, SearXNG, ERIC
4. Scoring de confiabilidad
5. Descarga de top-3 documentos
6. Reingesta mediante servicio de ingesta
7. Nueva búsqueda con documentos actualizados
8. Respuesta completa al usuario

---

## Comunicación Inter-Servicios

### Arquitectura de Comunicación

Los servicios se comunican mediante diferentes protocolos:

- **Core IA <-> Files Service**: HTTP REST (NestJS)
- **Core IA <-> Base de Datos**: PostgreSQL (puerto 5432)
- **Core IA <-> APIs Externas**: HTTP/HTTPS

### Interacción Core IA - Files Service

**Ciclo de vida de un documento:**

1. **Upload**: Cliente sube a Files Service (multipart/form-data)
2. **Respuesta**: Files Service retorna File con id y s3Key
3. **Procesamiento**: Core IA obtiene archivo via GET /files/{file_id}
4. **Indexación**: Core IA vincula embeddings a file_id
5. **Descarga**: Usuario accede via Files Service

**Endpoints utilizados:**

```
GET /files/{file_id}          # Obtener metadatos y acceso a archivo
GET /files/user/{user_id}     # Listar archivos del usuario
DELETE /files/{file_id}       # Eliminar archivo y referencias
```

**Ejemplo de integración en Servicio de Ingesta:**

```
1. Recibir file_id del cliente
2. GET /files/{file_id} -> obtener metadatos
3. Descargar desde S3 usando s3Key
4. Procesar: extracción, chunking, embeddings
5. Guardar en pgvector con referencia a file_id
6. Actualizar estado en BD local
```

---

## Conclusión

La Plataforma Educativa Inteligente propone una solución integral para optimizar la gestión de recursos pedagógicos mediante inteligencia artificial. Su arquitectura modular basada en microservicios especializados, comunicación HTTP REST con Files Service, y énfasis en seguridad y monitoreo, garantiza una solución escalable y confiable para instituciones educativas.
