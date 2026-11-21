# Predicción de zonas susceptibles a incendios forestales en Petén mediante modelado predictivo y análisis geoespacial de datos satelitales

## Descripción

Este proyecto estima la susceptibilidad a incendios forestales en el departamento de Petén, Guatemala, con un modelo **Random Forest** entrenado con detecciones satelitales de fuego (NASA FIRMS, VIIRS S-NPP) y siete variables predictoras: precipitación, temperatura máxima, elevación, pendiente, cobertura del suelo, distancia a vías y distancia a poblados.

El resultado es un mapa de susceptibilidad a 30 m de resolución, clasificado en cinco niveles (Muy baja, Baja, Media, Alta y Muy alta). A partir de ese mapa se elaboraron un análisis por municipio, una cuadrícula de 5 km para ubicar zonas críticas y un **dashboard interactivo** que funciona sin conexión a internet.

Resultados principales del modelo 2017–2024:

| Indicador | Valor |
|---|---|
| AUC (conjunto de prueba) | 0.8196 |
| Exactitud balanceada | 0.7347 |
| Sensibilidad (recall) | 0.7744 |
| Territorio en susceptibilidad Alta + Muy alta | 14.08 % |
| Detecciones de 2025 (año no usado en el entrenamiento) en zonas Alta + Muy alta | 50.66 % (AUC 0.784) |
| Variable más importante | Distancia a poblados (19.08 %) |
| Municipios con mayor superficie en Alta + Muy alta | San Andrés (31.15 %) y La Libertad (24.90 %) |

## Tecnologías Utilizadas

- **Python 3.13**: pandas, NumPy, scikit-learn, GeoPandas, rasterio, Matplotlib, joblib y Pillow
- **Jupyter Notebook**: entrenamiento, evaluación del modelo y generación de mapas
- **QGIS y GDAL**: preprocesamiento, reproyección a EPSG:32616 (UTM 16N), remuestreo a 30 m y cálculo de distancias
- **HTML, CSS y JavaScript**: dashboard interactivo en un solo archivo
- **Fuentes de datos**: NASA FIRMS (VIIRS S-NPP 375 m), CHIRPS, CHIRTS-ERA5, Copernicus DEM GLO-30, ESA WorldCover 2021, OpenStreetMap y geoBoundaries

## Requisitos Previos

- Python 3.13 (probado con 3.13.3)
- Jupyter Notebook o JupyterLab
- Un navegador web (Chrome, Edge o Firefox) para el dashboard
- QGIS, solo si se desea repetir el preprocesamiento de las capas

## Instalación

1. Clonar el repositorio:

```bash
git clone https://github.com/[usuario]/2026-[carnet]-silvia-alejandra-illescas-fernandez.git
cd 2026-[carnet]-silvia-alejandra-illescas-fernandez
```

2. Instalar las dependencias:

```bash
pip install pandas==2.3.1 numpy==2.3.1 scikit-learn==1.8.0 geopandas==1.1.4 rasterio matplotlib joblib pillow jupyter
```

3. Configurar las variables de entorno: el proyecto no utiliza credenciales ni claves de API, por lo que `src/.env.example` solo documenta las rutas de datos. Antes de ejecutar el notebook se deben actualizar las rutas de los archivos de entrada, que están escritas para la carpeta local `C:\Users\Silvia\Documents\GradIncendiosPeten`.

4. Ejecutar:

   - **Modelo y mapas:** abrir `src/modelo_random_forest_ordenado.ipynb` y ejecutar las celdas en orden. La sección 10 contiene el modelo principal 2017–2024 y la sección 13 la validación con FIRMS.
   - **Análisis geoespacial** (estadísticas, cambio, persistencia, municipios, cuadrícula y validación):

     ```bash
     python src/analisis_susceptibilidad_peten.py --datos "ruta/a/los/datos" --salida "ruta/salida"
     ```

   - **Dashboard:** abrir `src/dashboard_incendios_peten.html` con doble clic. No requiere instalación ni conexión a internet.

## Capturas

Las capturas del dashboard en funcionamiento se encuentran en la carpeta [`/capturas`](./capturas).

## Demo

El video demostrativo se encuentra en [`/demo/demo.mp4`](./demo/demo.mp4).

## Documentación

- Informe final: [`/docs/informe_final.pdf`](./docs/informe_final.pdf)
- Consentimiento de uso de foto y video: [`/docs/consentimiento_firmado.pdf`](./docs/consentimiento_firmado.pdf)

## Autores

- **Silvia Alejandra Illescas Fernández**, carné [carné]
- Asesor: Diego Josue Incer Nuñez

Universidad del Valle de Guatemala, 2026

## Licencia

Proyecto desarrollado con fines académicos como trabajo de graduación de la Universidad del Valle de Guatemala. Los datos de origen pertenecen a sus respectivos proveedores (NASA, CHC/UCSB, Copernicus/ESA, OpenStreetMap y geoBoundaries) y están sujetos a sus propias licencias.
