---
title: Contenedor Docker para ISIS Image Subtraction p.1
date: 2025-03-23 21:06:00 -0700
categories: [Astro, ISIS]
tags: [Astro, ISIS, Docker]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/whale_eye.jpg
  alt: docker eye
---

### No te corre el `./process.csh` ?

*Esto es un intento de comunicar [mi bitácora](https://veiled-foxtail-58f.notion.site/ISIS-docker-10747b4dc47e809c835ff61c5a42b4bf)*

* _Desde OAN-SPM, [Telescopio de 2.1m](https://www.astrossp.unam.mx/es/usuarios/telescopios/tel2m)_

## ISIS 

Si te has visto en la necesidad o preferencia de usar el paquete de sustracción de imagenes [ISIS Image Subtraction](https://www.iap.fr/useriap/alard/package.html) probablemente no hace falta que te hable de las complicaciónes que esta elección trae consigo. 


ISIS es un paquete (no relacionado con el grupo Estado Islámico de Irak y Siria) desarrollado para identificar variacones en una serie de imagenes. Esto se logra observando los residuos al sustraer una imagen de referencia con una de ciencia. Para que los resultados de la sustracción sean fructiferos, ambas imagenes, idealmente, deben coincidir en todas sus caracteristicas globales, garantizando que los residuos corresponan a una variación real del objeto de interés. 


Esto es especialmente util en Astronomía para la identificación de estrellas variables y satélites, como se desarrolla en el propio manual oficial. En Astronomía, las catacteristicas globales que necesitamos hacer coincidir corresponden al seeing, posiciones y otras variaciones originadas por la atmosfera y las condiciones de observación.

En este proposito nos ayuda la convolución, especificamente su kernel $K$, que caracteriza dichas condiciones en la imagen de ciencia $I$ respecto a la de referencia $R$.

$$
  I \approx R \otimes K + B_{bg}
$$

$B_{bg}$: background

la imagen de sustracción $D$: 

$$
  D = I - ( R \otimes K + B_{bg})
$$

Pero todo esto se explica en el manual. El primer problema con el que nos encontramos viene al intentar seguir sus pasos.

## Marco Histórico  

No sé de muchas personas que usen el programa. Solamente a un optico que conocí en una junta de la Asamblea Estudiantil, para quejarnos del transporte publico. También he oído de otro astronomo en el instituto, pero nunca se encuentra cuando lo he buscado. Escribo esto con la intención de pasarlo a colegas que lo intenten usar, o para que con suerte salga en una busqueda desesperada en google. 

Al optico que menciono le comenté de pura casualidad en lo que en ese entonces yo estaba trabajando, intentando correr ISIS sin exito. Él me mencionó que lo había requerido para un verano de investigación, y coincidió con el desastre que esto implica. En su caso había sido necesario formatearle el disco duro con un sistema algo mas antiguo. Dicha anecdota me abrumó un poco, solo seguí intentando correr el programa en mi laptop con su configuración actual esperando que no fuera necesario llegar a ese extremo. 

Justo por ese entonces estaba en mis primeros meses como voluntario en el Grupo de Ecología y Conservación de Islas [GECI](https://islas.org.mx/#gsc.tab=0) en el equipo de [ciencia de datos](https://islas.dev/blog/), para liberar mis prácticas por medio de un PVVC. Ahi es de vital importancia el uso de Docker para el flujo de trabajo del día a día, y llevaba un rato viendo tutoriales al respecto. 

## Docker

Para que sirve Docker? Es una herramienta ampliamente utilizada por equipos de desarrollo de software, soluciando problemas de compatibilidad y facilitanto el despliegue de aplicaciones. 
La cosa es que con ISIS nos encontramos justo con un problema de compatibilidad. El paquete fue hecho a finales de los 90s - principios de los 2000 sin el debido cuidado que el desarrollo de un paquete requiere para su perdurabilidad (ni hablar de su documentación, ese tema requiere otra entrada). 

Como funciona Docker? En terminos prácticos, crea un contenedor a partir de un archivo Dockerfile. Este contenedor representa un espacio con sus propias dependencias (especificadas por nosotros) y un entorno muy controlado. En nuestro caso queremos simular un ambiente en el que ISIS pueda correr sin problema. 

No me extenderé mucho sobre cosas que no necesitamos, procuraré brindar ejemplos muy practicos para nuestro caso. Y que al final de esta serie de posts tu problema esté solucionado. 

<br>
<br>
