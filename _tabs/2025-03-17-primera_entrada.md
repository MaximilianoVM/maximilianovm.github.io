---
title: Primera entrada
date: 2025-03-17 00:58:SS -0700
categories: [Animal, Insect]
tags: [bee]    # TAG names should always be lowercase
---

# Prueba: 3 sigma rejection para mis curvas de luz generadas por ISIS

```python3

# Leer el archivo
lc_data2 = pd.read_csv('../images3/lc139.data', delim_whitespace=True, header=None)


jd = lc_data2[0]
values = lc_data2[1]

lc = dict(zip(jd, values))


# valores del diccionario a un array de NumPy
elements = np.array(list(lc.values()))

# calcular media y desviacion estándar
mean = np.mean(elements)
sd = np.std(elements)

# Filtrar
final_dict = {
    key: value for key, value in lc.items()
    if (value > mean - 2 * sd) and (value < mean + 2 * sd)
}

# Mostrar el diccionario filtrado
print(final_dict)
```