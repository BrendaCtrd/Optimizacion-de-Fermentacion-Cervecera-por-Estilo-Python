# Optimización de Fermentación Cervecera por Estilo (Python)

Proyecto interactivo en **Python** para **modelar** y **optimizar** condiciones de fermentación por estilo de cerveza. A partir de un dataset realista de producción (583 lotes), se entrenan modelos para **predecir ABV** y **calidad sensorial**, y luego se **recomiendan set-points** (brew sheet) que maximizan la calidad respetando el rango de alcohol del estilo.

## 🚀 Funcionalidades

**Ingesta y preparación de datos**
- Normalización composicional de receta (`ratio_1 : ratio_2 : ratio_3` → suma = 1).
- Enriquecimiento temporal (`Brew_Date` → `Month`, `Week`).
- Cálculo de **pérdidas totales** (`Total_Loss`).
- **Split temporal 80/20** para evaluar generalización.

**Modelado**
- **ABV (Alcohol by Volume):** `RidgeCV` + `StandardScaler` + `OneHotEncoder`.  
  - **Desempeño:** **R² = 0.9976**, **RMSE = 0.044 %ABV** (sobre test).
- **Quality por estilo:** `PolynomialFeatures (grado 2)` + `RidgeCV` con `TimeSeriesSplit`.  
  - Comparación contra **baseline** (predecir la media del estilo).

**Optimización por estilo (ejemplo: Pilsner)**
- Búsqueda **Monte Carlo** (~30k candidatos) dentro de los rangos históricos del estilo.
- Recetas generadas como **composición** con distribución **Dirichlet**.
- **Restricción de ABV por estilo** mediante penalización en el *score*.
- Selección de **TOP-K** candidatos y generación de **brew sheet**.

**Validación de robustez**
- **Análisis de sensibilidad ±5%** alrededor del óptimo para identificar **palancas críticas**.

**Exportables**
- `pilsner_top5.csv` y `Pilsner_brew_sheet_MAX_QUALITY_v2.csv` con los set-points operables.

## 🧪 Datos

- **Archivo:** `brewery_data.csv` (583 lotes, 2020).  
- **Campos destacados:** `Fermentation_Time`, `Temperature`, `pH_Level`, `Gravity`, `Alcohol_Content`, `Bitterness (IBU)`, `Color (EBC)`, `Ingredient_Ratio`, `Quality_Score`, pérdidas por etapa, `SKU`, `Location`.

> Nota: datos con propósito educativo y anonimización. El flujo evita extrapolar fuera de los rangos históricos.

## 🛠 Tecnologías Utilizadas

- **Python 3.13**
- **pandas 2.2.3**, **NumPy**, **scikit-learn**, **matplotlib**
- **Jupyter Notebook** / **VS Code**
- **Git** (control de versiones)

## 📈 Resultados clave

### Modelo ABV (global)
- **R² = 0.9976**, **RMSE = 0.044 %ABV** → suficiente para imponer **restricción realista** en la optimización.

### Modelos de Quality por estilo
- Se comparan contra **baseline** (predecir la media por estilo).
- **Pilsner:** el modelo **supera** su baseline en test (**RMSE_model ≈ 0.426 < RMSE_base ≈ 0.470**).

### Optimización (Pilsner)
- **TOP-5** candidatos con **ABV en rango** (4.0–5.5%).
- Se selecciona el **set-point de máxima calidad** y se aplican **micro-ajustes “seguros”** validados por sensibilidad.

### Brew sheet final (Pilsner, Max Quality v2)
- **Fermentation_Time:** ~**12.04 d**
- **Temperature:** **~9.43 °C**
- **pH:** **~3.99**
- **Gravity:** **~1.199**
- **IBU:** **~27.5**
- **Color (EBC):** **~10.95**
- **Brewhouse Efficiency:** **~82.16 %**
- **Receta:** `ratio_1 ≈ 0.472`, `ratio_2 ≈ 0.368`, `ratio_3 ≈ 0.160`
- **ABV_pred:** **~5.151 %**
- **Quality_pred:** **~10.388**

---

## 🔍 Sensibilidad (±5%) y palancas
- **Gravity:** mayor impacto sobre **ABV** (subirla +5% empuja el ABV fuera de rango). **Control estricto**.
- **ratio_2:** subirla **mejora calidad** con **mínimo efecto** en ABV; compensar bajando **ratio_1**.
- **Temperature** y **Color:** alzas moderadas tienden a **mejorar calidad** sin mover el ABV.
- **pH / IBU / tiempo:** impacto local **pequeño** alrededor del óptimo seleccionado.

---

## ✨ Conclusiones
- El proyecto entrega un **MVP robusto** de optimización por estilo con **restricción de ABV** respaldada por un modelo de alta fidelidad.
- Para **Pilsner**, el modelo de calidad **generaliza** y la optimización produjo **set-points** que **maximizan la calidad** con **ABV** dentro del rango del estilo.
- La **sensibilidad** brinda reglas operativas: controlar **Gravity** y ajustar **Temperature/Color/ratio_2**.

---

## 🚧 Limitaciones
- La **calidad sensorial** es ruidosa; pueden faltar variables (cepa/lote de levadura, oxígeno disuelto, nutrientes, tiempos intermedios).
- Los modelos están **calibrados dentro de rangos históricos**; evitar **extrapolación**.

---

## 🔭 Próximos pasos
1. Ejecutar un **lote piloto** con la brew sheet final y registrar mediciones (**ABV**, **OG/FG**, **panel sensorial**).
2. **Reentrenar** incorporando ese lote (mejora continua).
3. Para estilos que no mejoran: probar `PolynomialFeatures(degree=3)` y ampliar `alphas` de Ridge; añadir la feature **`abv_delta`** (desvío al centro del rango del estilo).
4. *(Opcional)* Incluir **penalización por pérdidas** para generar un **frente calidad–eficiencia**.



## Referencia
Husain, A. (2025). Brewery Operations and Market Analysis Dataset [Dataset]. Kaggle. https://www.kaggle.com/datasets/aqmarh11/brewery-operations-and-market-analysis-dataset