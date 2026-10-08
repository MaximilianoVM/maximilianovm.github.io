---
title: "Proyectos en equipo: Como me pongo al corriente?"
date: 2026-09-07 03:47:00 -0700
categories: [Trabajo en Equipo]
tags: [python, jupyter, proyectos]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/GL_hal-john-gatos1.jpeg
  alt: equipo.
comments: true
---

* Desde Ags.
* Ya tengo escritorio

* Comandos ejecutados desde la terminal en Linux Mint Cinnamon. 
* Pendiente buscar equivalentes en Windows y Mac. 
* Pendiente describir instalaciones necesarias.


Me parece un uso adecuado de  modelos de lenguaje tales como Claude, Gemini o Google Notebooks. 

Ya que, idealmente, nuestro codigo no contiene informacion sensible sobre los datos que trata, podemos darnos la libertad de proveer la estructura del proyecto. 

Recomiendo Google Notebooks debido a su natualeza contenida dentro de las fuentes proveídas. En este caso, los archivos -no sensibles- de nuestro proyecto. 
# 1. Abstracción del proyecto
Con el objetivo de ahorrar tokens (en el caso de Claude, por ejemplo) o garantizar compatibilidad de archivos, lo mas adecuado es una fotografía tranversal del estado actual del proyecto en formato texto (se aceptan sugerencias alternativas). Para codigo en distintos tipos de archivos `.py`, `.ipynb`, `R`, etc. lo mas conveniente es una equivalencia en `.txt` o `.pdf`, para esto, nos apoyaremos de dos herramientas: 
### Para los notebooks `.ipynb`
Nos colocamos en la carpeta de interés, idealmente en `/notebooks` del proyecto
```bash
jupyter nbconvert --to pdf *.ipynb
```
Cuya salida es un archivo `.pdf` para cada `.ipynb` en el directorio, con un formato muy agradable incluso para entrega. 

### Para los `.ipynb`
Nos colocamos en el directorio de interés, ya sea el `\src` o el `\tests` y ejecutamos: 
```bash
for file in *.py; do
    pandoc "$file" -o "${file%.py}.pdf" \
    --pdf-engine=xelatex \
    -V mainfont="DejaVu Sans" \
    -V monofont="DejaVu Sans Mono" \
    --metadata title="${file%.py}"
done
```
Lo cual nos regresa un `.pdf` no muy bonito para cada `.py` en el directorio, afortunadamente no es para consumo humano. 

### prompt 
Recomiendo un prompt del tipo: 
> Quiero ponerme al corriente con el proyecto, explicame todo como un nuevo integrante programador al equipo ya que no entiendo lo que hacen las funciones ni como se conectan. Dime lo que falta para un entregable definitivo. Haz un diagrama que explique el flujo por modulo, la salida y entrada de cada uno.

### Resultado 
Usando como ejemplo un proyecto de segmentación de datos médicos, Google Notebooks nos facilitó el siguiente resumen: 

**Diagrama de Flujo del Pipeline por Módulos**

A continuación se presenta la arquitectura general del sistema, mostrando cómo fluye la información entre cada módulo ubicado dentro de la carpeta `src/`:

```
 [ Base de Datos ISIC ] ---> ( Archivos .jpg / .png )
                                   |
                                   v
                         +-------------------+
                         |   src/io_utils.py |
                         +-------------------+
                                   |
                                   |  Entrada: Rutas de archivos
                                   |  Salida:  imagen RGB (H, W, 3)
                                   |           mascara Ground-Truth (H, W)
                                   v
             +-------------------------------------------+
             | src/slic_custom.py / presegmentation.py  |
             +-------------------------------------------+
                                   |
                                   |  Entrada: imagen RGB (H, W, 3), K_segmentos
                                   |  Salida:  regiones_sp [ list of (H, W) bool ]
                                   v
                         +-----------------------+
                         | src/quaternion_qft.py |
                         +-----------------------+
                                   |
                                   |  Entrada: imagen RGB + regiones_sp
                                   |  Salida:  espectros_qft [ list of (M_k, N_k, 4) ]
                                   v
                      +-----------------------------+
                      | src/feature_extraction.py  |
                      +-----------------------------+
                                   |
                                   |  Entrada: espectros_qft
                                   |  Salida:  X_norm (Matriz K x 5 float64)
                                   v
                         +----------------------+
                         | src/classification.py| <--- mascara Ground-Truth (solo en Train)
                         +----------------------+
                                   |
                                   |  Entrada: X_norm, (y_true si es entrenamiento)
                                   |  Salida:  y_pred (Vector K de clases 0 o 1)
                                   v
                          +--------------------+
                          | src/evaluation.py  | <--- mascara Ground-Truth (evaluación)
                          +--------------------+
                                   |
                                   |  Entrada: regiones_sp + y_pred
                                   |  Salida:  mascara_pred (H, W) binaria
                                   |           Métricas (IoU, Dice, Precision, Recall)
                                   v
                   [ Máscara Segmentada & Reporte ]
```

---

**Entradas, Salidas y Responsabilidad de cada Módulo**

**1.** **src/io_utils.py** **(Carga e inspección de datos)**9

- **Entrada:** Rutas del sistema de archivos donde residen las imágenes y sus máscaras de referencia (por ejemplo, `data/raw/train/` y `data/masks/train/`)19.
- **Funciones principales:** `cargar_imagen()`, `cargar_mascara()`, `listar_pares_imagen_mascara()`9.
- **Salida:**
    - `imagen`: Arreglo NumPy de dimensiones `(H, W, 3)` de tipo `uint8` o `float64`9.
    - `mascara`: Arreglo NumPy en escala de grises / binario de dimensiones `(H, W)`9.
    - Lista de tuplas con los pares de rutas emparejados por el ID de la imagen9.

**2.** **src/slic_custom.py** **/** **src/presegmentation.py** **(Agrupamiento en superpíxeles)**1011

- **Entrada:** `imagen` RGB de `(H, W, 3)` y parámetros de segmentación (`n_segmentos=K`, `m` compacidad)1011.
- **Funciones principales:** `slic()` / `presegmentar_superpixeles()`1011.
- **Salida:** `regiones_sp`, una lista de $K$ máscaras booleanas de tamaño `(H, W)`, donde cada elemento marca con `True` los píxeles pertenecientes al superpíxel $k$1012.

**3.** **src/quaternion_qft.py** **(Álgebra Cuaterniónica y QFT 2D)**513

- **Entrada:** `imagen` RGB de `(H, W, 3)` y la lista de máscaras `regiones_sp`13.
- **Funciones principales:**
    - `rgb_a_cuaternion()`: Representa cada píxel como $q(x,y) = 0 + R\cdot i + G\cdot j + B\cdot k$, retornando una matriz de `(H, W, 4)` con la parte real $w=0$514.
    - `procesar_qft_superpixeles()`: Para cada superpíxel, extrae su caja envolvente (_bounding box_ $M_k \times N_k$), apaga los píxeles externos a la región y calcula la Transformada Cuaterniónica de Fourier 2D1315.
- **Salida:** `espectros_qft`, una lista de $K$ matrices de dimensión `(M_k, N_k, 4)` conteniendo las coeficientes espectrales cuaterniónicas de cada superpíxel1316.

**4.** **src/feature_extraction.py** **(Descriptores espectrales y escalado)**1718

- **Entrada:** Lista `espectros_qft` de $K$ regiones18.
- **Funciones principales:**
    - `calcular_magnitud_espectral()`: Obtiene $|F(u,v)| = \sqrt{|F_w|^2 + |F_x|^2 + |F_y|^2 + |F_z|^2}$17.
    - `extraer_matriz_caracteristicas()`: Calcula para cada superpíxel un vector de 5 características: **[media magnitud, desviación estándar, máximo, energía espectral, entropía de Shannon espectral]**1819.
    - `normalizar_caracteristicas()`: Aplica estandarización Z-Score sobre la matriz1819.
- **Salida:** Matriz de características normalizadas `X_norm` de forma `(K, 5)`1820.

**5.** **src/classification.py** **(Etiquetado y Clasificación Supervisada)**2122

- **Entrada:**
    - Para etiquetar (entrenamiento): `regiones_sp`, `mascara_gt` de `(H, W)` y un `umbral` (ej. 0.5 o 0.9)2122.
    - Para entrenar/predecir: Matriz `X_norm` de `(K, 5)` y vectores de etiquetas22.
- **Funciones principales:**
    - `etiquetar_superpixeles()`: Calcula la fracción de traslape entre el superpíxel y la lesión real. Si supera el umbral, asigna clase `1` (lesión); de lo contrario, `0` (piel sana)2123.
    - `entrenar_modelo()` y `predecir()`: Entrena modelos de Scikit-Learn (`SVM`, `Random Forest`, `KNN`)22.
- **Salida:**
    - Vector `y_true` de dimensión `(K,)` con clases binarias reales23.
    - Vector `y_pred` de dimensión `(K,)` con las clases predichas por el clasificador para cada superpíxel24.

**6.** **src/evaluation.py** **(Reconstrucción y Métricas)**2526

- **Entrada:** `regiones_sp`, vector de predicciones `y_pred` de `(K,)` y la `mascara_real` de `(H, W)`2527.
- **Funciones principales:**
    - `reconstruir_mascara()`: Repinta la imagen asignando a cada píxel el valor predicho (`0` o `1`) de su superpíxel correspondiente25.
    - `calcular_metricas()`: Calcula IoU, Dice (F1), Precisión y Recall a nivel de píxel comparando la máscara reconstruida contra la real2627.
- **Salida:**
    - `mascara_pred`: Matriz binaria reconstruida de `(H, W)`25.
    - Diccionario de métricas numéricas26.

---

**¿Qué falta para un Entregable Definitivo?**

Revisando el estado actual del repositorio, los módulos individuales en `src/` ya están construidos y validados mediante unit tests (`pytest tests/`)28 y notebooks explicativos (`fase1` a `fase4`)1more_horiz. Para completar un **entregable definitivo y listo para producción**, nos faltan los siguientes componentes clave:

1. **Construir el script principal ejecutable (****src/pipeline.py****)**29
    - Actualmente el archivo `pipeline.pdf` señala explícitamente: _"vacío por ahora, lo armamos al final, cuando cada módulo de arriba ya esté implementado y probado."_29
    - **Lo que se debe hacer:** Crear un script de Python ejecutable desde la línea de comandos (CLI) con `argparse` o `click`, que reciba la ruta de una imagen (y opcionalmente su máscara), ejecute todo el flujo de extremo a extremo sin depender de notebooks interactivos, y guarde la máscara predicha resultante y sus métricas en disco2429.
2. **Entrenamiento masivo con el Dataset completo (Full-Scale Benchmark)**3031
    - En los notebooks de desarrollo se utilizaron subconjuntos pequeños (por ejemplo, `N_TRAIN = 40` imágenes de 900 disponibles y `N_TEST = 15` de 379) para agilizar las pruebas iterativas2more_horiz.
    - **Lo que se debe hacer:** Ejecutar el pipeline completo sobre las **900 imágenes de entrenamiento** y **379 imágenes de test** de ISIC 2016, generando las tablas finales definitivas de IoU y Dice global2more_horiz.
3. **Persistencia y serialización de modelos (****joblib** **/** **pickle****)**
    - Actualmente el clasificador se entrena en memoria cada vez que se corre un experimento2431.
    - **Lo que se debe hacer:** Agregar lógica de guardado/carga del modelo entrenado (ej. `modelo_rf_qft.joblib`) y del escalador Z-Score, permitiendo inferencia inmediata en imágenes nuevas sin reentrenar31.
4. **Post-procesamiento morfológico de la máscara**
    - Como la predicción se realiza a nivel de superpíxel, las máscaras reconstruidas suelen presentar bordes escalonados o pequeños superpíxeles falsos positivos aislados2434.
    - **Lo que se debe hacer:** Aplicar operaciones morfológicas simples (apertura/cierre, eliminación de componentes conexas diminutas o relleno de agujeros) para suavizar el contorno final de la lesión34.
5. **Documentación final y reporte de resultados**34
    - Consolidar las tablas de métricas promedio (`mean`, `std`, `min`, `max`) y las comparativas visuales de casos de éxito y fallo (ej. imágenes con vello cutáneo o bajo contraste) para la memoria técnica del proyecto33more_horiz.

---

Si quieres empezar a meter mano en el código, una excelente primera tarea sería colaborar en la creación de `src/pipeline.py` tomando como plantilla la función `ejecutar_demostracion()` desarrollada en el cuaderno de la Fase 2. ¿Te gustaría que revisemos esa función juntos para armar el CLI?