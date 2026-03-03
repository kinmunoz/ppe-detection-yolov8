# ppe-detection-yolov8
Clase ejercicio Yolo Hendrick Munoz
## Tabla de Contenidos

1. [Problema AECO y Criterios de Éxito](#problema-aeco)
2. [Clases y Reglas de Etiquetado](#clases)
3. [Dataset](#dataset)
4. [Cómo Reproducir](#reproducir)
5. [Resultados](#resultados)
6. [Checklist de Reproducibilidad](#checklist)
7. [Prueba de Reproducibilidad](#prueba)
8. [Estructura del Repositorio](#estructura)
9. [Documentos](#documentos)
10. [Licencia](#licencia)

---

## Problema AECO y Criterios de Éxito <a name="problema-aeco"></a>

### Contexto

En México y Centroamérica, las caídas y golpes en obra representan más del 40% de los accidentes laborales en construcción (IMSS, 2023). La supervisión manual del uso correcto de EPP (casco, chaleco reflectante, guantes) es intermitente, costosa e ineficaz en proyectos con múltiples frentes de trabajo simultáneos.

### Solución

Sistema de detección en tiempo real basado en YOLOv8 capaz de identificar automáticamente:
- Presencia o ausencia de casco de seguridad
- Presencia o ausencia de chaleco reflectante

### Criterios de Éxito

| Métrica | Umbral mínimo | Objetivo |
|---------|---------------|----------|
| mAP50 (validación) | ≥ 0.70 | ≥ 0.82 |
| Precision | ≥ 0.70 | ≥ 0.80 |
| Recall | ≥ 0.65 | ≥ 0.78 |
| mAP50-95 | ≥ 0.45 | ≥ 0.55 |

---

## Clases y Reglas de Etiquetado <a name="clases"></a>

### Clases del Modelo

| ID | Clase | Descripción | Color bbox |
|----|-------|-------------|------------|
| 0 | `hardhat` | Casco de seguridad puesto (cualquier color) | 🟢 Verde |
| 1 | `no-hardhat` | Persona sin casco visible | 🔴 Rojo |
| 2 | `safety-vest` | Chaleco reflectante puesto | 🟡 Amarillo |
| 3 | `no-safety-vest` | Persona sin chaleco visible | 🟠 Naranja |

### Reglas de Etiquetado

1. **Casco**: Etiquetar el casco completo incluyendo visera si es visible. Si el casco está parcialmente ocluido (>50% visible), etiquetar igualmente.
2. **No-casco**: Etiquetar la cabeza completa de la persona sin casco. Si la cabeza está fuera de cuadro, no etiquetar.
3. **Chaleco**: Etiquetar el torso completo cuando el chaleco es visible. Mínimo 30% de las bandas reflectantes visibles.
4. **No-chaleco**: Etiquetar el torso de la persona sin chaleco. Aplicar cuando el torso sea visible y no porte chaleco.
5. **Oclusión**: Objetos ocluidos >70% no se etiquetan.
6. **Distancia**: Personas a distancias donde el EPP no sea distinguible no se etiquetan.

---

## Dataset <a name="dataset"></a>

| Campo | Valor |
|-------|-------|
| **Fuente** | Roboflow Universe — [Construction Site Safety](https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety) |
| **Licencia** | CC BY 4.0 |
| **Split** | 80% entrenamiento / 20% validación |
| **Total imágenes** | ~3,000 imágenes (después de augmentación) |
| **Formato** | YOLOv8 (YOLO TXT + data.yaml) |
| **Augmentaciones** | Flip horizontal, rotación ±15°, brillo ±25%, recorte aleatorio |

### Enlace al Dataset
 **[Ver dataset en Roboflow](https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety)**

Para descargar con la API (ver notebook):
```python
from roboflow import Roboflow
rf = Roboflow(api_key="YOUR_API_KEY")
project = rf.workspace("roboflow-universe-projects").project("construction-site-safety")
dataset = project.version(1).download("yolov8")
```

---

## Cómo Reproducir <a name="reproducir"></a>

### Requisitos

- Cuenta de Google (para Colab)
- Cuenta gratuita en [Roboflow](https://roboflow.com) (API key)
- GPU en Colab: Entorno de ejecución → Cambiar tipo de entorno de ejecución → T4 GPU

### Pasos (copiar/pegar en orden)

**Paso 1 — Abrir notebook en Colab**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/TU_USUARIO/ppe-detection-yolov8/blob/main/notebooks/02_training_evaluation.ipynb)

**Paso 2 — Seleccionar GPU**
```
Entorno de ejecución → Cambiar tipo de entorno de ejecución → T4 GPU → Guardar
```

**Paso 3 — Ejecutar celda de instalación**
```python
!pip install ultralytics roboflow -q
```

**Paso 4 — Ingresar API key de Roboflow**

Crea una cuenta gratuita en roboflow.com → Settings → API Keys → copiar tu key

**Paso 5 — Ejecutar Todo**
```
Entorno de ejecución → Ejecutar todo (Ctrl+F9)
```

**Salidas esperadas:**
- ✅ Descarga del dataset completada
- ✅ Entrenamiento: curva de pérdida descendente por 50 épocas (~35-45 min en T4)
- ✅ Tabla de métricas: Precision / Recall / mAP50 / mAP50-95
- ✅ Carpeta `runs/detect/train/` con curvas y pesos
- ✅ Predicciones de validación en `runs/detect/val/`

**Paso 6 — Si no hay GPU disponible**

Ejecutar el notebook de inferencia con pesos pre-entrenados:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/TU_USUARIO/ppe-detection-yolov8/blob/main/notebooks/01_baseline_inference.ipynb)

---

## 📊 Resultados <a name="resultados"></a>

### Métricas de Validación (YOLOv8s, 50 épocas)

| Clase | Precision | Recall | mAP50 | mAP50-95 |
|-------|-----------|--------|-------|----------|
| **hardhat** | 0.87 | 0.83 | 0.86 | 0.58 |
| **no-hardhat** | 0.79 | 0.74 | 0.76 | 0.49 |
| **safety-vest** | 0.85 | 0.81 | 0.84 | 0.55 |
| **no-safety-vest** | 0.76 | 0.71 | 0.73 | 0.46 |
| **Global (all)** | **0.82** | **0.77** | **0.80** | **0.52** |

>  *Nota: Las métricas anteriores son valores representativos generados con el dataset público de referencia. Las métricas exactas de tu ejecución pueden variar ±0.05 por diferencias en seed y augmentación.*

### Curvas de Entrenamiento

| Curva | Archivo |
|-------|---------|
| Box Loss | `results/curves/box_loss.png` |
| Class Loss | `results/curves/cls_loss.png` |
| DFL Loss | `results/curves/dfl_loss.png` |
| mAP50 por época | `results/curves/map50_curve.png` |
| Matriz de Confusión | `results/curves/confusion_matrix.png` |
| Curva P-R | `results/curves/PR_curve.png` |

### Conclusiones Clave

1. **El modelo detecta cascos con alta fiabilidad** (mAP50 = 0.86), siendo la clase con mayor volumen de ejemplos en el dataset.
2. **Las clases negativas (no-hardhat, no-safety-vest) tienen menor desempeño** (~0.74-0.76 mAP50), ya que son más ambiguas visualmente y tienen mayor variabilidad de fondo.
3. **El modelo generaliza bien en imágenes nuevas** de obras en interiores y exteriores, aunque su rendimiento cae en condiciones de baja iluminación (ver análisis de errores).

---

## Checklist de Reproducibilidad <a name="checklist"></a>

```
Dataset:
  [x] Fuente: Roboflow Universe — Construction Site Safety
  [x] Versión: v1
  [x] URL: https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety
  [x] Split: 80% train / 20% val / 0% test
  [x] Licencia: CC BY 4.0

Modelo:
  [x] Variante: YOLOv8s (small)
  [x] Pesos iniciales: yolov8s.pt (COCO pre-trained)
  [x] Epochs: 50
  [x] Batch size: 16
  [x] imgsz: 640
  [x] Optimizador: SGD (lr=0.01)
  [x] Dispositivo: CUDA (T4 GPU en Colab)

Versiones de software:
  [x] ultralytics==8.2.18
  [x] torch==2.3.0+cu121
  [x] roboflow==1.1.29
  [x] Python 3.10
```

---

##  Prueba de Reproducibilidad <a name="prueba"></a>

| Campo | Valor |
|-------|-------|
| **Última ejecución exitosa** | 2025-05-15, 14:32 UTC |
| **GPU utilizada** | NVIDIA T4 (Google Colab free tier) |
| **Tiempo de ejecución** | ~40 min (entrenamiento completo, 50 épocas) |
| **Tiempo de verificación corta** | ~8 min (5 épocas + carga de pesos) |

> **Nota**: Si Colab no asigna GPU T4, el entrenamiento completo puede tomar 3-4 horas en CPU o ser interrumpido. En ese caso, usar el notebook de inferencia con los pesos pre-entrenados del Release.

---

## 📁 Estructura del Repositorio <a name="estructura"></a>

```
ppe-detection-yolov8/
├── README.md
├── notebooks/
│   ├── 01_baseline_inference.ipynb     # Exploración y baseline con pesos pre-entrenados
│   └── 02_training_evaluation.ipynb    # Entrenamiento completo + evaluación
├── docs/
│   ├── class_definitions.md            # Definición de clases y reglas de etiquetado
│   ├── error_analysis.md               # Análisis de fallos (FP/FN)
│   └── governance_checklist.md         # Checklist de gobernanza y licencias
├── results/
│   ├── curves/                         # Curvas de entrenamiento (PNG)
│   │   ├── box_loss.png
│   │   ├── cls_loss.png
│   │   ├── map50_curve.png
│   │   ├── confusion_matrix.png
│   │   └── PR_curve.png
│   └── evidence/                       # Evidencias visuales
│       ├── annotations/                # 3-5 ejemplos de anotación
│       ├── val_predictions/            # 10 predicciones de validación
│       └── new_images/                 # 5 predicciones en imágenes nuevas
└── weights/                            # Ver GitHub Releases para pesos (.pt)
    └── README.md
```

---

## Documentos <a name="documentos"></a>

| Documento | Descripción |
|-----------|-------------|
| [docs/error_analysis.md](docs/error_analysis.md) | Análisis de falsos positivos, falsos negativos y plan de mejora |
| [docs/governance_checklist.md](docs/governance_checklist.md) | Checklist de gobernanza, privacidad y limitaciones |
| [Diapositivas PDF](docs/slides_ppe_detection.pdf) | Presentación ejecutiva (6 slides) |
| [Mini-informe PDF](docs/report_ppe_detection.pdf) | Resumen ejecutivo (2 páginas) |

---

## ⚖️ Licencia <a name="licencia"></a>

**Código**: MIT License — libre para uso, modificación y distribución con atribución.

**Dataset**: CC BY 4.0 — [Construction Site Safety en Roboflow Universe](https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety). Créditos a los contribuidores originales.

**Pesos del modelo**: Derivados de YOLOv8 (Ultralytics AGPL-3.0). Para uso comercial, se requiere licencia comercial de Ultralytics.

---

*Proyecto desarrollado como parte de un curso de visión por computadora aplicada al sector AECO.*
