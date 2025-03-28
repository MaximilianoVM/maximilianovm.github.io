---
title: Contenedor Docker para ISIS Image Subtraction p.2
date: 2025-03-28 15:02:00 -0700
categories: [Astro, ISIS]
tags: [Astro, ISIS, Docker]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/whale_eye.jpg
  alt: docker eye
---

### No te corre el `./process.csh` ?

*Esto es un intento de comunicar [mi bitácora](https://veiled-foxtail-58f.notion.site/ISIS-docker-10747b4dc47e809c835ff61c5a42b4bf)*
* _desde IA-UNAM Ensenada._
## 1. Estructura de directorios en tu host

```bash
mkdir isis_project
cd isis_project
mkdir -p host_package  # Aquí verás los archivos de ISIS desde el host
```

## 2. Dockerfile (optimizado para persistencia)

```dockerfile

# Usa CentOS 6
FROM centos:6

# Configura repositorios archivados
RUN sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-Base.repo && \
    sed -i 's|^#baseurl=http://mirror.centos.org/centos/$releasever|baseurl=http://vault.centos.org/6.10|g' /etc/yum.repos.d/CentOS-Base.repo

# Instala dependencias
RUN yum install -y tcsh nano glibc.i686 vim awk emacs wget csh && \
    yum groupinstall -y "Development Tools" && \
    yum update -y

# Directorio de trabajo
WORKDIR /isis_docker

# Copia el tar de ISIS
COPY ISIS2.2.tar /isis_docker/

# Instala ISIS (durante el build)
RUN tar -xvf ISIS2.2.tar && \
    cd package && \
    csh install.csh && \
    chown -R root:root /isis_docker

# Ruta al ejecutable
ENV PATH="/isis_docker/package/bin:/isis_docker/package/register*:$PATH"

# Punto de montaje para el volumen
VOLUME /isis_docker/package

# Comando por defecto
CMD ["/bin/bash"]

```

## 3. Flujo de trabajo

```bash
# Construye la imagen
docker build -t isis_image .

docker images # para comprobar

# Ejecuta el contenedor y copia los archivos de ISIS a tu host
docker run --name isis_temp isis_image true  # Contenedor efímero
docker cp isis_temp:/isis_docker/package ./host_package
docker rm isis_temp  # Elimina el contenedor temporal
```

## Ejecución normal (con volumen persistente):


```bash
docker run -it --name isis_container \
  -v $(pwd)/host_package:/isis_docker/package \
  isis_image
```






