# Predicción de Convenios Educativos en ATENEA
## Machine Learning para Política Pública

Identificación de convenios con Instituciones de Educación Superior (IES) en la
contratación pública de ATENEA, a partir de datos de SECOP II.

**Pregunta central:** ¿qué características de los contratos predicen que se trate de un
convenio educativo con una IES?

---

## Datos

- **Fuente:** SECOP II (datos abiertos, https://datos.gov.co). Consolidado en
  `SECOP_ATENEA_TOTAL-17.xlsx`, hoja `SECOP_ATENEA_SEL`.
- **Unidad de observación:** contrato público.
- **Periodo:** 2022–2026.
- **Tamaño:** 2.219 contratos y 86 variables (2.191 tras filtrar valores y plazos válidos).
- **Variable objetivo:** `es_ies` (1 = convenio con IES). Se construye por palabras clave
  en la descripción del proceso o el proveedor adjudicado.

> **Advertencia sobre la etiqueta:** como ATENEA es una agencia de educación superior,
> términos como "superior", "educación" o "institución" aparecen en muchos contratos por
> el nombre y mandato de la propia entidad, no porque sean convenios con una IES. Con la
> definición por palabras clave, ~42% de los contratos quedan marcados como IES. Una
> definición más estricta (que el proveedor sea realmente una universidad/IES) marca solo
> ~10% y suma cerca de 2,4 billones de pesos. Los resultados deben leerse con esto
> presente; conviene que un experto valide una muestra de la clasificación.

---

## Metodología

- **Modelos:** Random Forest y Regresión Logística.
- **Predictoras:** valor (log), tamaño del contrato, plazo, modalidad, estado y
  antigüedad. Las columnas de texto que definen la etiqueta se excluyen a propósito, para
  evitar que el modelo "haga trampa".
- **Desbalance de clase:** se aplica re-muestreo con `ROSE` (`ovun.sample`) sobre el
  entrenamiento y se ajusta el **umbral de decisión** a partir de la curva ROC (índice de
  Youden). La base de prueba nunca se toca.
- **Evaluación:** matriz de confusión, precision / recall / F1, AUC-ROC, Kappa y balanced
  accuracy, más validación cruzada 5-fold. Con clases desbalanceadas no basta con el
  Accuracy: se vigilan Kappa, especificidad y balanced accuracy.

---

## Resultados

Modelo reportado: **Random Forest balanceado con ROSE y umbral ajustado (~0,17)**,
evaluado sobre el conjunto de prueba.

| Métrica            | Valor |
|--------------------|-------|
| AUC-ROC            | ~0,73 |
| Recall (detecta IES) | ~0,63 |
| Especificidad      | ~0,71 |
| Precisión          | ~0,61 |
| Kappa              | ~0,33 |
| Balanced Accuracy  | ~0,67 |

Es un modelo de discriminación **moderada**: distingue convenios IES de no-IES por encima
del azar, principalmente por el tamaño y el plazo del contrato (`log_valor`, `plazo`,
`es_grande`), pero no es un clasificador de alta precisión. El principal hallazgo
metodológico es que el ajuste del umbral, más que el re-muestreo, es lo que equilibra
recall y precisión: el umbral es una **palanca de política** que se mueve según el costo
relativo de cada tipo de error.

---

## Estructura del repositorio

```
ATENEA-ML-ConveniosEducativos/
├── README.md
├── .gitignore
├── ATENEA_ML_ConveniosEducativos.Rmd     # Análisis reproducible en R
├── datos/
│   └── SECOP_ATENEA_TOTAL-17.xlsx        # Base de datos (hoja SECOP_ATENEA_SEL)
└── ATENEA_ML_ConveniosEducativos.html    # Reporte (se genera al hacer Knit)
```

---

## Cómo ejecutar

1. Instala R (https://cran.r-project.org/) y RStudio.
2. Abre `ATENEA_ML_ConveniosEducativos.Rmd` en RStudio.
3. Pulsa **Knit** (o `Cmd+Shift+K`). Los paquetes, incluido `ROSE`, se instalan solos.

El script lee el Excel de la carpeta `datos/`, construye la etiqueta, entrena y evalúa los
modelos, trata el desbalance y genera el reporte HTML.

---

## Limitaciones

- Modelo **predictivo, no causal**: identifica patrones, no relaciones causa-efecto.
- La **etiqueta por palabras clave** es la principal limitación (ver advertencia arriba).
- Datos **históricos** (2022–2026); cambios en las modalidades de contratación pueden
  afectar el desempeño futuro.
- Consideraciones **éticas**: privacidad de proveedores y transparencia en los criterios
  de clasificación.

---

## Datos y licencia

Datos públicos de SECOP II (https://datos.gov.co).
