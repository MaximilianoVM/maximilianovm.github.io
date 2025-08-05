---
title: "ISIS Image Subtraction: Datos propios, Rutinas comunes y Power ups"
date: 2025-04-15 00:26:00 -0700
categories: [Astro, ISIS]
tags: [Astro, ISIS]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/var_simbad.png
  alt: imagen de variaciones superpuesta en el catálogo de estrellas conocidas.
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




