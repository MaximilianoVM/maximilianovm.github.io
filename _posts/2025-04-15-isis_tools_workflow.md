---
title: "ISIS Image Subtraction: Datos propios, Rutinas comunes y Power ups"
date: 2025-04-15 00:26:00 -0700
categories: [Astro, ISIS]
tags: [Astro, ISIS]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/lc10_lightcurve.png
  alt: curva de luz con isis_tools
comments: true
---

* `./detect.csh` no genera el `var.fits` ?
*  dates file ? 
*  curvas de luz automatizadas ?

*Esto es un intento de comunicar [mi bitácora](https://veiled-foxtail-58f.notion.site/Bit-cora-Marzo-1-1ab47b4dc47e80a8966cd405cbfb964d)*
* _desde las periferias de Ensenada._

* Esta entrada no está del todo terminada ni generalizada. Por ahora es para uso personal y divulgación interna.

Notas: 
* 🍂 En Linux Mint 21.3 Cinnamon, pero no debería cambiar mucho en otras distribuciones.
* 🌌 Se asume que ya cuentas con IRAF en tu equipo.

## 🌌 IRAF init

```bash
cd iraf/isis_workspace/isis_host/package
source activate iraf27

pyraf
```

```shell 
# ---- en iraf ---- #
!ds9 -mode none -geo 760x760+2000+1000 -zscale -log -cmap grey -cmap invert yes -regions show no -mode none &

display dev$pix 1
```

# ❗ Detect: Weighted stack of subtracted images
Al trabajar con un gran volumen de imagenes, ISIS presenta problemas de memoria. Esto es debido a que el paquete está escrito en C, el cual requiere cierto conocimiento sobre el uso de memoria ya que no la gestiona de manera automatica como si lo hacen otros lenguajes de mas alto nivel. Lamentablemente el codigo fuente de ISIS no está bien optimizado en este aspecto. Evitaremos dentro de lo posible modificarlo directamente. 

Codigo fuente `abs/main.c` desde linea 68: 
```c
...
 file_list  = (FILE **)malloc(nargc*sizeof(FILE *));
 file_list2 = (FILE **)malloc(nargc*sizeof(FILE *));
 ndex = (int *)malloc(nargc*sizeof(int));
 buffer = (DATA_TYPE *)malloc(nargc*sizeof(DATA_TYPE));
...
```
En caso de ser necesario, sospecho que podemos moverle aquí.

Por ahora una solución que esperamos sea suficientre será utilizar 500 fits igualmente espaciados de todo nuestro conjunto para la creacion del var.fits, esto desde un `detect.csh` modificado al que llamaremos simplemente `detect2.csh`: 

```sh
#! /bin/csh -f

set dir           = `grep IM_DIR process_config|awk '{print $2}'`
set dir_mrj       = `grep MRJ_DIR process_config|awk '{print $2}'`
set ref_file      = `grep REF_SUB process_config|awk '{print $2}'`
set date_file     = `grep INFILE  process_config|awk '{print $2}'`
set phot_file     = `grep VARIABLES  process_config|awk '{print $2}'`
set dir_config    = `grep CONFIG_DIR process_config|awk '{print $2}'`
set thresh        = `grep SIG_THRESH process_config|awk '{print $2}'`
set n_reject      = `grep N_REJECT process_config|awk '{print $2}'`
set mesh_smooth   = `grep MESH_SMOOTH process_config|awk '{print $2}'`

cd $dir

# Calcular el paso usando awk
set list = `awk -v n=500 'NR==FNR{total++; next} FNR==1{step=int(total/n)} (FNR-1)%step==0 {print "conv_"$1}' $date_file $date_file | head -500`
# Ejecuta el abs con 500 fits igualmente espaciados de nuestro conjunto
$dir_mrj"/bin/abs" $list -o var.fits -c ../register/phot_config -t $n_reject -s $mesh_smooth -m
```

* **OTRA IDEA:** Cubrir todo el conjunto por bloques de 500 y luego de algun modo integrar cada bloque para crear el var.fits

# ⚒️ ISIS tools

[ISIS tools](https://github.com/MaximilianoVM/isis_tools)

Nos vamos a colocar en nuestro directorio `package`.

Entonces clonamos el paquete: 

```shell
git clone https://github.com/MaximilianoVM/isis_tools.git
```

* **`save_nonphased_lc.py`**: Guarda las curvas de luz en la carpeta `imagenes_curvas_<i>` (*i* especificada en input)
* **`save_lightcurves.py`**: Guarda las curvas de luz en fase con czerny en la misma `imagenes_curvas_<i>`.

Al ser un paquete principalmente hecho sobre la marcha de acuerdo a las necesidades que iban surgiendo, no está del todo optimizado para su uso publico. 
Por ahora cada comando deberá ejecutarse así desde el directorio isis_tools: 

```shell
python3 save_nonphased_lc.py

python3 save_lightcurves.py
```


# 🔨 Makefile (proximamente)

...

# 🌠 Datos propios

En progreso, por ahora, este es mi flujo de trabajo para cada nuevo conjunto de datos: 

```bash
mkdir images3 register3
cp register/* register3/

# traemos las imagenes a images3
cp ../../../1RXS/20241124* images3/
# EN EL PYRAF 
!xed ../register3/dates
# extraer JD de los headers
hselect 2024*o.fits $I,JD yes > ../register3/dates

!xed ../register3/dates
# EN TERMINAL
# restar -2460400 al JD
!awk '{printf "%s %.6f\n", $1, $2 - 2460400}' ../register3/dates > ../register3/dates.tmp && mv ../register3/dates.tmp ../register3/dates
```

### Resultado:
<!--
- Todo el contenido de `/isis` (incluyendo `package`, binarios y demás) estará **en tu host** dentro de `isis_host`.
- Los cambios que hagas dentro o fuera del contenedor estarán sincronizados.
- No vas a perder nada al hacer `exit`.
-->
## Que podemos hacer?
<!--
- ✅ Sí corre el `./install`.
- ✅ Sí corre el `./process.csh` y el resto de instrucciones `.csh` sin ningun problema. 
- ✅ Sí se pueden abrir en el el host los **.fits** generados en el contenedor.
-->




