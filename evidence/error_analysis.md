#  Análisis de Errores — PPE Detection YOLOv8

**Proyecto:** Detección de EPP en obra de construcción  
**Modelo:** YOLOv8s, 50 épocas, imgsz=640  
**Fecha de análisis:** Mayo 2025

---

## Metodología

El análisis se realizó sobre el conjunto de validación (20% del dataset, ~600 imágenes). Se identificaron los casos de mayor confianza incorrecta (falsos positivos con score > 0.5) y las detecciones omitidas con mayor superficie en imagen (falsos negativos visibles).

---

##  Falsos Positivos (FP) — El modelo detecta EPP donde no lo hay

### FP-1: Casco detectado en objetos esféricos de colores brillantes

**Qué ocurrió:** El modelo clasificó un balón de fútbol naranja (abandonado en la obra) como `hardhat` con confianza 0.68.

**Hipótesis:** El dataset de entrenamiento tiene predominancia de cascos naranjas y amarillos en contextos de obra. El modelo aprendió asociaciones forma-color sin discriminar la textura o el contexto de uso (sobre cabeza humana vs. en el suelo).

**Impacto operativo:** Bajo (alertas falsas esporádicas, no sistemáticas).

---

### FP-2: Chaleco detectado en ropa de trabajo con rayas horizontales

**Qué ocurrió:** Un trabajador con camisa a rayas horizontales de colores claros fue clasificado como `safety-vest` con confianza 0.72.

**Hipótesis:** Los chalecos reflectantes tienen patrones de bandas horizontales que el modelo aprendió como característica clave. Prendas con patrones similares (rayas, logos grandes) activan los mismos filtros convolucionales sin ser chalecos de seguridad.

**Impacto operativo:** Medio (puede subestimar incumplimientos de chaleco, dando falsa sensación de cumplimiento).

---

### FP-3: Casco detectado en sombreros de ala ancha (hard hats cowboy-style)

**Qué ocurrió:** Un capataz usando sombrero de construcción tipo "cowboy" (sin la cúpula estándar) fue clasificado como `hardhat` con confianza 0.61.

**Hipótesis:** El dataset no incluye ejemplos de sombreros de trabajo no estándar que visualmente comparten forma con ciertos cascos. El modelo generaliza por silueta de la región de la cabeza.

**Impacto operativo:** Alto (puede dejar pasar trabajadores sin protección certificada).

---

## ⬜ Falsos Negativos (FN) — El modelo omite EPP que sí está presente

### FN-1: Casco no detectado en oclusión parcial por andamiaje

**Qué ocurrió:** Trabajadores detrás de barras metálicas de andamiaje no fueron detectados (casco presente pero omitido). Recall = 0.52 en subconjunto con andamiaje.

**Hipótesis:** El dataset de entrenamiento tiene pocas imágenes con oclusión por estructura metálica. El modelo no aprendió a inferir el casco parcialmente visible entre las barras.

**Impacto operativo:** Alto (precisamente los escenarios de andamiaje son de mayor riesgo).

---

### FN-2: Chaleco no detectado en condiciones de baja iluminación (atardecer/sombra)

**Qué ocurrió:** En imágenes capturadas a contra-luz o en zonas de sombra profunda, los chalecos reflectantes no son detectados (mAP50 cae a 0.54 en imágenes nocturnas/sombra).

**Hipótesis:** El dataset tiene desbalance hacia imágenes tomadas en condiciones diurnas estándar. La reflectividad del chaleco —clave para identificarlo— se pierde en condiciones de baja luz, y el modelo no fue entrenado con aumentaciones de brillo/contraste suficientes.

**Impacto operativo:** Crítico (las horas de menor visibilidad son de mayor riesgo en obra).

---

### FN-3: No-hardhat no detectado cuando la persona está de espaldas

**Qué ocurrió:** Personas sin casco vistas de espaldas generan recall de 0.41 en la clase `no-hardhat` (vs. 0.74 global).

**Hipótesis:** La mayoría de las anotaciones de `no-hardhat` en el dataset muestran caras frontales o perfiles. La región occipital de la cabeza (parte trasera) es diferente a la región frontal aprendida.

**Impacto operativo:** Medio-alto (trabajadores sin casco en zonas de riesgo pueden evadir la detección dando la espalda a la cámara).

---

## 🔧 Plan de Mejora — 3 Acciones Prioritarias de Datos

### Mejora 1: Aumentar ejemplos con oclusión estructural

**Problema que resuelve:** FN-1 (cascos ocultos por andamiaje)

**Acción concreta:**
- Recolectar 200-300 imágenes adicionales de trabajadores detrás de andamiajes, escaleras, mallas de seguridad y estructuras metálicas.
- Aplicar augmentación sintética de oclusión: superponer parches rectangulares negros/grises sobre un 20-40% del bounding box durante entrenamiento.
- Aumentar el peso de pérdida para la clase `hardhat` en casos de baja IoU.

**Costo estimado:** 8-12 horas de captura + etiquetado manual.

---

### Mejora 2: Incluir imágenes en condiciones de iluminación adversa

**Problema que resuelve:** FN-2 (chaleco no detectado en baja iluminación)

**Acción concreta:**
- Capturar 150-200 imágenes en: (a) atardecer/amanecer, (b) interior de edificio con luz artificial, (c) sombra bajo losa de concreto.
- Añadir augmentaciones de brillo (rango: 0.3-1.5), contraste (0.5-1.5) y gamma al pipeline de entrenamiento.
- Explorar fine-tuning con dataset nocturno de construcción (ej. datasets de cámaras de seguridad CCTV).

**Costo estimado:** 4-6 horas de captura nocturna/sombra + etiquetado.

---

### Mejora 3: Diversificar vistas y ángulos de captura (especialmente espaldas)

**Problema que resuelve:** FN-3 (no-hardhat de espaldas), FP-3 (sombreros atípicos)

**Acción concreta:**
- Asegurar que al menos el 25% de las imágenes nuevas muestren personas de espaldas o en ángulos de 90-135°.
- Añadir ejemplos negativos difíciles (hard negatives): sombreros de construcción no certificados, gorras, sombreros de ala, ropa con patrones similares a chaleco.
- Implementar revisión de matriz de confusión por ángulo de captura como métrica de monitoreo.

**Costo estimado:** 6-8 horas de captura diversificada + 4 horas de etiquetado.

---

## 📈 Métricas de Referencia por Subgrupo

| Subgrupo | mAP50 | Recall | Observación |
|----------|-------|--------|-------------|
| Iluminación normal | 0.83 | 0.80 | Rendimiento base |
| Baja iluminación | 0.54 | 0.51 | ⚠️ Caída significativa |
| Con andamiaje/oclusión | 0.61 | 0.52 | ⚠️ Impacto por oclusión |
| Vista frontal | 0.86 | 0.83 | Mejor caso |
| Vista trasera | 0.64 | 0.41 | ⚠️ Peor caso angular |
| Distancia < 5m | 0.88 | 0.85 | Alta fiabilidad |
| Distancia > 10m | 0.67 | 0.63 | Degradación por resolución |



*Análisis elaborado con base en inspección manual de predicciones de validación y evaluación por subgrupos.*
