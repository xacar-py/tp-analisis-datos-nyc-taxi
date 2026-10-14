# NYC Yellow Taxi – agosto 2026 · Análisis de Datos

Trabajo práctico integrador de **Análisis de Datos** – Carrera de Especialización en Inteligencia Artificial (CEIA), FIUBA, cohorte 4B 2026.
Autor: Facundo (trabajo individual).

El trabajo analiza y prepara un dataset real para un problema de *machine learning* supervisado. **No se entrena ningún modelo**: el foco está en el análisis exploratorio (EDA) y en la preparación de los datos.

---

## 1. Problema planteado

| Elemento | Definición |
|---|---|
| Tipo de problema | Regresión supervisada |
| Variable target | `fare_amount`: tarifa del taxímetro en USD (sin recargos ni propina) |
| Momento de predicción | Cuando el pasajero sube al taxi e indica el destino |
| Cliente / usuario | Pasajero o aplicación de viajes que necesita cotizar el precio antes de viajar |
| Uso futuro | Estimar la tarifa con datos conocidos *a priori* (origen, destino, fecha y hora, pasajeros, tipo de tarifa), como hacen hoy los viajes *Flex Fare* pedidos por app, cuyo precio se pacta de antemano |
| Variables excluidas por *leakage* | Las que solo se conocen al terminar el viaje: distancia recorrida, hora de llegada, propina, peajes, total cobrado, medio de pago |

**¿Por qué no el taxista como cliente?** Para el conductor, la tarifa la calcula el taxímetro en cada viaje, así que predecirla le aporta poco. Una versión orientada al taxista (predecir el ingreso por minuto según zona y horario) queda como trabajo futuro.

---

## 2. Datos: de dónde salen y cómo se obtienen

### Fuente

| | |
|---|---|
| Organismo | NYC Taxi & Limousine Commission (TLC) |
| Página oficial | https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page |
| Archivo de viajes | Yellow Taxi Trip Records – agosto 2026 (`yellow_tripdata_2026-08.parquet`, ~59 MB, 3.336.716 viajes, 21 columnas) |
| Tabla de zonas | Taxi Zone Lookup Table (`taxi_zone_lookup.csv`): traduce el número de zona a barrio y borough |
| Diccionario de datos | https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf (versión 18/03/2025) |
| Fecha de descarga usada en este trabajo | 12/10/2026 |

Se trabaja con **un solo mes** siguiendo la recomendación de la cátedra, por el volumen de procesamiento.

### Los datos NO están en este repositorio

El archivo de viajes pesa ~59 MB y es un dato público de la TLC, por eso **no se sube a GitHub** (la carpeta `data/` está excluida en `.gitignore`).
No hace falta pedírselo a nadie ni descargarlo a mano: **el notebook lo descarga solo**.

### Cómo funciona la descarga automática

La primera celda de código del notebook hace lo siguiente:

1. Busca los archivos en la carpeta `data/` (al lado del notebook).
2. **Si no están**, los descarga desde los links directos de la TLC y los guarda en `data/`:
   - `https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2026-08.parquet`
   - `https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv`
3. **Si ya están**, usa esa copia local y no descarga nada.

Resultado: la **primera** ejecución tarda un poco más (descarga de ~59 MB, requiere internet); las siguientes leen la copia local.

**Alternativa manual:** descargar los dos archivos desde la página oficial y copiarlos en `data/` con exactamente esos nombres.

### Control de versión de los datos

La TLC puede republicar un mes con correcciones. Para verificar que se trabaja con la misma versión de los datos, al cargar el archivo deben obtenerse **3.336.716 filas y 21 columnas**. Si los números difieren, el archivo fue actualizado por la TLC después del 12/10/2026 y los resultados pueden variar levemente.

---

## 3. Cómo ejecutar el notebook

1. Clonar o descargar este repositorio.
2. Instalar las dependencias (Python 3.10 o superior):
   ```bash
   pip install -r requirements.txt
   ```
   `pyarrow` es necesario para leer archivos `.parquet`.
3. Abrir `TP_AdD_YellowTaxi.ipynb` en Jupyter y ejecutar todas las celdas (*Kernel → Restart & Run All*).
   La primera vez descarga los datos (ver sección 2). Con los datos ya descargados tarda unos 3 a 5 minutos (procesa 3,3 millones de viajes) y requiere alrededor de 8 GB de RAM.

---

## 4. Estructura del repositorio

```
├── README.md                      ← este archivo
├── TP_AdD_YellowTaxi.ipynb        ← notebook único: EDA, limpieza, split, preparación, selección y PCA
├── requirements.txt               ← bibliotecas necesarias
├── .gitignore                     ← excluye data/ del repositorio
├── figures/
│   ├── eda/                       ← gráficos del análisis exploratorio (PNG)
│   ├── limpieza/                  ← gráfico de la limpieza
│   ├── split/                     ← distribución del target en train y test
│   └── preparacion/               ← outliers, correlación y relevancia de features, PCA
└── data/                          ← NO se sube: se crea y llena al ejecutar el notebook
```

---

## 5. Flujo de trabajo

Se sigue el flujo estándar de preparación de datos:

| Etapa | Secciones del notebook | Estado |
|---|---|---|
| **EDA** sobre el dataset completo: solo se observa y describe, no se modifica nada | 1 a 11 | ✅ |
| **Limpieza básica** (errores imposibles) y **definición de la población** (alcance: tarifa de taxímetro) | 12 | ✅ |
| **Wrangling**: zonas, hora, día y tipos (por fila, sin estadísticos) | 13 | ✅ |
| **Split** aleatorio 80 / 20 con semilla fija | 14 | ✅ |
| **Faltantes**: moda por proveedor (`passenger_count`) y categoría "desconocido" (`RatecodeID`) | 15 | ✅ |
| **Outliers**: IQR sobre log(tarifa), se elimina el 1,15 % más extremo de train | 16 | ✅ |
| **Creación de features**: distancia típica origen–destino, aeropuerto, fin de semana | 17 | ✅ |
| **Discretización**: hora → franja horaria | 18 | ✅ |
| **Codificación**: target encoding con k-fold (zonas), one-hot, cíclica (hora y día) | 19 | ✅ |
| **Escalado**: estandarización (z-score) | 20 | ✅ |
| **Balanceo**: no aplica (regresión) | 21 | ✅ |
| **Selección de features**: dominio, redundancia, relación con el target, información mutua | 22 | ✅ |
| **PCA**: 4 componentes retienen el 80 % de la varianza | 23 | ✅ |
| **Conclusiones** | 24 | ✅ |

Todas las transformaciones se **calculan en train y se aplican en train y test**, para evitar *data leakage*.

**Población del modelo:** viajes de calle con tarifa de taxímetro. Se excluyen los viajes *Flex Fare* (precio pactado en la app), la tarifa fija JFK (USD 70) y la tarifa negociada, porque su precio no lo calcula el taxímetro (justificación con datos en la sección 12).

---

## 6. Principales observaciones del EDA

| Aspecto | Observación |
|---|---|
| Dimensiones | 3.336.716 viajes × 21 variables |
| Tipos de variables | Numéricas continuas (montos, distancia, tiempos), numéricas discretas (pasajeros, recargos) y categóricas nominales codificadas con números (zonas, tipo de tarifa, proveedor, medio de pago) |
| Target `fare_amount` | Mediana 15,6 USD, IQR 16,1. Muy asimétrico (cola derecha) y con colas pesadas. Escalonado en saltos de USD 0,70 (taxímetro), con un pico en USD 70 (tarifa fija JFK) |
| Asociación con el target | Fuerte con zonas de origen y destino y con el tipo de tarifa (η² entre 0,26 y 0,36). Muy fuerte con distancia y duración, pero son *leakage* |
| Faltantes | Bloque estructural de 5 columnas que coincide con los viajes *Flex Fare* (27,6 %, tarifa pactada por app). `request_source` solo existe para esos viajes. Faltantes "disfrazados" de tipo MAR: `passenger_count = 0` y `RatecodeID = 99` |
| Errores | Duraciones negativas, distancias 0, tarifas negativas o menores a la bajada de bandera (USD 3), zonas desconocidas, fechas fuera del mes |
| Atípicos | El 6,7 % del target según IQR; la mayoría son viajes de aeropuerto (legítimos). Los extremos con distancia 0 son errores de carga |

El detalle, con gráficos e interpretación de cada punto, está en el notebook.

---

## 7. Resultado de la preparación

| Etapa | Viajes / features |
|---|---|
| Dataset original | 3.336.716 viajes × 21 variables |
| Tras limpieza (errores, 5,9 %) y definición de la población (Flex Fare, tarifa fija JFK y negociada) | 2.212.153 viajes |
| Train (80 %, sin outliers) / Test (20 %) | 1.749.379 / 442.431 viajes |
| Features candidatas → seleccionadas | 31 → 10 |
| Componentes PCA (80 % de la varianza) | 4 |

**Features seleccionadas:** distancia típica origen–destino, zona de origen y de destino (target encoding), destino aeropuerto, tarifa estándar, origen Queens y boroughs de destino.

---

## 8. Conclusiones

1. **El dataset crudo no es directamente utilizable**: un 27,6 % de los viajes del dataset crudo (Flex Fare) sigue otra lógica de precio y ~6 % son errores imposibles. Definir la población fue tan importante como limpiar.
2. **La estadística robusta es imprescindible**: unos pocos valores absurdos llevaban la curtosis del target a más de 5.000 y anulaban a Pearson (r = 0,01 frente a ρ de Spearman = 0,79).
3. **Las variables más asociadas al precio son *leakage*** (distancia, duración, total): se reemplazaron por la **distancia típica** de cada recorrido, conocida antes de viajar (ρ = 0,85 con la tarifa).
4. **El "dónde" importa más que el "cuándo"**: zonas y tipo de tarifa tienen efecto fuerte (η² 0,26–0,36); hora, día y pasajeros casi no explican la tarifa y la selección de features los descartó.
5. **Los outliers no son todos errores**: el IQR directo marcaba viajes normales de aeropuerto; sobre el logaritmo solo marcó el ~1 % más extremo.
6. **Con millones de datos la significancia no discrimina**: ANOVA y χ² dieron p ≈ 0; las decisiones se basaron en el tamaño del efecto.

**Limitaciones:** un solo mes de datos; partición aleatoria (train y test comparten días); el modelo no cubre Flex Fare, tarifa fija JFK ni tarifas negociadas; los viajes con `RatecodeID` desconocido (~3 %) se conservan pero casi no siguen el patrón del taxímetro.
