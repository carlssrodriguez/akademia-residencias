# Residencias de Mayores — Algoritmo de Selección de Ubicaciones

> Proyecto de Data Science desarrollado para la Akademia Future Builders de Fundación Innovación Bankinter.  
> Selección de las 100 ubicaciones óptimas en España para la apertura de residencias de mayores, mediante un índice de atractivo territorial construido sobre datos públicos del INE, IMSERSO y Ministerio de Sanidad.

---

## Resumen del proyecto

El reto consistía en actuar como comité de dirección de una cadena de residencias de mayores y proponer 100 ubicaciones candidatas en España, con una justificación estratégica y cuantitativa sólida.

El resultado es un **pipeline completo de Data Science** que parte de datos crudos del INE (106MB, 36.334 secciones censales), cruza siete fuentes de datos públicas, calcula 24 variables de negocio, construye un índice F-of-M ponderado y selecciona las 100 mejores ubicaciones con diversificación geográfica garantizada.

---

## Stack técnico

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)
![pandas](https://img.shields.io/badge/pandas-2.1-lightblue?logo=pandas)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3-orange?logo=scikit-learn)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)
![folium](https://img.shields.io/badge/folium-maps-green)

```
Python 3.11 · pandas · numpy · scikit-learn · folium · matplotlib · seaborn · openpyxl
```

---

## Estructura del proyecto

```
akademia-residencias/
│
├── M1_pipeline_datos.ipynb          # Data Engineering: lectura y limpieza de datos crudos
├── M2_variables_demanda.ipynb       # Feature Engineering: cálculo de variables de negocio
├── M2_actualizacion.ipynb           # Integración de nuevas fuentes de datos
├── M3_fofm.ipynb                    # Modelo F-of-M v1 (sin datos de competencia)
├── M3_fofm_actualizado.ipynb        # Modelo F-of-M completo con 5 componentes
├── M4_clustering.ipynb              # K-Means: segmentación en perfiles estratégicos
├── M5_visualizaciones.ipynb         # Visualizaciones: mapa interactivo y gráficos
│
├── dataset_maestro.csv              # Dataset central: 36.334 secciones × 24 variables
├── top100_ubicaciones.csv           # Las 100 ubicaciones finalistas
├── top100_con_clusters.csv          # Top 100 con perfil estratégico asignado
├── ranking_municipios.csv           # Ranking completo de 2.825 municipios candidatos
├── resumen_provincias.csv           # Agregado por provincia
├── resultados_akademia.xlsx         # Excel con todas las pestañas para el equipo
│
├── mapa_top100.html                 # Mapa interactivo (abrir en navegador)
├── grafico_perfiles.png             # Distribución por perfil estratégico
├── grafico_top20.png                # Ranking visual del top 20
├── grafico_solvencia_mercado.png    # Scatter solvencia vs tamaño de mercado
│
└── README.md
```

---

## Fuentes de datos

| Fuente | Dataset | Granularidad |
|--------|---------|--------------|
| INE | Padrón municipal — pirámide poblacional | Sección censal (~36.000) |
| INE | Renta media por hogar 2024 | Municipio |
| INE | Proyección de hogares unipersonales 2024-2039 | Provincia |
| INE | Proyección de población mayor 2024-2039 | Nacional |
| Ministerio de Sanidad | Catálogo Nacional de Hospitales 2025 | Municipio |
| IMSERSO | Oferta residencial e índice de cobertura 2024 | Comunidad Autónoma |
| INE | Coordenadas geográficas por sección censal | Sección censal |

> Los archivos de datos crudos (ODS originales del INE) no están incluidos en el repositorio por su tamaño (106MB). Los datasets limpios generados por el pipeline sí están disponibles.

---

## Metodología

### 1. Data Engineering (M1)
Lectura incremental de un archivo ODS de 106MB con XML interno de 1.3GB mediante parser iterativo que evita cargar el archivo completo en memoria. Limpieza y normalización de tres datasets heterogéneos del INE.

### 2. Feature Engineering (M2)
Cálculo de 24 variables de negocio sobre las 36.334 secciones censales de España:

- **Demanda**: volumen y porcentaje de mayores de 75+, cohorte futura proyectada a 2029
- **Solvencia**: renta media del hogar, filtro duro de no morosidad (≥ 25.200€/año)
- **Soledad**: ratio de viudez (mujeres 80+ / hombres 80+), hogares unipersonales proyectados
- **Competencia**: índice de cobertura residencial IMSERSO, presencia de hospital general
- **Proyección**: tasa de crecimiento real del INE aplicada a la cohorte futura (+6,2% hasta 2029)

### 3. Modelo F-of-M (M3)
Índice de atractivo territorial con normalización min-max y pesos definidos por criterio de negocio:

```
F-of-M = 0.30 · Solvencia + 0.25 · Demanda + 0.25 · Soledad + 0.15 · Concentración + 0.05 · Futuro
```

Aplicación de filtro duro de solvencia (zonas no solventes = puntuación 0), agregación por municipio y selección final con restricciones de tamaño mínimo de mercado (≥ 500 personas de 75+) y diversificación geográfica (máximo 3 municipios por provincia).

### 4. Clustering K-Means (M4)
Segmentación de las 100 ubicaciones en 4 perfiles estratégicos mediante K-Means con StandardScaler. Selección de K=4 por silhouette score y criterio de interpretabilidad de negocio.

| Perfil | Municipios | Renta media | F-of-M medio |
|--------|-----------|-------------|--------------|
| Zona premium — máxima solvencia | 5 | 67.280€ | 0.496 |
| Gran ciudad — mercado masivo | 11 | 30.034€ | 0.277 |
| Municipio equilibrado — perfil estándar | 30 | 31.317€ | 0.259 |
| Zona envejecida — solvencia ajustada | 54 | 30.271€ | 0.257 |

### 5. Visualización (M5)
Mapa interactivo con folium (coordenadas GPS por sección censal del INE), gráficos de análisis con matplotlib y seaborn, y exportación a Excel para el equipo no técnico.

---

## Resultados principales

- **36.334 secciones censales** evaluadas en toda España
- **11.896 secciones descartadas** por no superar el filtro de solvencia
- **2.825 municipios candidatos** identificados
- **100 ubicaciones finalistas** en **49 provincias**
- Top 3: Pozuelo de Alarcón (F-of-M: 0.581), Boadilla del Monte (0.497), Majadahonda (0.487)

---

## Cómo ejecutar el proyecto

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/akademia-residencias.git
cd akademia-residencias

# 2. Crear entorno virtual
python -m venv .venv
.venv\Scripts\activate  # Windows
source .venv/bin/activate  # Mac/Linux

# 3. Instalar dependencias
pip install pandas numpy scikit-learn folium matplotlib seaborn openpyxl notebook ipykernel odfpy

# 4. Abrir Jupyter
jupyter notebook
```

Ejecutar los notebooks en orden: M1 → M2 → M2_actualizacion → M3_fofm_actualizado → M4 → M5.

> **Nota**: Los archivos ODS originales del INE deben descargarse por separado y colocarse en la raíz del proyecto antes de ejecutar M1.

---

## Decisiones metodológicas destacadas

**¿Por qué Valencia aparece en el top 10 si la renta es menor que Madrid?**  
El modelo penaliza las CCAA con alta oferta residencial existente. La Comunitat Valenciana tiene un índice de cobertura de 1,77 plazas por cada 100 mayores (el más bajo de las grandes CCAA), frente a 6,04 de Castilla y León. Con 84.138 personas de 75+ y mercado poco saturado, Valencia representa un océano azul estratégico.

**¿Por qué un modelo explicable en lugar de ML supervisado?**  
Una decisión de inversión de esta magnitud requiere que cada puntuación sea justificable. El F-of-M permite responder exactamente por qué cualquier municipio tiene la puntuación que tiene. Se realizó análisis de sensibilidad sobre los pesos confirmando la robustez del modelo.

**¿Por qué leer el ODS con parser XML iterativo?**  
El archivo de pirámide poblacional del INE tiene un XML interno de 1.3GB. `pd.read_excel()` intentaría cargarlo entero en memoria y agota los 4GB del sistema. El parser iterativo con `xml.etree.ElementTree` procesa y libera cada fila, manteniéndose por debajo de 500MB de uso.

---

## Contexto

Desarrollado como caso práctico para la **Akademia Future Builders** de la **Fundación Innovación Bankinter**, programa de formación en innovación y tecnología para jóvenes.

---

*Desarrollado con Python · Datos públicos INE, IMSERSO y Ministerio de Sanidad · Abril 2026*
