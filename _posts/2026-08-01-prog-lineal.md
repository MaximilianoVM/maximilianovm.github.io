---
title: "Problemas de Programacion Lineal"
date: 2026-08-01 03:47:00 -0700
categories: [Metodos de Optimizacion, Programación Lineal]
tags: [python]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/cimat_1.jpeg
  alt: prog_lineal.
comments: true
---
# 1. Planeación de la producción en una empresa textil

Una empresa textil produce cinco tipos de telas. Cada tela puede tejerse en uno o más de los 38 telares con que cuenta la industria. El Departamento de Ventas ya pronosticó la demanda del próximo mes; ese pronóstico aparece en la **Tabla 1**, junto con el precio de venta, el costo
variable y el precio de compra, todos expresados por metro de tela con un ancho de 140 cm. La empresa opera las 24 horas del dı́a y tiene programado trabajar los 30 dı́as del mes siguiente

TABLA 

La industria cuenta con dos tipos de telares: jacquard y ratier. Los telares jacquard son más versátiles y pueden producir los cinco tipos de tela; los telares ratier solo producen tres de los cinco tipos. En total existen 38 telares: 8 jacquard y 30 ratier. La **Tabla 2** indica la velocidad de tejido de cada tela en ambos tipos de telar. El tiempo requerido para cambiar de
tela no es significativo, por lo que no se toma en cuenta.

La empresa satisface como mı́nimo toda la demanda requerida, ya sea con sus propios tejidos o con telas adquiridas a otra fábrica. Es decir, dadas las limitaciones de capacidad de los telares, las telas que no puedan tejerse en la propia industria se adquirirán a otra fábrica. El precio de compra de cada tela también aparece en la **Tabla 1**.

**a)** Construya un modelo que sirva para programar la producción de esta empresa textil y que además determine cuántos metros de cada tela deben adquirirse a la otra fábrica.


----
## Datos y supuestos  
* 5 tipos de telas: 1, 2, 3, 4, 5
* Se puede producir en 1 o mas de los 38 telares 
* Ancho de 140 cm. 
* Trabajan 24 horas 30 días 
* Dos tipos de telares: 
	* Jacquard
		* produce 5 tipos de tela 
		* tenemos 8 
	* Ratier 
		* Solo produce 3 de los 5 tipos de tela
		* tenemos 30 
* En Tabla II viene la velocidad de tejido por telar por tipo de tela 
* Para satisfacer la demanda podemos comprar tela, al precio indicado en la Tabla II 

## Formulacion matematica 
### Variables de decision
* Tengo 5 tipos de telas que producir y 2 tipos de telares donde lo puedo hacer 
Metros de los distintos tipos de telas producidas por telar (+ las compradas) por mes: 
$$J_1, J_2, J_3, J_4, J_5$$
$$R_3, R_4, R_5$$
$$C_1, C_2, C_3, C_4, C_5$$
en unidades de $[\frac{m}{mes}]$
### Restricciones 
#### Demanda
$$J_1+C_1 \geq 16500$$
$$J_2+C_2 \geq 22000$$
$$J_3+R_3+C_3 \geq 62000$$
$$J_4+R_4+C_4 \geq 7500$$
$$J_5+R_5+C_5 \geq 62000$$
#### Tiempo ($\leq\ 1\ mes$)
Para los 8 telares Jacquard
$$\frac{1}{8}\frac{1}{720}[\frac{1}{4.63}(J_1+J_2) + \frac{1}{5.23}(J_3+J_4) + \frac{J_5}{4.17}] \leq 1 \ \text{mes}$$
Para los 30 telares Ratier
$$\frac{1}{30}\frac{1}{720}[ \frac{1}{5.23} (R_3+R_4) + \frac{R_5}{4.17} ] \leq 1 \ \text{mes}$$

### Función Objetivo
$[\frac{\$}{m}][m]$

$$min\ z = 2.66J_1 + 2.55J_2 + 2.86 C_1 + 2.7C_2 +$$ 
$$2.49[J_3+R_3] + 2.6C_3 + 2.51[J_4+R_4] +$$ 
$$2.7C_4 + 2.5[J_5 + R_5] + 2.7C_5$$
## Implementación 
#### código 
```c
# =============== Variables de decision =============
# Metros de tela especifica producidos por telar / compradas por mes

# metros de tipos de tela producidos por mes en telar Jaqcuard
var xj1 >= 0;
var xj2 >= 0;
var xj3 >= 0;
var xj4 >= 0;
var xj5 >= 0;

# metros de tipos de tela producidos por mes en telar Ratier
var xr3 >= 0;
var xr4 >= 0;
var xr5 >= 0;

# metros de tipos de tela comprados por mes
var xc1 >= 0;
var xc2 >= 0;
var xc3 >= 0;
var xc4 >= 0;
var xc5 >= 0;

# ======= Funcion objetivo =======

# costo variable [$/m] + preciod de compra [$/m] <= precio de venta [$/m]
# * el preciod e venta que acumulamos es fijo porque solo producimos para cubrir la demanda
# solo controlamos el reducir los costos tanto variables como de compra

minimize z: 2.66*xj1 + 2.55*xj2 +2.86*xc1 + 2.7*xc2 + 2.49*(xj3+xr3) + 2.6*xc3 + 2.51*(xj4+xr4) + 2.7*xc4 + 2.5*(xj5+xr5) + 2.7*xc5;

# ======= Demanda =======
subject to c11:   xj1 + xc1 >=  16500;
subject to c12:   xj2 + xc2 >=  22000;
subject to c13:   xj3 + xr3 + xc3 >=  62000;
subject to c14:   xj4 + xr4 + xc4 >=  7500;
subject to c15:   xj5 + xr5 + xc5 >=  62000;

# ======= Tiempo =======

# para 8 talleres jacard
subject to c21:   1/8* 1/720* ( 1/4.63 * (xj1+xj2) + 1/5.23 * (xj3+xj4) + 1/4.17 * xj5 ) <= 1; # menor a un mes


# para 30 talleres ratier
subject to c22:   1/30* 1/720* ( 1/5.23 * (xr3+xr4) + 1/4.17 * xr5 ) <= 1; # menor a un mes


end;
```


## Resultados e interpretación
##### Optimal objective value

$$z = 433\,741.8211031$$

El valor óptimo de la función objetivo obtenido por el solver es **$z = 433,741.8211031$**, lo que representa un costo mínimo total de **$433,741.82** para satisfacer la demanda del mes.

### Plan óptimo de producción y compras

- **Telares Jacquard (8 telares):**
    - Se producen $16,500\text{ m}$ de tela 1 ($xj_1 = 16500$).
    - Se producen $10,168.8\text{ m}$ de tela 2 ($xj_2 = 10168.8$).
    - No se produce tela 3, 4 ni 5 ($xj_3 = xj_4 = xj_5 = 0$).
- **Telares Ratier (30 telares):**
    - Se producen $27,707.8081535\text{ m}$ de tela 3 ($xr_3 = 27707.8081535$).
    - Se producen $7,500\text{ m}$ de tela 4 ($xr_4 = 7500$).
    - Se producen $62,000\text{ m}$ de tela 5 ($xr_5 = 62000$).
- **Compras a fábrica externa:**
    - No se compra tela 1, 4 ni 5 ($xc_1 = xc_4 = xc_5 = 0$).
    - Se compran $11,831.2\text{ m}$ de tela 2 ($xc_2 = 11831.2$).
    - Se compran $34,292.1918465\text{ m}$ de tela 3 ($xc_3 = 34292.1918465$).