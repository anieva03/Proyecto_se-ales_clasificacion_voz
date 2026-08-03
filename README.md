# Análisis y clasificación de voz

## Descripción General

**Análisis y clasificación de voz** es un proyecto de procesado digital de señales orientado al estudio de grabaciones de voz humana y voz generada por inteligencia artificial. A partir de una serie de descriptores acústicos extraídos de cada audio, el sistema aborda tres tareas de clasificación: el sexo del hablante, el origen de la señal (voz humana frente a voz sintética) y el acento (español neutro, andaluz y argentino).

El proyecto integra técnicas de procesado de señal (detección de voz/silencio, extracción de pitch, MFCC y centroide espectral) junto con modelos de aprendizaje automático (regresión logística, SVM y random forest) dentro de un flujo de trabajo reproducible.

---

## Características Principales

* Conversión y normalización de audios (.m4a a .wav, mono, 16 kHz)
* Detección y eliminación de ruido/silencios mediante STE y ZCR
* Extracción de características acústicas: period pitch, MFCC (+ Δ y ΔΔ), centroide espectral global y segmentado
* Construcción automatizada de datasets a partir de carpetas de audio
* Clasificación del sexo del hablante mediante regresión logística
* Detección de voces generadas por IA mediante SVM (kernel RBF) y random forest
* Clasificación del acento mediante regresión logística multinomial
* Evaluación mediante matrices de confusión, curvas ROC y validación cruzada repetida

---

## Arquitectura del Sistema

El sistema se divide en cuatro bloques principales:

### Bloque 1 — Preprocesado

* Conversión de audios de .m4a a .wav (librerías `av` y `tuneR`)
* Homogeneización a señal monofónica y frecuencia de muestreo de 16 kHz
* Detección de voz/silencio mediante ventanas de 20 ms (solapamiento 50%) usando STE y ZCR
* Reconstrucción de la señal limpia descartando los tramos de ruido/silencio

### Bloque 2 — Extracción de características

* Cálculo del period pitch mediante autocorrelación (`period_pitch`)
* Cálculo de coeficientes MFCC (`melfcc()` de `tuneR`) y sus derivadas Δ y ΔΔ (`delta_simple`)
* Cálculo del centroide espectral global y segmentado en 12 tramos temporales
* Construcción del dataset general (`df_carpeta`, `añadir_voz`, `etiquetar_por_nombre`)
* Construcción del dataset de MFCC para detección de IA (`extract_feat_from_wav`, `build_feat_dataframe`)

### Bloque 3 — Análisis exploratorio

* Estadística descriptiva de duración, ZCR, energía RMS, pitch y centroide
* Análisis de correlación entre variables
* Comparativa de distribuciones por sexo y por origen (humano vs IA)

### Bloque 4 — Modelado y validación

* Clasificación del sexo mediante regresión logística (pitch + centroide)
* Clasificación IA vs humano mediante SVM con kernel RBF sobre MFCC (validación cruzada repetida, 5 folds x 2 repeticiones)
* Clasificación IA vs humano mediante random forest sobre centroide espectral (global y segmentado)
* Clasificación del acento mediante regresión logística multinomial (ZCR, energía RMS, duración, pitch)
* Evaluación con matrices de confusión, accuracy, pseudo-R² y curvas ROC

---

## Estructura del Proyecto

```
Analisis-voz/
│
├── preprocesado.R
├── extraccion_caracteristicas.R
├── construccion_dataset.R
├── modelos_clasificacion.R
│
├── voces_df.RData
├── audios/
│   ├── real/
│   └── fake/
│
├── resultados/
│   ├── figuras/
│   └── matrices_confusion/
│
└── README.md
```

---

## Instalación

1. Clona el repositorio:

```bash
git clone https://github.com/diazcas2/Proyecto-An-lisis-de-Se-ales-Grupo-E.git
cd Proyecto-An-lisis-de-Se-ales-Grupo-E
```

2. Instala las dependencias necesarias en R:

```r
install.packages(c("av", "tuneR", "seewave", "caret", "pROC", "e1071", "randomForest", "nnet"))
```

3. Organiza los audios de entrada:

* Coloca las grabaciones originales (.m4a) en la carpeta correspondiente.
* Los audios de voz sintética (Voz IA) deben estar ya en formato .wav.

---

## Uso

### Ejecución por etapas

```r
source("preprocesado.R")
source("extraccion_caracteristicas.R")
source("construccion_dataset.R")
source("modelos_clasificacion.R")
```

### Flujo recomendado

1. Ejecuta el script de preprocesado para limpiar los audios (eliminación de ruido/silencios).
2. Ejecuta la extracción de características para generar el dataset (`voces_df`).
3. Ejecuta el script de modelos para entrenar y evaluar los clasificadores de sexo, origen (IA vs humano) y acento.
4. Consulta las matrices de confusión y curvas ROC generadas en `resultados/`.

**Nota:** El cálculo del pitch mediante autocorrelación requiere señales con un número mínimo de muestras; los audios excesivamente cortos se descartan automáticamente para evitar estimaciones no válidas.

---

## Tecnologías Utilizadas

* R
* `av` y `tuneR` (lectura y conversión de audio)
* `seewave` (extracción de descriptores acústicos)
* `caret` (entrenamiento y validación cruzada)
* `e1071` / `svmRadial` (SVM con kernel RBF)
* `randomForest` (clasificación basada en centroide espectral)
* `pROC` (curvas ROC)
* Regresión logística binaria y multinomial

---

## Posibles Mejoras

* Ampliar la base de datos de audios, especialmente para la clasificación de acento
* Incorporar más variedad de hablantes por acento para reducir el sobreajuste a rasgos individuales
* Probar nuevos descriptores acústicos y modelos más avanzados (p. ej. redes neuronales)
* Evaluar la generalización del detector de IA frente a nuevos generadores de voz sintética
* Aplicación del sistema a tareas de seguridad y verificación de identidad
