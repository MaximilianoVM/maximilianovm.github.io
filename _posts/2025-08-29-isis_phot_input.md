---
title: "ISIS: curvas de luz para coordenadas Physical propias"
date: 2025-08-29 17:28:00 -0700
categories: [Astro, ISIS]
tags: [Astro, ISIS]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/phot_2506A_I.png
  alt: NGC6426 stars.
comments: true
---

la lista sobre la cual se hace la fotometria es **`registerX/phot.data`**, que se ve así: 

para **`register2/phot.data` :**

```bash
47.818095 53.288971 48 53 lc0.data 1.108024 18.838448
98.324050 53.530079 99 53 lc1.data 0.530456 6.804914
149.000000 105.000000 149 105 lc2.data 1.504538 20.553673
177.773145 153.121297 177 153 lc3.data 0.579896 13.999767
197.284906 203.871915 197 204 lc4.data 0.291704 9.640295
```

Columnas: `x, y, int(x), int(y), name.data, var.fits, abs.fits`

<aside>
podremos prescindir de las ultimas dos columnas?  → ❌
</aside>

💡 las ultimas dos columnas corresponden al valor en `var.fits` y en `abs.fits`

La fotometría se hace para valores de `var.fits` mayores a los establecidos en el `sig_thresh`, filtro ahora innecesario debido a nuestro input manual. 

Podemos modificar phot.data (despues de `./detect.csh` y  `./find.csh`). 

### modificamos el `phot.data`
→ Nótese que el nombre de la curva de luz es personalizable, mientras que los valores de las ultimas dos columnas deben ser consistenes con lo que hayamos puesto en `sig_thresh` para que se haga la fotometría.

```bash
47.818095 53.288971 48 53 lc0.data 1.108024 18.838448
98.324050 53.530079 99 53 lc1.data 0.530456 6.804914
99.25 54.25 99 54 lcmod1.data 1 17
149.000000 105.000000 149 105 lc2.data 1.504538 20.553673
177.773145 153.121297 177 153 lc3.data 0.579896 13.999767
197.284906 203.871915 197 204 lc4.data 0.291704 9.640295
175.75 67.75 176 68 lcmax0.data 1 17
68.75 147.25 69 147 lcmax1.data 1 17
174.25 178.75 174 179 lcmax2.data 1 17
101.25 132.75 101 133 lcmax3.data 1 17
```

De esta forma, al correr `./phot.csh` se generarán curvas de luz sobre nuestras coordenadas con los nombres `.data` indicados. 