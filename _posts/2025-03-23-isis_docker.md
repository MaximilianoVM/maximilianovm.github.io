---
title: Contenedor Docker para ISIS Image Subtraction
date: 2025-03-23 21:06:00 -0700
categories: [Astro, ISIS]
tags: [Astronomía]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/whale_eye.jpg
  alt: docker eye
---

### No te corre el `./process.csh` ?

*Esto es un intento de comunicar [mi bitácora](https://veiled-foxtail-58f.notion.site/ISIS-docker-10747b4dc47e809c835ff61c5a42b4bf)*

## ISIS 

Si te has visto en la necesidad o preferencia de usar el paquete de sustracción de imagenes [ISIS Image Subtraction](https://www.iap.fr/useriap/alard/package.html) probablemente no hace falta que te hable de las complicaciónes que esta elección trae consigo. 


ISIS es un paquete (no relacionado con el grupo Estado Islámico de Irak y Siria) desarrollado para identificar variacones en una serie de imagenes. Esto se logra observando los residuos al sustraer una imagen de referencia con una de ciencia. Para que los resultados de la sustracción sean fructiferos, ambas imagenes, idealmente, deben coincidir en todas sus caracteristicas globales, garantizando que los residuos corresponan a una variación real del objeto de interés. 


Esto es especialmente util en Astronomía para la identificación de estrellas variables y asteroides, como se desarrolla en el propio manual oficial. En Astronomía, las catacteristicas globales que necesitamos hacer coincidir corresponden al seeing, posiciones y otras variaciones originadas por la atmosfera y las condiciones de observación.

En este proposito nos ayuda la convolución, especificamente su kernel $K$, caracterizando dichas condiciones en la imagen de ciencia $I$ respecto a la de referencia $R$.

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

No conozco mucha gente que use el programa. Solamente a un optico que conocí en una junta de la Asamblea Estudiantil, para quejarnos del transporte publico. También a otro astronomo, de cuya existencia solamente me cuenta mi asesor, pero nunca se encuentra en el instituto. Escribo esto con la intención de pasarlo a colegas que lo intenten usar, o para que con suerte salga en una busqueda desesperada en google. 

Al optico que menciono le comenté de pura casualidad en lo que en ese entonces yo estaba trabajando, intentando correr ISIS sin exito. Él me mencionó que lo había requerido para un verano de investigación, y coincidió con el desastre que esto implica. En su caso había sido necesario formatearle el disco duro con un sistema algo mas antiguo. Dicha anecdota me abrumó, no me sobran equipos y necesito mantener mi laptop con su configuración actual para una que otra cosa. 

Justo por ese entonces estaba en mis primeros meses como voluntario en el Grupo de Ecología y Conservación de Islas [GECI](https://islas.org.mx/#gsc.tab=0) en el equipo de [ciencia de datos](https://islas.dev/blog/), para liberar mis prácticas por medio de un PVVC. Ahi es de vital importancia el uso de Docker para el flujo de trabajo del día a día, y llevaba un rato viendo tutoriales al respecto. 

## Docker

Para que sirve Docker? Es una herramienta ampliamente utilizada por equipos de desarrollo de software, soluciando problemas de compatibilidad y facilitanto el despliegue de aplicaciones. La cosa es que con ISIS nos encontramos justo con un problema de compatibilidad. El paquete fue hecho a finales de los 90s - principios de los 2000 sin el debido cuidado que el desarrollo de un paquete requiere para su perdurabilidad (ni hablar de su documentación, ese tema requiere otra entrada). 

Como funciona Docker? En terminos prácticos, crea un contenedor a partir de un archivo Dockerfile. Este contenedor representa un espacio con sus propias dependencias (especificadas por nosotros en el Dockerfile) y un entorno muy controlado. En nuestro caso queremos simular un ambiente en el que ISIS pueda correr sin problema. 

No me extenderé mucho sobre cosas que no necesitamos, procuraré brindar ejemplos muy practicos para nuestro caso. Y que al final de este post tu problema esté solucionado. 

<br>
<br>
<br>


```dockerfile
# utiliza la imagen oficial de CentOS 6
FROM centos:6

# reemplaza los repositorios de CentOS 6 por los repos archivados
RUN sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-Base.repo && \
    sed -i 's|^#baseurl=http://mirror.centos.org/centos/$releasever|baseurl=http://vault.centos.org/6.10|g' /etc/yum.repos.d/CentOS-Base.repo

# ====== Instala tcsh, nano y las bibliotecas de 32 bits *
#RUN yum install -y tcsh nano glibc.i686

# Instala tcsh, nano, las bibliotecas de 32 bits, vim y awk
RUN yum install -y tcsh nano glibc.i686 vim awk

# actualiza los paquetes disponibles
RUN yum update -y

# instala las herramientas básicas y GCC
RUN yum groupinstall -y "Development Tools" \
    && yum install -y wget csh

# ====== establece el directorio de trabajo *
WORKDIR /opt/isis

# copia el archivo tar de ISIS al contenedor (asegúrate de tenerlo en el mismo directorio que tu Dockerfile)
#COPY ISIS2.2.tar /opt/isis/

# extrae el archivo, ejecuta el script install.csh, y cambia permisos si es necesario
#RUN tar -xvf ISIS2.2.tar \
#    && cd package \
#    && csh install.csh \
#    && chown -R root:root /opt/isis

# ====== establece el PATH para incluir los directorios necesarios *
#ENV PATH="/opt/isis/package/bin:/opt/isis/package/register:$PATH"

# ====== cambia el directorio de trabajo *
#WORKDIR /opt/isis/package

# ====== define el comando por defecto para iniciar la shell *
CMD ["/bin/bash"]


```
