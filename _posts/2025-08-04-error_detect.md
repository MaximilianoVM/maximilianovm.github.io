---
title: "Error en ./detect.csh al trabajar con muchas imagenes"
date: 2025-08-04 17:26:00 -0700
categories: [Astro, ISIS]
tags: [Astro, ISIS]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/penguin_cafe.png
  alt: pinguino es isis.
comments: true
---

* `./detect.csh` no genera el `var.fits` ?

* Ideas y soluciones no definitivas.

* _desde IA-UNAM Ensenada._ ya mero le terminamos de saber a ISIS.

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


