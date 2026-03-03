# ⚖️ Checklist de Gobernanza y Licencias — PPE Detection

**Proyecto:** Detección de EPP en obra de construcción  
**Responsable:** [Tu Nombre]  
**Fecha:** Mayo 2025  
**Versión:** 1.0

---

## 1. Privacidad y Consentimiento

| Ítem | Estado | Notas |
|------|--------|-------|
| ¿El dataset contiene imágenes de personas identificables? | ⚠️ Posible | El dataset público de Roboflow puede contener rostros reconocibles |
| ¿Se cuenta con consentimiento de las personas fotografiadas? | ✅ Delegado | El dataset es CC BY 4.0; el colector original asume responsabilidad de consentimiento |
| ¿Se aplica anonimización (blur de rostros)? | ❌ No aplicado | **Recomendado** antes de uso en producción con imágenes propias |
| ¿El modelo procesa datos en tiempo real de cámaras? | ⚠️ Depende | En deploy real, se requiere aviso visible a trabajadores ("zona bajo videovigilancia") |
| ¿Cumple con LFPDPPP (México)? | ✅ Para uso académico | Para uso comercial/productivo se requiere análisis legal específico |

**Acción requerida antes de producción:** Implementar blur automático de rostros en el pipeline de inferencia. Colocar señalización visible en zonas monitoreadas.

---

## 2. Minimización de Datos

| Ítem | Estado | Notas |
|------|--------|-------|
| ¿Se recopilan solo los datos necesarios? | ✅ | El modelo procesa frames; no se almacenan videos completos en el diseño propuesto |
| ¿Existe política de retención de imágenes? | ⚠️ Pendiente | Para producción: definir retención máxima (ej. 72 horas para alertas, eliminar después) |
| ¿Se almacenan datos biométricos? | ❌ No | El modelo detecta EPP, no identifica personas |
| ¿Las predicciones se asocian a individuos? | ❌ No | Las alertas se asocian a zonas/turnos, no a trabajadores específicos |

---

## 3. Declaración de Limitaciones — Cuándo NO usar este modelo

### ❌ Casos de Uso No Recomendados

1. **Identificación de individuos**: El modelo no fue diseñado ni validado para identificar trabajadores específicos. No usar para vigilancia nominativa.

2. **Sustitución de supervisor humano**: El modelo es una herramienta de apoyo. La decisión final sobre sanciones, reportes o acciones disciplinarias debe recaer en un supervisor humano.

3. **Condiciones de baja iluminación sin calibración**: El rendimiento cae significativamente en zonas oscuras (mAP50 ~0.54). No desplegar como sistema único de seguridad en obra nocturna sin validación específica.

4. **EPP no incluido en el modelo**: El modelo solo detecta cascos y chalecos. No detecta guantes, botas de seguridad, gafas protectoras, arneses u otro EPP.

5. **Obras con tipos de EPP no estándar**: Cascos de colores inusuales, chalecos sin franjas reflectantes estándar pueden no ser detectados correctamente.

6. **Imágenes de muy baja resolución**: El modelo fue entrenado con imgsz=640. Cámaras con resolución menor a 720p pueden generar más fallos.

7. **Uso como evidencia legal única**: Las predicciones del modelo no constituyen evidencia legal. Deben complementarse con registros de video originales y supervisión humana.

---

## 4. Nota de Riesgos — Falsos Negativos vs. Falsos Positivos

### Análisis de Riesgos Asimétricos

En el contexto de seguridad laboral, los dos tipos de error tienen consecuencias muy distintas:

| Tipo de Error | Descripción | Consecuencia | Gravedad |
|---------------|-------------|--------------|----------|
| **Falso Negativo (FN)** | Trabajador sin EPP NO detectado | El riesgo pasa desapercibido; posible accidente | 🔴 **ALTA** |
| **Falso Positivo (FP)** | EPP detectado cuando no hay / diferente | Alerta falsa; posible fricción operativa | 🟡 **MEDIA** |

### Implicación para Umbral de Decisión

**Recomendación:** En el deploy productivo, priorizar **reducir falsos negativos** (aumentar recall) sobre reducir falsos positivos. Esto implica:

- Usar umbral de confianza más bajo (0.35-0.40 en vez del default 0.50)
- Aceptar más alertas falsas a cambio de no perder incumplimientos reales
- Implementar confirmación multi-frame (solo alertar si el incumplimiento persiste en 3+ frames consecutivos para reducir FP sin perder FN)

### Matriz de Riesgo

```
                    PREDICCIÓN
                 EPP OK    SIN EPP
REALIDAD  EPP OK   TP        FP → Fricción operativa
          SIN EPP  FN        TN
                   ↑
           RIESGO CRÍTICO: trabajador desprotegido no detectado
```

---

## 5. Declaración de Licencia

### Código

```
MIT License

Copyright (c) 2025 [Tu Nombre]

Se concede permiso, de forma gratuita, a cualquier persona que obtenga
una copia de este software y los archivos de documentación asociados,
para utilizar el software sin restricciones, incluyendo sin limitación
los derechos de usar, copiar, modificar, fusionar, publicar, distribuir,
sublicenciar y/o vender copias del software.

EL SOFTWARE SE PROPORCIONA "TAL CUAL", SIN GARANTÍA DE NINGÚN TIPO.
```

### Dataset

**Fuente**: Roboflow Universe — Construction Site Safety  
**Licencia**: Creative Commons Attribution 4.0 International (CC BY 4.0)  
**Requerimiento**: Atribuir a los contribuidores originales en cualquier trabajo derivado.  
**URL**: https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety

### Pesos del Modelo (YOLOv8)

**Framework**: Ultralytics YOLOv8  
**Licencia base**: AGPL-3.0  
**Implicación**: Los pesos derivados de YOLOv8 heredan AGPL-3.0. Para uso comercial en productos cerrados, se requiere licencia comercial de Ultralytics.  
**URL**: https://ultralytics.com/license

---

## 6. Resumen Ejecutivo de Gobernanza

| Dimensión | Estado | Prioridad de Acción |
|-----------|--------|---------------------|
| Privacidad y consentimiento | ✅ OK (académico) / ⚠️ Pendiente (producción) | Alta para deploy |
| Minimización de datos | ✅ Diseño adecuado | Media |
| Limitaciones documentadas | ✅ Completo | — |
| Análisis de riesgos FP/FN | ✅ Completo | — |
| Licencia de código | ✅ MIT | — |
| Derechos de dataset | ✅ CC BY 4.0 atribuido | — |
| Licencia de pesos | ⚠️ AGPL-3.0 (comercial requiere gestión) | Media-Alta |

---

*Este checklist debe revisarse antes de cualquier deploy en entorno productivo real.*
