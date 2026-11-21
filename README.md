# Predicción de zonas susceptibles a incendios forestales en Petén

**Modelado predictivo y análisis geoespacial de datos satelitales**

Trabajo profesional · Universidad del Valle de Guatemala
Autora: Silvia Alejandra Illescas Fernández
Asesor: Diego Josue Incer Nuñez

---

## Descripción

Este proyecto estima la susceptibilidad a incendios forestales en el departamento de Petén, Guatemala, con un modelo Random Forest entrenado con detecciones satelitales de fuego (NASA FIRMS) y siete variables ambientales y humanas. El resultado es un mapa de susceptibilidad a 30 m de resolución, clasificado en cinco niveles, junto con un análisis por municipio, una cuadrícula de 5 km y un dashboard interactivo para consultar los resultados.

El proyecto incluye:

- Un **modelo principal 2017–2024**, validado con años que no se usaron en el entrenamiento (2015, 2016 y 2025).
- Un **modelo complementario 2015–2025**, con mapas anuales para observar la evolución en el tiempo.
- Un **dashboard web** que funciona sin conexión a internet.

## Resultados principales

| Indicador | Valor |
|---|---|
| AUC (conjunto de prueba, modelo 2017–2024) | 0.8196 |
| Exactitud balanceada | 0.7347 |
| Sensibilidad (recall) | 0.7744 |
| Territorio en susceptibilidad Alta + Muy alta | 14.08 % |
| Detecciones 2025 en zonas Alta + Muy alta | 50.66 % (AUC 0.784) |
| Detecciones 2015–2016 en zonas Alta + Muy alta | 34.45 % (AUC 0.731) |
| Variable más importante | Distancia a poblados (19.08 %) |
| Municipios con mayor superficie Alta + Muy alta | San Andrés (31.15 %) y La Libertad (24.90 %) |

## Fuentes de datos

| Variable | Fuente | Resolución original |
|---|---|---|
| Detecciones de fuego (variable respuesta) | NASA FIRMS, VIIRS S-NPP | 375 m |
| Precipitación anual | CHIRPS | ~5.5 km |
| Temperatura máxima | CHIRTS-ERA5 | ~5.5 km |
| Elevación y pendiente | Copernicus DEM GLO-30 | 30 m |
| Cobertura del suelo | ESA WorldCover 2021 | 10 m |
| Distancia a vías y a poblados | OpenStreetMap | Vectorial |
| Límites administrativos | geoBoundaries | Vectorial |

Todas las capas se reproyectaron a **WGS 84 / UTM zona 16N (EPSG:32616)** y se remuestrearon a una cuadrícula común de **30 m** tomando el DEM como referencia: bilineal para variables continuas y vecino más cercano para la cobertura del suelo. Las distancias se calcularon con la herramienta *Proximity* de GDAL. Este preprocesamiento se realizó en QGIS.

## Estructura del proyecto

```
GradIncendiosPeten/
├── data/
│   └── processed/
│       ├── dataset_modelo/        # dataset_modelado_2015_2025_limpio.shp
│       ├── topografia/            # dem_peten_utm.tif, pendiente_peten.tif
│       ├── worldcover/            # worldcover_peten_30m_final.tif
│       ├── vias/                  # distancia_vias.tif
│       ├── poblados/              # distancia_poblados.tif
│       ├── chirps_anual/          # precip_<año>_30m_final1.tif
│       ├── temperature_anual/     # tmax_<año>_30m.tif
│       └── firms/                 # firms_peten_2015_2025_utm.shp
├── modelado/
│   ├── modelo_random_forest_ordenado.ipynb
│   ├── random_forest_incendios.pkl              # modelo 2015–2025
│   └── random_forest_incendios_2017_2024.pkl    # modelo principal
├── resultados/
│   ├── mapas_susceptibilidad/     # susceptibilidad_2015.tif … 2025.tif, final_2015_2025.tif
│   └── mapas_2017_2024_RF/        # mapas anuales y susceptibilidad_final_2017_2024_RF.tif
├── analisis/
│   └── analisis_susceptibilidad_peten.py
└── dashboard/
    └── dashboard_incendios_peten.html
```

## Requisitos

- Python 3.13 (probado con 3.13.3)
- Bibliotecas del notebook:

```bash
pip install pandas==2.3.1 numpy==2.3.1 scikit-learn==1.8.0 geopandas==1.1.4 rasterio matplotlib joblib
```

- Bibliotecas del script de análisis (no requiere GDAL ni rasterio):

```bash
pip install numpy pillow
```

- QGIS, solo si se desea repetir el preprocesamiento de las capas.
- Un navegador web (Chrome, Edge o Firefox) para el dashboard.

## Instrucciones de uso

### 1. Modelo y mapas de susceptibilidad

Abrir `modelo_random_forest_ordenado.ipynb` en Jupyter y ejecutar las celdas en orden.

> **Importante:** las rutas del notebook están escritas para la carpeta `C:\Users\Silvia\Documents\GradIncendiosPeten`. Si el proyecto se ejecuta en otra computadora, se deben actualizar las rutas al inicio de cada sección.

| Sección | Contenido |
|---|---|
| 1 | Versiones de las bibliotecas |
| 2–3 | Carga del dataset (290,122 registros: 144,904 presencias y 145,218 pseudoausencias) y división estratificada 80/20 |
| 4–7 | Entrenamiento, evaluación, importancia de variables y guardado del modelo 2015–2025 |
| 8 | Mapas anuales 2015–2025, procesados por bloques de 1024 × 1024 píxeles |
| 9 | Mapa integrado 2015–2025 (promedio de los mapas anuales con `np.nanmean`) y clasificación |
| 10 | **Modelo principal 2017–2024**: entrenamiento, métricas, mapas anuales, mapa integrado, clases y zonas Alta + Muy alta |
| 11–12 | Estadísticas anuales, figuras y tablas para el documento |
| 13 | Validación temporal con detecciones FIRMS de 2015, 2016 y 2025 |

Configuración del modelo: `RandomForestClassifier(n_estimators=300, random_state=42, n_jobs=-1)`; los demás parámetros quedan con sus valores por defecto. Si un modelo `.pkl` o un mapa ya existe, el notebook lo carga en lugar de generarlo de nuevo.

**Clases de susceptibilidad** (intervalos iguales de 0.2):

| Clase | Rango |
|---|---|
| Muy baja | 0.0 – 0.2 |
| Baja | 0.2 – 0.4 |
| Media | 0.4 – 0.6 |
| Alta | 0.6 – 0.8 |
| Muy alta | 0.8 – 1.0 |

### 2. Análisis geoespacial

El script calcula estadísticas por capa, cambio 2015–2025, persistencia de valores altos, resúmenes por municipio y por celda de 5 km, y la validación con FIRMS.

```bash
python analisis_susceptibilidad_peten.py --datos "C:/ruta/a/mis_datos" --salida "C:/ruta/salida"
```

La carpeta indicada en `--datos` debe contener:

- `susceptibilidad_2015.tif` … `susceptibilidad_2025.tif`
- `susceptibilidad_final_2015_2025.tif` y `susceptibilidad_final_2017_2024_RF.tif`
- `cuadricula_5km_peten_recortada_2017_2024.shp`, `zonas_5km_municipios_2017_2024.dbf`, `susceptibilidad_municipios_2017_2024.shp` y `zonas_maximas_por_municipio_2017_2024.dbf`
- `firms_peten_2015_2025_utm.shp`

Si falta alguna capa vectorial o el archivo FIRMS, esa parte se omite y el resto del análisis continúa.

En la carpeta `--salida` se generan, entre otros: `estadisticas_por_capa.csv`, `cambio_2015_2025.csv`, `persistencia_alta.csv`, `municipios.csv`, `celdas_5km.csv`, `validacion_firms.csv`, `focos_por_municipio.csv` y `estadisticas.json`. Estos resultados son los que se integraron en el dashboard.

### 3. Dashboard

Abrir `dashboard_incendios_peten.html` con doble clic. Es un solo archivo HTML con los datos incluidos, por lo que **no necesita instalación ni conexión a internet**.

Secciones del dashboard:

- **Mapa de susceptibilidad:** serie anual 2015–2025 con animación, modelos finales, cambio 2015–2025 y persistencia. Al hacer clic en un punto se muestran su valor, su clase y sus coordenadas.
- **Modelo predictivo:** métricas e importancia de las variables.
- **Análisis por municipio:** los 14 municipios ordenados por superficie en susceptibilidad Alta + Muy alta.
- **Cuadrícula de 5 km:** zonas críticas dentro de cada municipio, con filtro por municipio.
- **Validación:** distribución de las detecciones FIRMS por clase de susceptibilidad.

El mapa del dashboard se muestra a 120 m para mantener un tamaño de archivo manejable; los análisis se calcularon a 30 m.

## Limitaciones

- FIRMS registra **detecciones de calor**, no incendios confirmados en campo; una detección puede corresponder a quemas agrícolas.
- La resolución de las detecciones (375 m) es mayor que la de los mapas (30 m).
- La cobertura del suelo corresponde a un solo año (WorldCover 2021).
- Las pseudoausencias son puntos sin detección registrada, no confirmaciones de ausencia de fuego.
- El resultado indica **susceptibilidad** (condiciones favorables al fuego), no el riesgo ni la probabilidad de que ocurra un incendio en una fecha concreta.

## Licencia y uso

Proyecto desarrollado con fines académicos como trabajo profesional de graduación de la Universidad del Valle de Guatemala. Los datos de origen pertenecen a sus respectivos proveedores (NASA, CHC/UCSB, Copernicus/ESA, OpenStreetMap y geoBoundaries) y están sujetos a sus propias licencias.
