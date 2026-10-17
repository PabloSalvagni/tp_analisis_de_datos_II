
# Propósito

Modelo educativo (TP) para practicar feature engineering, prevención de data leakage y auditoría de sesgos. Estima el ingreso anual (`income`) de una persona a partir de datos del censo de EE. UU. No apto para decisiones reales sobre personas.

# Entradas y salidas

- **Entradas (8 features):** age, hours_per_week, education, employer_type, marital_status, sex, race, born_in_us.
- **Salida:** `income` estimado, en dólares (problema de regresión).
- **Variables sensibles auditadas:** `sex` y `race`.
- **Modelo:** regresión lineal + preprocesamiento (imputación, estandarización, one-hot) dentro de un Pipeline; target transformada con Box-Cox (`TransformedTargetRegressor`).

# Métricas de desempeño global

MAE en dólares, con validación cruzada 5-fold (predicciones out-of-fold, sin data leakage).

|        Metric       | Value |
| :-----------------: | :---: |
| MAE_target_original | 31104 |
|  MAE_target_boxcox  | 27950 |

# Métricas por subgrupo

MAE por subgrupo de `sex` y `race` (predicciones out-of-fold). Umbral de equidad: ratio min/max de MAE >= 0.8 (regla del 80%).

## MAE por sex

| sex    |   MAE |   sesgo_medio |    n |
|:-------|------:|--------------:|-----:|
| Female | 21932 |         -7537 | 4864 |
| Male   | 33649 |        -12810 | 5136 |

## MAE por race

| race              |   MAE |   sesgo_medio |    n |
|:------------------|------:|--------------:|-----:|
| Asian             | 32760 |        -12803 |  432 |
| Black             | 19619 |         -5295 | 2341 |
| Other or multiple | 19015 |         -5664 |  430 |
| White             | 31079 |        -12077 | 6797 |

## Resumen

| Atributo   |   MAE diferencia |   ratio (min/max) | Resultado   |
|:-----------|-----------------:|------------------:|:------------|
| sex        |            11717 |              0.65 | ALERTA      |
| race       |            13745 |              0.58 | ALERTA      |

## Conclusión

Se detectaron disparidades de MAE en: sex, race (ratio min/max por debajo de 0.8).

# Limitaciones de uso

- **Sesgos históricos:** los ingresos del censo ya reflejan desigualdades entre grupos; el modelo las reproduce (subestima a todos los grupos, ver sesgo medio negativo).
- **Disparidad por subgrupo:** el MAE es mayor en Male y en White/Asian (grupos de ingreso más alto): el error absoluto crece con el nivel de ingreso.
- **Subgrupos chicos:** en `race`, Asian y Other (~430 c/u) dan un MAE más ruidoso.
- **Modelo lineal:** no captura relaciones no lineales ni interacciones complejas.
- **No usar** para decisiones reales ni fuera del rango de entrenamiento (edad 17-92 años).
