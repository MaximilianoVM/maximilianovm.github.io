---
title: Docker para ISIS Image Subtraction p.2: Docker y Contenedor
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

Notas: 
* En Linux Mint 21.3 Cinnamon
* Se asume que ya cuentas con IRAF en tu equipo

# Instalación de Docker
Sigue el tutorial con la propia [documentación de Docker](https://docs.docker.com/engine/install/)

Si te sale un error al hacer el build del tipo: 

> ERROR: permission denied while trying to connect to the Docker daemon socket at unix

Puedes probar con la siguiente solución: 

```bash
sudo chmod 777 /var/run/docker.sock
```

Una vez que hayas concluído el tutorial, puedes comprobar la correcta instalación ejecutando la imagen `hello-world`: 

```bash
sudo docker run hello-world
```

# Construcción de nuestro entorno de trabajo

A este punto ya deberías tener tu archivo `ISIS2.2.tar`, descargado desde la pagina oficial. 

## 1. Estructura de directorios en tu host

Nos vamos a colocar en nuestro entorno de trabajo, donde crearemos dos directorios: 

* **`isis_env_full`**: Aquí van a ir nuestros archivos `Dockerfile` e `ISIS2.2.tar`
* **`prueba_host`**: Aquí es donde vamos a interactuar con el paquete **ISIS**

```bash
mkdir isis_env_full
cd isis_env_full # aqui se debe encontrar el ISIS2.2.tar
mkdir prueba_host  # Aquí verás los archivos de ISIS desde el host, aqui tambien va un  ISIS2.2.tar?
```

## 2. Dockerfile
Creamos nuestro archivo Dockerfile: 
```bash
touch Dockerfile
```

En el cual escribiremos lo siguiente: 
```dockerfile

# Utiliza la imagen oficial de CentOS 6
FROM centos:6

# Reemplaza los repositorios de CentOS 6 por los repos archivados
RUN sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-Base.repo && \
    sed -i 's|^#baseurl=http://mirror.centos.org/centos/$releasever|baseurl=http://vault.centos.org/6.10|g' /etc/yum.repos.d/CentOS-Base.repo

# ====== Instala tcsh, nano y las bibliotecas de 32 bits *
RUN yum install -y tcsh nano glibc.i686

# Instala tcsh, nano, las bibliotecas de 32 bits, vim, awk y emacs
RUN yum install -y tcsh nano glibc.i686 vim awk emacs

# Actualiza los paquetes disponibles
RUN yum update -y

# Instala las herramientas básicas y GCC
RUN yum groupinstall -y "Development Tools" \
    && yum install -y wget csh

# ====== Establece el directorio de trabajo *
WORKDIR /opt/isis

# Copia el archivo tar de ISIS al contenedor (asegúrate de tenerlo en el mismo directorio que tu Dockerfile)
COPY ISIS2.2.tar /opt/isis/

# Extrae el archivo, ejecuta el script install.csh, y cambia permisos si es necesario
RUN tar -xvf ISIS2.2.tar \
    && cd package \
    && csh install.csh \
    && chown -R root:root /opt/isis

# ====== Establece el PATH para incluir los directorios necesarios *
ENV PATH="/opt/isis/package/bin:/opt/isis/package/register:$PATH"

# ====== Cambia el directorio de trabajo *
WORKDIR /opt/isis/package

# ====== Define el comando por defecto para iniciar la shell *
CMD ["/bin/bash"]

```

## 3. BUILD: Primera ejecución

En el mismo directorio `isis_env_full` donde se deben encontrar el `Dockerfile` e `ISIS2.2.tar` vamos a ejecutar el contenedor con los siguientes comandos: 

```bash
docker build -t isis_env_image . 

docker images # para comprobar
```

### Ejecutar (sin volumen)
```bash
docker run -it isis_env /bin/bash
```

# Volumen
```bash
# evita problemas de rutas
docker run -it --name isis_local_env -v $(pwd)/prueba_host:/containter_volume isis_env_image /bin/bash
```
desde el contenedor nos regresamos a `../../..containter_volume/package`

## Permisos
```bash
# desde el prueba_host:
sudo chmod -R a+rwx .
```
## ISIS en el Volume
Añadimos el ISIS y extraemos desde host 
```bash
tar -xvf ISIS2.2.tar # Unpack

# si se puede, solo sale error al principio: 
xed install.csh short.h & 
```
- no corre en `./install`
- si corre el `./process.csh`
- si se pueden abrir desde xgterm los fits





