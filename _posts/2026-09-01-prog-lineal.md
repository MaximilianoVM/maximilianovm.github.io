---
title: "Problemas de Programacion Lineal"
date: 2026-09-01 03:47:00 -0700
categories: [Metodos de Optimizacion, Programación Lineal]
tags: [python]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/cimat_1.jpeg
  alt: prog_lineal.
comments: true
---

## 1. Planeación de la producción en una empresa textil

Una empresa textil produce cinco tipos de telas. Cada tela puede tejerse en uno o más de los 38 telares con que cuenta la industria. El Departamento de Ventas ya pronosticó la demanda del próximo mes; ese pronóstico aparece en la **Tabla 1**, junto con el precio de venta, el costo
variable y el precio de compra, todos expresados por metro de tela con un ancho de 140 cm. La empresa opera las 24 horas del dı́a y tiene programado trabajar los 30 dı́as del mes siguiente


|**Tela**|**Demanda (m)**|**Precio de venta ($/m)**|**Costo variable ($/m)**|**Precio de compra ($/m)**|
|---|---|---|---|---|
|1|16,500|3.99|2.66|2.86|
|2|22,000|3.86|2.55|2.70|
|3|62,000|4.10|2.49|2.60|
|4|7,500|4.24|2.51|2.70|
|5|62,000|3.70|2.50|2.70|

_Tabla 1: Demanda mensual, precio de venta, costo variable y precio de compra de las telas._

La industria cuenta con dos tipos de telares: jacquard y ratier. Los telares jacquard son más versátiles y pueden producir los cinco tipos de tela; los telares ratier solo producen tres de los cinco tipos. En total existen 38 telares: 8 jacquard y 30 ratier. La **Tabla 2** indica la velocidad de tejido de cada tela en ambos tipos de telar. El tiempo requerido para cambiar de
tela no es significativo, por lo que no se toma en cuenta.

La empresa satisface como mı́nimo toda la demanda requerida, ya sea con sus propios tejidos o con telas adquiridas a otra fábrica. Es decir, dadas las limitaciones de capacidad de los telares, las telas que no puedan tejerse en la propia industria se adquirirán a otra fábrica. El precio de compra de cada tela también aparece en la **Tabla 1**.

**a)** Construya un modelo que sirva para programar la producción de esta empresa textil y que además determine cuántos metros de cada tela deben adquirirse a la otra fábrica.


----
### Datos y supuestos
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

### Formulacion matematica 
#### Variables de decision
* Tengo 5 tipos de telas que producir y 2 tipos de telares donde lo puedo hacer 
Metros de los distintos tipos de telas producidas por telar (+ las compradas) por mes: 

$$J_1, J_2, J_3, J_4, J_5$$


$$R_3, R_4, R_5$$

$$C_1, C_2, C_3, C_4, C_5$$

en unidades de $[\frac{m}{mes}]$

#### Restricciones 
##### Demanda
$$J_1+C_1 \geq 16500$$

$$J_2+C_2 \geq 22000$$

$$J_3+R_3+C_3 \geq 62000$$

$$J_4+R_4+C_4 \geq 7500$$

$$J_5+R_5+C_5 \geq 62000$$
##### Tiempo ($\leq\ 1\ mes$)
Para los 8 telares Jacquard

$$\frac{1}{8}\frac{1}{720}[\frac{1}{4.63}(J_1+J_2) + \frac{1}{5.23}(J_3+J_4) + \frac{J_5}{4.17}] \leq 1 \ \text{mes}$$

Para los 30 telares Ratier

$$\frac{1}{30}\frac{1}{720}[ \frac{1}{5.23} (R_3+R_4) + \frac{R_5}{4.17} ] \leq 1 \ \text{mes}$$

#### Función Objetivo
$[\frac{\$}{m}][m]$

$$min\ z = 2.66J_1 + 2.55J_2 + 2.86 C_1 + 2.7C_2 + \\ 2.49[J_3+R_3] + 2.6C_3 + 2.51[J_4+R_4] + \\ 2.7C_4 + 2.5[J_5 + R_5] + 2.7C_5$$

### Implementación 
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm)

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


### Resultados e interpretación
###### Optimal objective value

$$z = 433\,741.8211031$$

El valor óptimo de la función objetivo obtenido por el solver es **$z = 433,741.8211031$**, lo que representa un costo mínimo total de **$433,741.82** para satisfacer la demanda del mes.

#### Plan óptimo de producción y compras

- **Telares Jacquard (8 telares):**
    - Se producen $16,500\text{ m}$ de tela 1.
    - Se producen $10,168.8\text{ m}$ de tela 2.
    - No se produce tela 3, 4 ni 5 ($xj_3 = xj_4 = xj_5 = 0$).
- **Telares Ratier (30 telares):**
    - Se producen $27,707.8081535\text{ m}$ de tela 3.
    - Se producen $7,500\text{ m}$ de tela 4.
    - Se producen $62,000\text{ m}$ de tela 5.
- **Compras a fábrica externa:**
    - No se compra tela 1, 4 ni 5 ($xc_1 = xc_4 = xc_5 = 0$).
    - Se compran $11,831.2\text{ m}$ de tela 2.
    - Se compran $34,292.1918465\text{ m}$ de tela 3.

## 2. Portafolio de inversiones

Saúl Cortés, ingeniero en organización industrial, desea formar su propio portafolio de inversiones con el fin de emplear la mı́nima inversión inicial posible y generar con ella cantidades especı́ficas de capital durante los próximos seis años (él considera del año 1 al año 6). 
El propósito de su análisis de inversión es planear los gastos de su hija Susana cuando ingrese a la universidad, dentro de dos años (año 3). 
Los requerimientos financieros de Saúl se presentan en la **Tabla 3**.

|**Año**|**Capital requerido ($)**|
|---|---|
|3|20,000|
|4|22,000|
|5|24,000|
|6|26,000|

_Tabla 3: Requerimientos financieros de Saúl Cortés._

Las caracterı́sticas de las inversiones entre las que Saúl puede elegir se muestran en la **Tabla 4**.

|**Opción**|**Rentabilidad (%)**|**Vencimiento (años)**|
|---|---|---|
|A|5|1|
|B|13|2|
|C|28|3|
|D|40|4|

_Tabla 4: Características de las inversiones._

Los productos C y D implican riesgo, por lo cual Saúl no quiere destinarles en conjunto más del 20 % de la inversión total.

**a)** Construya un modelo de programación lineal que ayude a Saúl a resolver de manera óptima su problema de inversión.

---

### Datos y supuestos  
* mínima inversión inicial posible
* generar capital específico 
* 6 años -> Empezar a ganar en el año 3
* no invertir mas del 20% de la inversión total en el conjunto C y D

### Formulacion matematica 
### Variables de decision
Inversión en $ por acción por año 

**año 1**


$$A_1, B_1, C_1, D_1$$

$$A_2, B_2, C_2, D_2$$

$$A_3, B_3, C_3, D_3$$

$$A_4, B_4, C_4, D_4$$

$$A_5, B_5, C_5, D_5$$

$$A_6, B_6, C_6, D_6$$

#### Restricciones 
* no invertir mas del 20% de la inversión total en el conjunto C y D
$$\frac{C_i + D_i}{A_i + B_i + C_i + D_i} \leq 0.2$$
* Cuanto dinero tengo por año? 
* **Las igualdades representan un balance de caja: lo que entra en un año debe ser exactamente igual a lo que sale ese mismo año.**
* todo lo que entra este año se reinvierte por completo

Año 1: 

$$A_1 + B_1 + C_1 + D_1\ \text{<- no es restriccion, sera la func. objetivo}$$

Año 2: 

$$1.05 A_1 = A_2 + B_2 + C_2 + D_2$$

Año 3: 

$$1.05A_2 + 1.13B_1 = A_3 + B_3 + C_3 + 20000$$

Año 4: 

$$1.28C_1 + 1.05A_3 + 1.13B_2 = A_4 + B_4 + 22000$$

Año 5: 

$$1.05A_4 + 1.14B_3 + 1.28C_2 + 1.4D_1 = A_5 + 24000$$

Año 6: 

$$1.05A_5 + 1.13B_4 + 1.28C_3 + 1.4D_2 \geq 26000$$

#### Función Objetivo
$$min\ z = A_1 + B_1 + C_1 + D_1$$

### Implementación 
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm)

```c
# =============== Variables de decision =============

# Inversion en $ por accion por anio

# anio 1 - inversion de capíital inicial (minimizar)
var xa1 >= 0;
var xb1 >= 0;
var xc1 >= 0;
var xd1 >= 0;

# anio 2 - 
var xa2 >= 0;
var xb2 >= 0;
var xc2 >= 0;
var xd2 >= 0;

# anio 3 - 
var xa3 >= 0;
var xb3 >= 0;
var xc3 >= 0;
var xd3 >= 0;

# anio 4 - 
var xa4 >= 0;
var xb4 >= 0;
var xc4 >= 0;
var xd4 >= 0;

# anio 5 - 
var xa5 >= 0;
var xb5 >= 0;
var xc5 >= 0;
var xd5 >= 0;

# anio 6 - 
var xa6 >= 0;
var xb6 >= 0;
var xc6 >= 0;
var xd6 >= 0;


# ================================ 
# ======= Funcion objetivo =======
# ================================

# Minmizar el capital inicial
minimize z: xa1 + xb1 + xc1 + xd1;


# ================================ 
# ======== Restricciones =========
# ================================

# ======= Consideracion del reisgo de C y D =======
subject to c11:   (xc1 + xd1) <=  0.2*(xa1+xb1+xc1+xd1);
subject to c12:   (xc2 + xd2) <=  0.2*(xa2+xb2+xc2+xd2);


# ======= Capital por anio =======
# anio 1: capital inicial, es la funcion objetivo

# anio 2
subject to c22:   1.05*xa1 = xa2+xb2+xc2+xd2;
# anio 3
subject to c23:   1.05*xa2 + 1.13*xb1 = xa3+xb3+xc3+ 20000;
# anio 4
subject to c24:   1.05*xa3 + 1.13*xb2 + 1.28*xc1 = xa4+xb4+22000;
# anio 5
subject to c25:   1.05*xa4 + 1.13*xb3 + 1.28*xc2 + 1.4*xd1 = xa5+24000;
# anio 6
subject to c26:   1.05*xa5 + 1.13*xb4 + 1.28*xc3 + 1.4*xd2 = 26000;

end;
```

###### Optimal objective value

$$z = aaaa$$

### Resultados e interpretación

###### Optimal objective value

$$z = 71\,431.5813381$$

El valor óptimo de la función objetivo obtenido por el solver es **$z = 71,431.5813381$**, lo que indica que Saúl Cortés requiere una inversión inicial mínima de **$71,431.58** en el año 1 para poder cubrir todos sus compromisos financieros proyectados del año 3 al año 6.

#### Plan óptimo de inversión
- **Año 1:**
    - Inversión en $A_1$: $21,470.49$
    - Inversión en $B_1$: $35,674.78$
    - Inversión en $C_1$: $1,265.79$
    - Inversión en $D_1$: $13,020.52$
- **Año 2:**
    - Inversión en $A_2$: $0.00$ 
    - Inversión en $B_2$: $18,035.21$ 
    - Inversión en $C_2$: $4,508.80$ 
    - Inversión en $D_2$: $0.00$
- **Año 3:**
    - Inversión en $A_3$: $0.00$ 
    - Inversión en $B_3$: $0.00$ 
    - Inversión en $C_3$: $20,312.50$ 
    - Inversión en $D_3$: $0.00$ 
- **Año 4 a 6:**
    - No se realiza ninguna nueva inversión en los productos $A$, $B$, $C$ ni $D$ durante los años 4, 5 y 6 ($xa_4 = xb_4 = xa_5 = 0$).

#### Análisis del flujo de capital y riesgo

- **Cumplimiento del límite de riesgo:** En el año 1, la suma invertida en los instrumentos riesgosos $C_1 + D_1$ asciende a $\$1,265.79 + \$13,020.52 = \$14,286.31$, lo cual representa exactamente el 20% de la inversión inicial total ($\$71,431.58 \times 0.20 = \$14,286.316$), saturando por completo la restricción impuesta por Saúl.
    
- **Generación de los requerimientos de capital:**
    - _Año 3 (\$20,000 requeridos):_ El retorno proveniente de $B_1$ ($1.13 \times \$35,674.78 = \$40,312.50$) más el de $A_2$ genera un total de \$40,312.50. Se destinan \$20,000 al compromiso de Susana y los \$20,312.50 remanentes se reinvierten en $C_3$.
    - _Año 4 (\$22,000 requeridos):_ El retorno de $C_1$ ($1.28 \times \$1,265.79 = \$1,620.21$) sumado al retorno de $B_2$ ($1.13 \times \$18,035.21 = \$20,379.79$) genera exactamente los \$22,000 requeridos para este periodo.
    - _Año 5 (\$24,000 requeridos):_ El retorno de $C_2$ ($1.28 \times \$4,508.80 = \$5,771.27$) sumado al vencimiento de 4 años del producto $D_1$ ($1.40 \times \$13,020.52 = \$18,228.73$) cubre exactamente los \$24,000 necesarios.
    - _Año 6 ($\ge \$26,000$ requeridos):_ La maduración del producto $C_3$ contratado en el año 3 ($1.28 \times \$20,312.50 = \$26,000$) satisface por completo la meta final de gasto.



## 3. Planeación de la producción en una compañı́a metalúrgica

Un fabricante de una empresa metalúrgica de Frankfurt produce cuatro tipos de productos, que se procesan de manera secuencial en dos máquinas. **La Tabla 5** presenta los detalles técnicos de esta producción.


#### Tabla 5: Detalles de producción del fabricante metalúrgico

|**Máquina**|**Costo por minuto ($)**|**Producto 1**|**Producto 2**|**Producto 3**|**Producto 4**|**Capacidad diaria de producción (min)**|
|---|---|---|---|---|---|---|
|**1**|10|2|3|4|2|500|
|**2**|5|3|2|1|2|380|
|**Precio de venta ($)**|—|65|70|55|45|—|

_Tabla 5: Detalles de producción del fabricante metalúrgico._

**a)** Construya un modelo de programación lineal que optimice la producción diaria del fabricante.


---

### Datos y supuestos  
* 4 tipos de productos 
* Procesados de manera secuencial en 2 maquinas 
* optimizar produccion diaria
### Variables de decisión
unidades del producto i a producir por día 
$$x_1, x_2, x_3, x_4$$
cada una pasa por una maquina y luego la otra. 
### Restricciones 
de tiempo 

$$2x_1 + 3x_2 + 4x_3 + 2x_4 \leq 500\ \text{mins}$$

$[\frac{min}{u}][u]$

$$3x_1 + 2x_2 + 1x_3 + 2x_4 \leq 380\ \text{mins}$$

### Implementación 
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm) 

```c
# =============== Variables de decision =============
# unidades del producto i a producir por dia
var x1 >= 0;
var x2 >= 0;
var x3 >= 0;
var x4 >= 0;

# ======= Funcion objetivo =======
# Maximizar ganancia neta diaria: Ganancia = Precio - (Tiempo M1 * 10 + Tiempo M2 * 5)
maximize z: 30*x1 + 30*x2 + 10*x3 + 15*x4;

# ======= Restricciones =======
subject to c11: 2*x1 + 3*x2 + 4*x3 + 2*x4 <= 500;
subject to c12: 3*x1 + 2*x2 + 1*x3 + 2*x4 <= 380;

end;
```

###### Optimal objective value

$$z = 5\,280$$

El valor óptimo de la función objetivo obtenido por el solver es **$z = 5,280$**, lo que representa una ganancia neta máxima de **$5,280** al día.

#### Plan óptimo de producción diaria
- **Producto 1 ($x_1$):** Producir $28$ unidades por día ($x_1 = 28$).
- **Producto 2 ($x_2$):** Producir $148$ unidades por día ($x_2 = 148$).
- **Producto 3 ($x_3$):** No producir ninguna unidad ($x_3 = 0$).
- **Producto 4 ($x_4$):** No producir ninguna unidad ($x_4 = 0$).

#### Uso de capacidad de las máquinas
- **Máquina 1:** Se utiliza al 100% de su capacidad disponible ($2 \times 28 + 3 \times 148 = 56 + 444 = 500$ minutos de 500 disponibles).
- **Máquina 2:** Se utiliza al 100% de su capacidad disponible ($3 \times 28 + 2 \times 148 = 84 + 296 = 380$ minutos de 380 disponibles).

## 4. Planeación de la producción en una empresa de cosméticos

Una empresa de Milán vende productos quı́micos para cosmética profesional. Dicha empresa
planea la producción de tres productos, GCA, GCB y GCC, para un periodo determinado,
mezclando dos componentes: C1 y C2. Todo producto final debe contener al menos uno de
los dos componentes, pero no necesariamente ambos.

Para el próximo periodo de planeación se dispone de 10,000 litros de C1 y 15,000 litros de
C2. La producción de GCA, GCB y GCC debe programarse de modo que cubra al menos los
niveles mı́nimos de demanda de 6,000, 7,000 y 9,000 litros, respectivamente. Se supone que,
al mezclar los componentes, no hay pérdida ni ganancia de volumen.

Cada componente quı́mico, C1 y C2, tiene una proporción de elemento crı́tico de 0.4 y
0.2, respectivamente; es decir, cada litro de C1 contiene 0.4 litros del elemento crı́tico. Para
obtener GCA, la mezcla debe contener una proporción de al menos 0.3 del elemento crı́tico.
Otro requisito es que la proporción del elemento crı́tico presente en GCB sea a lo más de 0.3.

Además, la proporción mı́nima de C1 respecto de C2 en el producto GCC debe ser 0.3. La
ganancia esperada por la venta de cada litro de GCA, GCB y GCC es de $125, $135 y $155,
respectivamente.

**a)** Construya un modelo de programación lineal que optimice la planeación de la producción
de esta empresa de Milán.


_________________

### Datos y supuestos  
* producción de 3 productos: **GCA, GCB y GCC.**  
* los productos se obtienen mezclando dos componentes: **C1** y **C2**.  
* Todo producto final debe contener **al menos uno de los dos componente**s, pero no necesaria-mente ambos.  
* se dispone de **10,000 litros de C1** y **15,000 litros de C2**.  
* **demanda** (al menos) de 6,000, 7,000 y 9,000 litros para GCA, GCB y GCC respectivamente.  
* al mezclar los componentes, no hay pérdida ni ganancia de volumen.  
* C1 y C2, tienen una proporción de elemento crítico de 0.4 y 0.2, respectivamente (por litro)
* GCA debe contener una proporción de al menos 0.3 del elemento crıtico.  
* la proporción del elemento crıtico presente en GCB sea a lo más de 0.3.  
* la proporción mınima de C1 respecto de C2 en el producto GCC debe ser 0.3
* Ganancias por litro:

| producto | ganancia por litro |
| -------- | ------------------ |
| GCA      | 125                |
| GCB      | 135                |
| GCC      | 155                |


### Formulacion matematica 
#### Variables de decision 
Cuantos litros de cada componente (C1, C2) se destina a cada producto, ya que cada producto es una mezcla. 
* renombramos: $GCA$ -> $A$ ; $GCB$ -> $B$ ; $GCC$ -> $C$

$$C1_A, C1_B, C1_C$$, $$C2_A, C2_B, C2_C$$

en unidades de litros $[L]$

#### Restricciones 
##### Recursos
* se dispone de **10,000 litros de C1** y **15,000 litros de C2**.  

$$C1_A + C1_B + C1_C \leq 10,000 \ l \\ C2_A + C2_B + C2_C \leq 15,000 \ l$$ 

##### Demanda 
* **demanda** (al menos) de 6,000, 7,000 y 9,000 litros para GCA, GCB y GCC respectivamente.  

$$C1_A + C2_A \geq 6,000 \ l$$

$$C1_B + C2_B \geq 7,000 \ l$$

$$C1_C + C2_C \geq 9,000 \ l$$

##### Proporciones
* C1 y C2, tienen una proporción de elemento crítico de $0.4$ y $0.2$, respectivamente (por litro)
		elemento critico en C1 = $0.4*C1$,    $[L]$
		elemento critico en C2 = $0.2*C2$,    $[L]$

* GCA debe contener una proporción de al menos 0.3 del elemento crıtico.  

$$\frac{Elemento\ critico}{Total}=\frac{0.4*C1_A + 0.2*C2_A}{C1_A + C2_A} \geq 0.3$$

* la proporción del elemento crıtico presente en GCB sea a lo más de 0.3.  

$$\frac{Elemento\ critico}{Total}=\frac{0.4*C1_B + 0.2*C2_B}{C1_B + C2_B} \leq 0.3$$

* la proporción mınima de C1 respecto de C2 en el producto GCC debe ser 0.3

$$\frac{elemento\ critico\ en\ C1\ para\ C}{elemento\ critico\ en\ C2\ para\ C} = \frac{C1_C}{C2_C} \geq0.3$$

#### Función Objetivo (En base a ganancias)

| producto | ganancia por litro |
| -------- | ------------------ |
| GCA      | 125                |
| GCB      | 135                |
| GCC      | 155                |


$$\text{GCi producido [L]} = \sum_k Ck_i = C1_A + C2_A$$

Funcion objetivo en unidades de dinero:

$$[\text{L}] \times \left[ \frac{\text{\$}}{\text{L}} \right]$$

$$\max z = 125 \cdot GCA + 135 \cdot GCB + 155 \cdot GCC$$

en terminos de nuestras variables de decisión:

$$\max z = 125 \cdot (C1_A + C2_A) + 135 \cdot (C1_B + C2_B) + 155 \cdot (C1_C + C2_C)$$

### Implementación
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm)

```
var C1_A >= 0;
var C1_B >= 0;
var C1_C >= 0;

var C2_A >= 0;
var C2_B >= 0;
var C2_C >= 0;


maximize z:     125*(C1_A+C2_A) + 135*(C1_B+C2_B) + 155*(C1_C+C2_C);

# recursos
subject to c11:   C1_A + C1_B + C1_C <= 10000;
subject to c12:   C2_A + C2_B + C2_C <= 15000;

# demanda
subject to c21:   C1_A + C2_A >= 6000;
subject to c22:   C1_B + C2_B >= 7000;
subject to c23:   C1_C + C2_C >= 9000;

# proporciones
subject to c31:   0.4*C1_A + 0.2*C2_A >= 0.3*(C1_A + C2_A);
subject to c32:   0.4*C1_B + 0.2*C2_B <= 0.3*(C1_B + C2_B);

subject to c33:                  C1_C >= 0.3*C2_C;

end;

```
#### Resultados 
##### Valor óptimo para la función objetivo: 
$$z= 3,555,000$$
##### Valores óptimos para las variables de decisión: 

| Variable $[Litros\ de\ componente\ dedicado\ a\ producto]$ | Valor $[Litros]$ | Elemento Critico |
| ---------------------------------------------------------- | ---------------- | ---------------- |
| C1_A                                                       | 3730.7692308     |                  |
| C1_B                                                       | 3500             |                  |
| C1_C                                                       | 2769.2307692     |                  |
| C2_A                                                       | 2269.2307692     |                  |
| C2_B                                                       | 3500             |                  |
| C2_C                                                       | 9230.7692308     |                  |

## 5. Planeación de la producción en una industria automotriz


Una planta automotriz ensambla dos tipos de vehı́culos: un **sedán** de cuatro puertas y una
**vagoneta**. Ambos tipos deben pasar por la **planta de pintura** y por la **planta de ensamble**. Si la
planta de pintura se dedicara únicamente a pintar sedanes de cuatro puertas, podrı́a pintar
alrededor de **2,000 vehı́culos por dı́a**, mientras que si solo pintara vagonetas, podrı́a pintar
alrededor de **1,500 vehı́culos por dı́a**. Por otra parte, si la planta de ensamble se dedicara a
un solo tipo de vehı́culo, sedán de cuatro puertas o vagoneta, podrı́a ensamblar alrededor de
**2,200 vehı́culos por dı́a**. Cada vagoneta deja una ganancia promedio de **$3,000**, mientras que
cada sedán de cuatro puertas deja una ganancia promedio de **$2,100**.

a) Utilice programación lineal e indique eCanl plan de producción diaria que maximice la **ganancia**
**diaria** de la planta de ensamble de vehı́culos.

----

## Datos y supuestos  
**Variables**
* Dos tipos de vehiculos: Sedán y Vagoneta 
* Dos plantas: Pintura y Ensamble 
**Capacidades**
* Pintura: 2000 sedanes por día (si solo pinta sedanes) -> tarda 1/2000 dias efectivos en pintar 1 sedan 
* Pintura: 1500 vagonetas por día (si solo pinta vagonetas) -> tarda 1/1500 dias efectivos en pintar 1 vagoneta 
* Ensamble: 2200 vehiculos por día si solo se dedican a uno -> tarda 1/2200 dias efectivos en ensamblar cualquier carro
**Ganancias**
* $3000 por vagoneta 
* $2100 por sedan
## Formulacion matematica 
### Variables de decision 
* ~~Cantidad de determinado vehiculo producido en determinada planta~~ -> cada vehiculo necesita ambos
* No hay variables temporales, solo deben caber en un día
* Cantidad de cada vehiculo producido por día

$$S, V$$

### Restricciones

**uso de planta de pintura**

$$\frac{1}{2000} S +\frac{1}{1500}V \leq 1 \ \text{día efectivo}$$

**uso de planta ensambladora**

$$\frac{1}{2200} (S + V) \leq 1 \ \text{día efectivo} $$

### Función objetivo 
$$max\ z = 3000V + 2100S$$

## Implementación
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm)

```
var S >= 0;
var V >= 0;

maximize z:     3000*V + 2100*S;

# uso de plantas
subject to c11:   (1/2000)*S + (1/1500)*V <= 1;
subject to c12:   (1/2200)*(S+V) <= 1;

end;
```

Optimal objective value

$$z = \$4,500,000 $$

## 6. Cartera de inversiones


ORGASA tiene una cartera de inversiones en acciones, bonos y otros instrumentos alternativos.
Actualmente dispone de $200,000 que deben destinarse a nuevas inversiones. Las cuatro
alternativas que ORGASA está considerando se muestran en la **Tabla 6.**




La medida de riesgo indica la incertidumbre asociada a cada acción en cuanto a su capacidad
para alcanzar el rendimiento anual previsto: a *mayor valor, mayor riesgo.*

ORGASA ha establecido las siguientes condiciones para sus inversiones:

* Regla 1: la tasa anual de rendimiento de la cartera debe ser de al menos 9 %.
* Regla 2: ningún valor puede representar más del 50 % de la inversión total en dólares.

**a)** Use programación lineal para integrar una cartera de inversiones que minimice el riesgo.
**b)** Si la compañı́a ignorara los riesgos implicados y utilizara una estrategia de rendimiento
máximo, ¿cómo se modificarı́a el modelo anterior?


| Detalles financieros | Telefónita | Sankander | Ferrofial | Gamefa |
| :--- | :---: | :---: | :---: | :---: |
| Precio por acción ($) | 100 | 50 | 80 | 40 |
| Tasa de rendimiento anual | 0.12 | 0.08 | 0.06 | 0.10 |
| Medida de riesgo por $ invertido | 0.10 | 0.07 | 0.05 | 0.08 |

*Tabla 6: Datos de las inversiones.*

### Datos y supuestos  
* Dispone de $200000 
* 4 alternativas en Tabla 6.
* Regla 1: la tasa anual de rendimiento de la cartera debe ser de al menos 9 %.
* Regla 2: ningún valor puede representar más del 50 % de la inversión total en dólares.
### Formulacion matematica 
### a) 
#### Variables de decision 

**Dinero invertido por acción** $[ \$ ]$


$$T, S, F, G$$

#### Restricciones
Capital inicial

$$T+S+F+G= \text{\$}200000$$

* Regla 1: la tasa anual de rendimiento de la cartera debe ser de al menos 9 %.

$$\frac{0.12T + 0.08S + 0.06F + 0.10G}{200000} \geq 0.09 $$

* Regla 2: ningún valor puede representar más del 50 % de la inversión total en dólares.

$$T\leq\text{\$}200000/2$$

$$S\leq\text{\$}200000/2$$

$$F\leq\text{\$}200000/2$$

$$G\leq \text{\$}200000/2$$

#### Función objetivo 
Minimizar el reisgo

$$min\ z = 0.10T + 0.07S + 0.05F + 0.08G$$

### Implementación
#### Código
```
var T >= 0, <= 100000;
var S >= 0, <= 100000;
var F >= 0, <= 100000;
var G >= 0, <= 100000;

minimize riesgo: 0.10*T + 0.07*S + 0.05*F + 0.08*G;

subject to capital:    T + S + F + G = 200000;
subject to rendimiento: 0.12*T + 0.08*S + 0.06*F + 0.10*G >= 18000;

end;
```

Optimal objective value

$$ z = 14666.6666667 $$

| Variable | Type | Value |
| :---: | :---: | :---: |
| T | Real | 33333.3333333 |
| S | Real | 0 |
| F | Real | 66666.6666667 |
| G | Real | 100000 |


### b) 
Cambia la funcion objetivo por una que maximice el rendimiento 
$$max\ z = 0.12T + 0.08S + 0.06F + 0.10G$$
y no haría falta la regla 1 ya que sería redundante, maximizar el rendimiento automáticamente garantiza que sea al menos 9%.

### Implementación
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm)
```
var T >= 0, <= 100000;
var S >= 0, <= 100000;
var F >= 0, <= 100000;
var G >= 0, <= 100000;

maximize rendimiento: 0.12*T + 0.08*S + 0.06*F + 0.10*G;

subject to capital: T + S + F + G = 200000;

end;
```

Optimal objective value

$$ z= 22000 $$

Resultados que rroja el solver: 

| Variable | Type | Value |
| :---: | :---: | :---: |
| T | Real | 100000 |
| S | Real | 0 |
| F | Real | 0 |
| G | Real | 100000 |

## 7. Fondos de inversión

Un pequeño inversionista dispone de \$12,000 para invertir y puede elegir entre tres fondos
distintos. Los fondos de inversión garantizados ofrecen una tasa de rendimiento esperada del
7 %; los fondos mixtos, en los que una parte del capital está garantizada, tienen una tasa de
rendimiento esperada del 8 %; mientras que una inversión en la Bolsa de Valores implica una
tasa de rendimiento esperada del 12 %, pero sin capital de inversión garantizado. Con el fin
de minimizar el riesgo, el inversionista ha decidido no invertir más de $2,000 en la Bolsa de
Valores. Además, por razones fiscales, debe invertir al menos tres veces más en fondos de
inversión garantizados que en fondos mixtos. Suponga que al final del año los rendimientos
son los esperados: **¿cuáles son los montos óptimos de inversión?**

**a)** Considere este problema como si fuera un modelo de programación lineal con dos variables
de decisión.
**b)** Resuelva el problema con el método gráfico e indique la solución óptima.

---
### Datos y supuestos  
*  \$12,000 para invertir
* tres fondos distintos.
* Los fondos de inversión **garantizados** ofrecen una tasa de rendimiento esperada del 7 %; 
* los fondos **mixtos**, en los que una parte del capital está garantizada, tienen una tasa de rendimiento esperada del 8 %; 
* La **Bolsa** de Valores implica una tasa de rendimiento esperada del 12 %, pero sin capital de inversión garantizado.
* no invertir más de $2,000 en la Bolsa de Valores.
* invertir al menos tres veces más en fondos de inversión garantizados que en fondos mixtos.
* al final del año los rendimientos son los esperados

|                                | Garantizados | Mixtos    | Bolsa |
| ------------------------------ | ------------ | --------- | ----- |
| **tasa de rentimiento**        | 0.07         | 0.08      | 0.12  |
| **$ de inversión garantizado** | si           | una parte | no    |

### Formulacion matematica 
#### Variables de decision 
Capital \$ a invertir en cada fondo

* Fondos de Inversión Garantizados: $G$
* Fondos Mixtos: $M$
* Bolsa de Valores *(invisible)*: $\text{\$}12000-G-M$

$$G, M$$

#### Restricciones

*  \$12,000 para invertir

  $$G + M \leq \text{\$}12,000$$

 * no invertir más de \$2,000 en la Bolsa de Valores.

  $$\text{\$}12000 - G -M \leq \text{\$} 2,000$$

* invertir al menos tres veces más en fondos de inversión garantizados que en fondos mixtos.

$$\frac{G}{M} \geq 3$$

#### Función objetivo 

¿cuáles son los montos óptimos de inversión?

$$max \ z = 0.07G + 0.08M + 0.12(12000 - G -M)$$

### Implementación
#### Código
```
var G >= 0;
var M >= 0;

# maximizar rendimiento total: 7%G + 8%M + 12%*(Bolsa)
maximize z: 0.07*G + 0.08*M + 0.12*(12000 - G - M);

subject to c_bolsa:     12000 - G - M <= 2000;   # no más de $2000 en Bolsa
subject to c_proporcion: G >= 3*M;                # al menos 3 veces más en garantizados que en mixtos
subject to c_nonneg_bolsa: G + M <= 12000;        # Bolsa no puede ser negativa

end;
```
Optimal objective value

$$ z= 965 $$

| Variable | Type | Value |
| :---: | :---: | :---: |
| G | Real | 7500 |
| M | Real | 2500 |

## 8. Renta de almacenes


Una empresa se ha dado cuenta de que no tendrá suficiente espacio de almacenamiento
durante los próximos **tres meses**. Los requerimientos adicionales de almacenamiento para ese
periodo se muestran en la Tabla 7.

| Mes | Enero | Febrero | Marzo |
| :--- | :---: | :---: | :---: |
| Espacio requerido (1,000 m²) | 25 | 10 | 20 |

*Tabla 7: Requerimientos adicionales de espacio de almacenamiento.*

Para cubrirlos, la empresa planea rentar espacio adicional a corto plazo. **Al inicio de cada**
**mes puede rentar cualquier cantidad de espacio por cualquier número de meses**, y puede
contratar de manera independiente distintas cantidades de espacio con distintas duraciones.
Por ejemplo, durante el primer mes puede rentar 20,000 m2 por dos meses y, además, 5,000
m2 por un mes. También puede contratar nuevas rentas antes de que venzan las anteriores.
Los costos por cada 1,000 m2 de espacio rentado, según la duración del contrato, se muestran
en la Tabla 8.

| Duración de la renta | 1 mes | 2 meses | 3 meses |
| :--- | :---: | :---: | :---: |
| Costo ($ por 1,000 m²) | 280 | 450 | 600 |

*Tabla 8: Costos de renta.*

a) Construya un modelo de programación lineal cuya solución proporcione una polı́tica de
renta que cubra los requerimientos de espacio a un costo mı́nimo.

----
### Datos y supuestos  
*  Al inicio de cada mes puede rentar cualquier cantidad de espacio por cualquier número de meses
* puede contratar de manera independiente distintas cantidades de espacio con distintas duraciones.
* puede contratar nuevas rentas antes de que venzan las anteriores

### Formulacion matematica 

#### Variables de decision

Miles de $m^2$ rentados en un mes dado (Enero, Febrero, Marzo) durante cierta cantidad de meses (1, 2, 3). 
En $[1k \ \ m^2]$:

$$E1, E2, E3$$

$$F1, F2$$

$$M1$$

#### Restricciones
De espacio 
* Enero: lo cubren los contratos que empiezan en enero, de cualquier duración

$$E1+ E2+ E3 \geq 25 \ m^2$$

* Febrero: lo cubren E2 y E3​ (contratos de enero que duran ≥2 meses, así que alcanzan febrero), más los que empiezan en febrero:

$$E2 + E3 + F1 + F2 \geq 10 \ m^2$$

* Marzo: lo cubren E3 (hechos en enero que llegan hasta marzo), F2 de febrero que llega hasta marzo y los iniciados el mismo marzo (solo M1). 

$$E3 + F2 + M1 \geq 20 \ m^2$$

#### Funcion objetivo 
Minimizando costos 

$[1k \ \ m^2] [\frac{\$}{1k \ m^2}]$

$$min \ z = 280(E1 + F1 + M1) + 450(E2 + F2) + 600E3$$

### Implementación

##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm)

```c
var E1 >= 0;
var E2 >= 0;
var E3 >= 0;
var F1 >= 0;
var F2 >= 0;
var M1 >= 0;

minimize z: 280*(E1+F1+M1) + 450*(E2+F2) + 600*E3;

subject to c11:   E1 + E2 + E3 >= 25;
subject to c12:   E2 + E3 + F1 + F2 >= 10;
subject to c13:   E3 + F2 + M1 >= 20;

end;
```

Optimal objective value

$$z = 13000$$

| Variable | Type | Value |
| :---: | :---: | :---: |
| E1 | Real | 15 |
| E2 | Real | 0 |
| E3 | Real | 10 |
| F1 | Real | 0 |
| F2 | Real | 0 |
| M1 | Real | 10 |


## 9. Planeación de la producción para un fabricante de alambre


Una empresa de Valencia fabrica alambre de aluminio y alambre de cobre. Cada kilogramo
de alambre de aluminio requiere 5 kWh de electricidad y 0.25 horas de trabajo, mientras que
cada kilogramo de alambre de cobre requiere 2 kWh de electricidad y 0.5 horas de trabajo. La
producción de alambre de cobre está limitada por la materia prima disponible, que permite
fabricar a lo más 60 kg al dı́a. La electricidad está limitada a 500 kWh diarios y el tiempo de
trabajo, a 40 horas diarias. La ganancia del alambre de aluminio es de $0.25 por kilogramo y
la del alambre de cobre, de $0.40 por kilogramo.

a) Construya un modelo de programación lineal.
b) ¿Qué cantidad de cada alambre deberı́a producirse para maximizar la ganancia y cuál serı́a
esa ganancia?

----
### Datos y supuestos  
* Fabrican: alambres de aluminio y de cobre 
* 1kg de aluminio requiere 5 kWh de electricidad y 0.25 horas de trabajo
* 1kg de cobre requiere 2 kWh de electricidad y 0.5 horas de trabajo
* cobre: fabricar a lo más 60 kg al dı́a
* electricidad limitada a 500 kWh diarios
* electricidad limitada a 40 h diarias
* ganancias: 
	* aluminio -> $0.25 por kilogramo
	* cobre -> $0.40 por kilogramo
* maximizar la ganancia
### Formulacion matematica 
#### Variables de decision
kg de cada alambre a producir por dia 
$$A, C$$
#### Restricciones
Electricidad 
$[\frac{kWh}{kg}][kg]$
$$5A + 2C \leq 500 \ kWh$$

Horas de trabajo 
$[\frac{h}{kg}][kg]$
$$0.25A + 0.5C \leq 40 \ h$$
Materia prima de cobre
$[kg]$
$$C \leq 60 \ kg$$

#### Funcion objetivo 
Minimizando costos 
$[\frac{\$}{kg}][kg]$
$$max \ z = 0.25A + 0.40 C$$

### Implementación
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm)
```c
var x1 >= 0;
var x2 >= 0;

maximize z: 0.25*x1 + 0.40*x2;

subject to c_electricidad: 5*x1 + 2*x2 <= 500;
subject to c_trabajo:      0.25*x1 + 0.5*x2 <= 40;
subject to c_cobre:        x2 <= 60;

end;
```

Optimal objective value
$$z = 36.25$$

![[Pasted image 20260914015648.png]]

![[Pasted image 20260914015712.png]]

## 10. Inversiones mixtas


La empresa Inversiones Internacionales, S.A.U. cuenta con hasta cinco millones de dólares
para invertir en seis opciones posibles. La Tabla 9 muestra las caracterı́sticas de cada una de
ellas. Por experiencia, la compañı́a sabe que no es recomendable destinar más del 25 % del
total a una sola de estas opciones. Además, es necesario invertir al menos el 30 % en metales
preciosos y al menos el 45 % entre créditos comerciales y bonos corporativos. Por último, se
exige que el riesgo global de la cartera no sea mayor que 2.0.

a) Construya un modelo de programación lineal cuya solución indique cuánto deberı́a invertir
la compañı́a en las seis opciones posibles para maximizar la rentabilidad de sus inversiones.

![[Pasted image 20260914103049.png]]

------

### Datos y supuestos
* **Hasta** 5 MDD para invertir 
* En 6 opciones posibles 
* No destinar mas del 25% del total a una sola de las opciones 
* Invertir al menos el 30% en metales preciosos 
* Invertir al menos el 45% entre créditos comerciales y bonos corporativos 
* Se exige que el riesgo total de la cartera no sea mayor que 2.0

Cuanto debería invertir en las 6 opciones 
para 
maximizar la rentabilidad de sus inversiones? 

### Formulacion matematica 
#### Variables de decisión

dolares invertidos en: creditos comerciales, bonos corporativos, acciones en oro, acciones en platino, bonos hipotecarios, prestamos para edificios
$$x_1,…,x_6$$


#### Restricciones

**Presupuesto** 
("hasta cinco millones" → no es obligatorio invertir todo):  

$$ x_1 + x_2 + x_3 + x_4 + x_5 + x_6 \leq 5,000,000$$
**Diversificación máxima** 
(ninguna opción $> 25\%$ del total invertido, el "total" es la suma de lo que si se invierte, no siempre 5M, ya que el presupuesto es "hasta"):  

$$x_i \le 0.25 (x_1 + x_2 + x_3 + x_4 + x_5 + x_6), \quad i=1,\dots,6$$

**Mínimo en metales preciosos** 
oro + platino ≥ 30% del total:  

$$x_3+x_4 \geq 0.30(x1+ \dots +x6)$$

**Mínimo en créditos comerciales + bonos corporativos** 
≥45% del total:  

$$ x1+x2 \geq 0.45 (x1+ \dots +x6) $$

**Riesgo global máximo** 
el riesgo de la cartera es un promedio ponderado por el monto invertido en cada opción — no puede pasar de 2.0:  

$$\frac{1.7x_1+1.2x_2+3.7x_3+2.4x_4+2.0x_5+2.9x_6}{x_1+\dots+x_6} \leq 2.0$$
### Funcion objetivo 
$$max\ z= 0.07x_1 ​+ 0.10x_2 ​+ 0.19x_3 ​+ 0.12x_4 ​+ 0.08x_5​ + 0.14x_6​$$

### Implementacion 
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm) 
```c
var x1 >= 0;   # creditos comerciales
var x2 >= 0;   # bonos corporativos
var x3 >= 0;   # acciones en oro
var x4 >= 0;   # acciones en platino
var x5 >= 0;   # bonos hipotecarios
var x6 >= 0;   # prestamos para edificios

maximize z: 0.07*x1 + 0.10*x2 + 0.19*x3 + 0.12*x4 + 0.08*x5 + 0.14*x6;

subject to c_presupuesto: x1+x2+x3+x4+x5+x6 <= 5000000;

subject to c_max1: x1 <= 0.25*(x1+x2+x3+x4+x5+x6);
subject to c_max2: x2 <= 0.25*(x1+x2+x3+x4+x5+x6);
subject to c_max3: x3 <= 0.25*(x1+x2+x3+x4+x5+x6);
subject to c_max4: x4 <= 0.25*(x1+x2+x3+x4+x5+x6);
subject to c_max5: x5 <= 0.25*(x1+x2+x3+x4+x5+x6);
subject to c_max6: x6 <= 0.25*(x1+x2+x3+x4+x5+x6);

subject to c_metales: x3+x4 >= 0.30*(x1+x2+x3+x4+x5+x6);
subject to c_creditos_bonos: x1+x2 >= 0.45*(x1+x2+x3+x4+x5+x6);

subject to c_riesgo: 1.7*x1+1.2*x2+3.7*x3+2.4*x4+2.0*x5+2.9*x6 <= 2.0*(x1+x2+x3+x4+x5+x6);

end;
```

**Optimal objective value**

$$z = 520000$$
![[Pasted image 20260914104759.png]]

## 11. Planificación del transporte en una empresa productora de aceitunas


Una empresa de Jaén tiene ==tres plantas== productoras de aceitunas, ubicadas en Jaén, Sevilla y
Almerı́a. La capacidad de producción estimada, en kilogramos, para los próximos tres meses se
muestra en la **Tabla 10**.
![[Pasted image 20260916135000.png|517]]

La compañı́a distribuye las aceitunas a través de ==cuatro centros regionales de distribución==,
localizados en Valencia, Madrid, Barcelona y La Coruña. La demanda pronosticada en dichos
centros para los próximos tres meses se presenta en la **Tabla 11**.
![[Pasted image 20260916202540.png|524]]

La administración de la empresa desea ==determinar cuánto de esa producción debe enviarse==
==de cada planta a cada centro de distribución==. El costo unitario, en dólares por kilogramo de
aceituna enviada por cada ruta, se muestra en la **Tabla 12**.
![[Pasted image 20260916202555.png|605]]

* Construya un modelo de programación lineal que ayude a la empresa a tomar esta decisión.
* Por razones estratégicas, la empresa ha adoptado las siguientes polı́ticas para la planificación de su transporte:
	* Al menos el 60 % de la producción total de Jaén debe enviarse a Valencia. (restricción lineal: no requiere variables enteras)
	* Los envı́os de Sevilla a Valencia tendrán un costo fijo de $200. (requiere una variable binaria: el costo fijo se activa solo si se envı́a una cantidad positiva)
	* Solo Sevilla o Almerı́a pueden hacer envı́os a La Coruña, pero nunca ambas. (requiere una variable binaria: condición disyuntiva)

---
### Datos y supuestos
* 3 plantas 
	* con una capacidad de produccion en kg para los prox 3 meses en Tabla 10
* 4 centros de distribucion
	* La demanda pronosticada en dichos centros para los próximos tres meses se presenta en la Tabla 11

Determinar cuánto de esa producción debe enviarse de cada planta a cada centro de distribución
### Formulacion matematica 
#### Variables de decisión
Produccion enviada desde la planta $i$ al centro de distribucion $j$
$$x_{ij} \ \ \ \ \ [kg]$$
Planta $i$:
$$i \ \epsilon \ \{1, 2, 3\} \ \ \ \text{(Jaen, Sevilla, Almeria)} $$

Centros de distribucion $j$:
$$j \ \epsilon \ \{1, 2, 3, 4\} \ \ \ \text{(Valencia, Madrid, Barcelona, Coruña)} $$
#### Restricciones

**Capacidad de produccion**
Por planta $i$, en kg, para los proximos 3 meses
$$x_{1j} = x_{11} + x_{12} + x_{13} + x_{14} \leq 5000$$
$$x_{2j} = x_{21} + x_{22} + x_{23} + x_{24} \leq 6000$$
$$x_{34} = x_{31} + x_{32} + x_{33} + x_{34} \leq 2500$$
**Demanda pronosticada**
Demanda pronosticada en kg en los centros de distribucion $j$ para los próximos tres meses
$$x_{i1} = x_{11} + x_{21} + x_{31} \ge 6000$$
$$x_{i2} \ge 4000$$
$$x_{i3} \ge 2000$$
$$x_{i4} \ge 1500$$
**No negatividad**
$$x_{ij} >= 0 \ \ \ \forall \ \ \ i,j$$
### Funcion objetivo base
Determinar cuanta producción debe enviarse de cada planta a cada centro de distribución dado el costo unitario, en dólares por kilogramo de aceituna enviada.

Es una matriz, podemos asignar $c_{ij}$ como el costo por kg $[\frac{\$}{kg}]$ de enviar el producto de la planta $i$ al centro $j$. Minimizando precios: 

$$min \ z = \sum_{i=1}^3 \sum_{j=1}^4 c_{ij} \ x_{ij}  + 200 y_{sv}$$

#### Restricciones politicas 
* Al menos el 60 % de la producción total de Jaén ($i=1$) debe enviarse a Valencia ($j=1$). (restricción lineal: no requiere variables enteras) 

$$\frac{ \text{enviadas de i=Jaen a j=valencia} }{ \text{enviadas de i=Jaen a todos los j} } \ge 0.6$$
$$\frac{ x_{11} }{ \sum_{j}^4 x_{1j} } \ge 0.6$$

> [!example] Variables Binarias
> Como lo voy a usar? 

* Los envı́os de Sevilla a Valencia tendrán un costo fijo de $200. (requiere una [[variable binaria]]: el costo fijo se activa solo si se envı́a una cantidad positiva) 
	* variable binaria nueva: $y_{sv}$
    * restriccion:

    $$x_{21} \le 6000 \cdot y_{sv}$$

    nuevo termino en la funcion objetivo:

    $$+ 200 y_{sv}$$
    donde $y_{sv} \in \{0, 1\}$

> [!example] Variables Binarias:  Condición disyuntiva
> Como lo voy a usar? 

* Solo Sevilla o Almerı́a pueden hacer envı́os a La Coruña, pero nunca ambas. (requiere una variable binaria: [[condición disyuntiva]])
		* En la funcion objetivo no creamos $x_{14}$ o hacemos restricción $x_{14}=0$ 
		* Nueva variable: 
			$$z \ \epsilon \ \{ 0,1 \} $$
		* Restricciones: 
		$$x_{24} \leq 1500z \ \ \ , \ \ \ x_{34} \leq 1500(1-z)$$


### Implementacion 
##### código en [Solver Online](https://online-optimizer.appspot.com/?model=builtin:default.modm) 
```c
var x11 >= 0; var x12 >= 0; var x13 >= 0; var x14 >= 0;
var x21 >= 0; var x22 >= 0; var x23 >= 0; var x24 >= 0;
var x31 >= 0; var x32 >= 0; var x33 >= 0; var x34 >= 0;

var y_sv binary;
var z binary;

minimize costo: 30*x11+20*x12+70*x13+60*x14
              + 70*x21+50*x22+20*x23+30*x24
              + 20*x31+50*x32+40*x33+50*x34
              + 200*y_sv;

subject to cap_jaen:    x11+x12+x13+x14 <= 5000;
subject to cap_sevilla: x21+x22+x23+x24 <= 6000;
subject to cap_almeria: x31+x32+x33+x34 <= 2500;

subject to dem_valencia:  x11+x21+x31 >= 6000;
subject to dem_madrid:    x12+x22+x32 >= 4000;
subject to dem_barcelona: x13+x23+x33 >= 2000;
subject to dem_coruna:    x14+x24+x34 >= 1500;

subject to politica1: x11 >= 0.6*(x11+x12+x13+x14);
subject to enlace_ysv: x21 <= 6000*y_sv;
subject to enlace_z1:  x24 <= 1500*z;
subject to enlace_z2:  x34 <= 1500*(1-z);
subject to jaen_coruna: x14 = 0;

end;
```

**Optimal objective value**

$$ z = 395000 $$


![[Pasted image 20260917021252.png]]
![[Pasted image 20260917021310.png]]