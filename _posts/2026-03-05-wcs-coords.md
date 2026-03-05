---
title: "Texis + VSCode: Plantilla para tesis en VSCode."
date: 2026-03-05 03:47:00 -0700
categories: [Astro]
tags: [Python]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/overlay_wcs_imagen.png
  alt: jinja.
comments: true
---

* AAAAAHHHH 🥀

* _desde Ensenada._ En correcciones, aún no defiendo.

Notas: 
* 🍂 

# Del Notion

### Generamos nuestro catalogo de coordenadas

- Usamos como referencia el REFERENCE`.fit`

```bash
mkdir ../isis_tools/NGC6426_2407_V

cp interp_**2024071800630o**.fit ../isis_tools/NGC6426_2407_V/
```

- Lo metemos al https://nova.astrometry.net/
- traemos el wcs.fits a NGC6426_2407_V
- corremos el `phot_input.py` con el REFERENCE`.fit`
- output = `202..._I`
- reemplazamos la lista en `phot.data`

```bash
xed phot.data
```

Nos traemos el `var.fits` 

```python
cp var.fits ../isis_tools/NGC6426_2407_V/
cp abs.fits ../isis_tools/NGC6426_2407_V/
cp ref.fits ../isis_tools/NGC6426_2407_V/
```


```python
ya cerré el vs, mañana le sigo
```