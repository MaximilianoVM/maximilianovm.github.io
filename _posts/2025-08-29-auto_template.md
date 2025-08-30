---
title: "JINJA: Plantilla automática de procesamiento para cada conjunto de datos."
date: 2025-08-29 17:49:00 -0700
categories: [Astro, ISIS]
tags: [Astro, ISIS]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/jinja.png
  alt: jinja.
comments: true
---

Se hace un procesamiento independiente para cada set de imagenes, ya sea dividido por filtro, noche o temporada de forma que cada imagen del conjunto pueda ser efectivamente alineada con el resto.

Para cada uno de estos sets tocan una serie de modificaciones muy sutiles a los parametros de ISIS y del procesamiento anterior o posterior. En mi caso, a cada set le doy su propia entrada de bitácora que al igual que los parametros, cambia muy poco aunque no suficiente para hacer menos agobiante y repetitiva la tarea cuando se trabaja con varios sets. 

El formato de cada una de estas entradas corresponde simplemente a la serie de instrucciones que copio y pego en IRAF, ISIS o la terminal común, además de los parámetros que también copio y pego en los archivos de ISIS. 

Fue al encontrarme con 10 sets de datos que recordé una herramienta que, entre otras cosas, permite realizar reportes parcialmente automatizados cuando lo que cambia entre cada uno son unicamente valores numericos o variables muy puntuales. Me supongo, por ejemplo, que es lo que se usa en industria y gobierno para la realizacion de reportes trimestrales.

Otro beneficio que viene al hacer las cosas de esta forma es que me libré de la incertidumbre de haber dejado uno de los valores sin cambiar. 

Para que sirva como ejemplo tanto de flujo de trabajo como para la implementación de jinja, adjunto mi plantilla, guardada como `template.j2`: 

Al final de este largo codigo explico como genero el archivo en markdown.

### 🎨🖌️ Plantilla

```python
    # Para `{{ register_dir }}`

    `{{ images_dir }}`

    - [ ] dates
    - [ ] FWHM list
    - [ ] ref_list
    - [ ] config params
    - [ ] input list
    - [ ] exportamos lcs

    ### directorios

    ```bash
    # desde package
    mkdir {{ register_dir }}

    cp {{ prev_register_dir }}/* {{ register_dir }}/
    ```

    ### Dates

    ```bash
    cd {{ images_dir }}

    # extraer **HJD** de los headers
    hselect 202*o.fit $I,**HJD** yes > ../{{ register_dir }}/dates
    !xed ../{{ register_dir }}/dates
    ```

    ### FWHM list

    ```bash
    hselect 202*o.fit $I,FWHM,SIGMA,E_AMASS,SKY yes > fwhm_sigma_amass_sky
    !xed fwhm_sigma_amass_sky

    # ordenar #ta columna (-k<#>) numericamente (-n) 
    !sort -k2 -n fwhm_sigma_amass_sky > FSAS_sortby_fwhm
    !xed FSAS_sortby_fwhm
    ```

    ### ref_list:

    ```bash
    imexam *.fit
    ```

    ```bash
    # top 10 FSAS_sortby_fwhm
    ```

    **verificamos integridad del top**

    ```python
    !awk 'NR<=15 {print "display", $1, NR}' FSAS_sortby_fwhm
    ```


    ### parámetros:

    (`{{ register_dir }}`) 

    (`{{ images_dir }}`)

    ```bash
    xed process_config default_config phot_config ref_list &
    ```

    =====

    ```bash
    IM_DIR        ../{{ images_dir }}     Directory where the images are
    MRJ_DIR       ..            Installation directory
    REFERENCE     REFERENCE.FIT    Reference image for astrometry
    REF_SUB       ref.fits      Reference image for subtraction
    INFILE        ../{{ register_dir }}/dates         Dates of the frames
    VARIABLES     phot.data     coordinates of objects for which we want to make light curves
    DEGREE          1           The degree of the polynomial astrometric transform between frames 
    CONFIG_DIR      ../{{ register_dir }}          Where to find the configuration files
    SIG_THRESH      10.0
    COSMIC_THRESH   1000.0
    REF_STACK       REFERENCE.FIT
    N_REJECT        2
    MESH_SMOOTH     1
    ```


    ```bash
     sub_x       1
     sub_y       1
     rad1_bg     15.0
     rad2_bg     20.0
     nstars      5
     mesh        2
     saturation  64000.0
     min         5.0
     psf_width   23
     radphot     6.0
     nstar_max   8
     rad_aper    7.0
     nb_adu_el   1
     rmax        0.5
     first       1
     keep        5
    ```

    ```bash
    nstamps_x         10       /*** Number of stamps along X axis ***/
    nstamps_y         10       /*** Number of stamps along Y axis***/
    sub_x             1       /*** Number of sub_division of the image along X axis ***/
    sub_y             1       /*** Number of sub_division of the image along Y axis ***/
    half_mesh_size    9      /*** Half kernel size ***/
    half_stamp_size   15      /*** Half stamp size ***/
    deg_bg            3       /** degree to fit differential bakground variations **/
    saturation        64000.0   /** degree to fit background varaitions **/
    pix_min           5.0       /*** Minimum vaue of the pixels to fit *****/
    min_stamp_center  130     /*** Minimum value for object to enter kernel fit *****/
    ngauss            3       /*** Number of Gaussians ****/
    deg_gauss1        6       /*** Degree associated with 1st Gaussian ****/
    deg_gauss2        4       /*** Degree associated with 2nd Gaussian ****/
    deg_gauss3        3       /*** Degree associated with 3rd Gaussian ****/
    sigma_gauss1      0.7     /*** Sigma of 1st Gaussian ****/
    sigma_gauss2      2.0     /*** Sigma of 2nd Gaussian ****/
    sigma_gauss3      4.0     /*** Sigma of 3rd Gaussian ****/
    deg_spatial       2       /*** Degree of the fit of the spatial variations of the Kernel ****/
    ```

    ```bash

    LISTA DE REFERENCES

    ```

    .

    =====

    ### Corremos hasta antes del phot

    ```bash
    ./process2.csh
    ```

    ### Generamos nuestro catalogo de coordenadas

    - Usamos como referencia el REFERENCE`.fits`

    ```bash
    mkdir ../isis_tools/{{ outputs_dir }}

    cp interp_REFERENCE.FIT ../isis_tools/{{ outputs_dir }}/
    ```

    - Lo metemos al https://nova.astrometry.net/
    - corremos el `phot_input.py` con el REFERENCE`.fits`
    - output = `202..._I`
    - reemplazamos la lista en `phot.data`

    ```bash
    xed phot.data
    ```

    ### Corremos el `./phot.csh`

    …

    ### Corremos después del phot

    …

    ### lista de `lc*.data`

    ```bash
    # desde {{ images_dir }}

    !ls lc*.data* | sort -u > LCS_LIST
    ```

    ### Copiar lista de archivo:

    ```bash
    xargs -a LCS_LIST -I {} cp ../../../imagesDIR/{} directorio_destino/

    # desde imagesXXXX 
    !mkdir ../isis_tools/{{ outputs_dir }}/lcs
    !xargs -a LCS_LIST -I {} cp ../{{ images_dir }}/{} ../isis_tools/{{ outputs_dir }}/lcs
    ```

    ---
```

### 🏗️ Script para renderizar la plantilla en Markdown

```python 
from jinja2 import Template
import os

# Leer la plantilla
with open('template.j2', 'r') as f:
    template_content = f.read()

template = Template(template_content)


# Variables para cada conjunto de datos
variables = {
    'prev_register_dir': 'register202506A_I',
    'register_dir': 'register202506B_I',
    'images_dir': 'images202506B_I',
    'outputs_dir': 'NGC6426_2506B_I'
}


# Renderizar la plantilla
output = template.render(**variables)


# Guardar el resultado
output_filename = f"{variables['outputs_dir']}.md"
with open(output_filename, 'w') as f:
    f.write(output)

print(f"Archivo generado: {output_filename}")

```