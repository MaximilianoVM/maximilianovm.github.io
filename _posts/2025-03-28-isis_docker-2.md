---
title: "Docker para ISIS Image Subtraction p.2: Docker y Contenedor"
date: 2025-03-28 15:02:00 -0700
categories: [Astro, ISIS]
tags: [Astro, ISIS, Docker]    # TAG names should always be lowercase
math: true
image:
  path: assets/img/whale_eye.jpg
  alt: docker eye
comments: true
---

### No te corre el `./process.csh` ?

*Esto es un intento de comunicar [mi bitácora](https://veiled-foxtail-58f.notion.site/ISIS-docker-10747b4dc47e809c835ff61c5a42b4bf)*
* _desde IA-UNAM Ensenada._

Notas: 
* 🍂 En Linux Mint 21.3 Cinnamon
* 🌌 Se asume que ya cuentas con IRAF en tu equipo

# 🐋 Instalación de Docker
Sigue el tutorial con la propia [documentación de Docker](https://docs.docker.com/engine/install/)


🔴 Si te sale un error al hacer el build del tipo: 

> ERROR: permission denied while trying to connect to the Docker daemon socket at unix

Puedes probar con la siguiente solución: 

```bash
sudo chmod 777 /var/run/docker.sock
```

Una vez que hayas concluído el tutorial, puedes comprobar la instalación ejecutando la imagen `hello-world`: 

```bash
sudo docker run hello-world
```

# 🧑🏾‍💻 Construcción de nuestro entorno de trabajo

A este punto ya deberías tener tu archivo `ISIS2.2.tar`, descargado desde la [pagina oficial](https://www.iap.fr/useriap/alard/download.html). 

## 1. 🗃️ Estructura de directorios en tu host

Nos vamos a colocar en nuestro entorno de trabajo, donde crearemos dos directorios: 

* **`isis_workdir`**: Aquí van a ir nuestros archivos `Dockerfile` e `ISIS2.2.tar`
* **`isis_host`**: Aquí es donde vamos a interactuar con el paquete **ISIS**

```bash
mkdir isis_workdir
cd isis_workdir # aqui se debe encontrar el ISIS2.2.tar
mkdir isis_host  # Aquí verás los archivos de ISIS desde el host, aqui tambien va un  ISIS2.2.tar?
```

## 2. 🐳 Dockerfile
Creamos nuestro archivo Dockerfile: 
```bash
touch Dockerfile
```

Accedemos al Dockerfile y escribimos lo siguiente: 
```dockerfile
# Utiliza la imagen oficial de CentOS 6
FROM centos:6

# Reemplaza los repositorios de CentOS 6 por los repos archivados
RUN sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-Base.repo && \
    sed -i 's|^#baseurl=http://mirror.centos.org/centos/$releasever|baseurl=http://vault.centos.org/6.10|g' /etc/yum.repos.d/CentOS-Base.repo

# Instala tcsh, nano, las bibliotecas de 32 bits, vim, awk y emacs
RUN yum install -y tcsh nano glibc.i686 vim awk emacs

# Actualiza los paquetes disponibles
RUN yum update -y

# Instala las herramientas básicas y GCC
RUN yum groupinstall -y "Development Tools" \
    && yum install -y wget csh

# ====== Establece el directorio de trabajo *
WORKDIR /isis

# Copia el archivo tar de ISIS al contenedor (asegúrate de tenerlo en el mismo directorio que tu Dockerfile)
COPY ISIS2.2.tar /isis/

# Extrae el archivo, ejecuta el script install.csh, y cambia permisos si es necesario
RUN tar -xvf ISIS2.2.tar \
    && cd package \
    && csh install.csh \
    && chown -R root:root /isis
    
# ====== Establece el PATH para incluir los directorios necesarios *
ENV PATH="/isis/package/bin:/isis/package/register:$PATH"

# ====== Cambia el directorio de trabajo *
WORKDIR /isis/package

# ====== Define el comando por defecto para iniciar la shell *
CMD ["/bin/bash"]
```

## 3. 👷🏾‍♀️ BUILD: Primera ejecución

En el mismo directorio `isis_workdir` donde se deben encontrar el `Dockerfile` e `ISIS2.2.tar` vamos a ejecutar el contenedor con los siguientes comandos: 

* Nombraremos a nuestra imagen: `isis_env_image`

```bash
docker build -t isis_env_image . 

# para comprobar: 
docker images # despliega una lista de nuestras imagenes
```
Ya tenemos nuestra imagen, a partir de esta es de donde vamos a poder crear contenedores con el comando `run`. 

La forma mas simple de crear un contenedor a partir de nuestra imagen es la siguiente: 

### 🏃🏾‍♂️ Ejecutar (sin volumen)
```bash
docker run -it isis_env_image /bin/bash
```
Esto creará un contenedor de acuerdo a las instrucciones que indicamos en el `Dockerfile`, al cual podremos acceder para trabajar dentro de su entorno. 

La cosa con esto es que los archivos generados dentro del contenedor no se encontrarán disponibles tan facil desde nuestro entorno local. Incluso, añadir archivos al entorno requiere el mismo nivel de molestia, lo cual es de especial preocupación si nuestro flujo de trabajo requiere la manipulación de distintos grupos de imagenes, que necesitaremos visualizar y procesar con IRAF en distintas partes del proceso. 

Una solucion algo tediosa (que es lo que yo solía hacer al principio) implica copiar archivos hacia y desde el directorio con el [comando cp de docker](https://docs.docker.com/reference/cli/docker/container/cp/), algo parecido al de bash. 

```docker
docker container cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|- docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

```

Sin embargo, hay una manera de garantizar que los archivos existan y sean accesibles de forma local y en el contenedor, de forma que cada cambio en uno sea visible inmediatamente en el otro. Esto se logra mediante [Volumenes](https://docs.docker.com/engine/storage/volumes/). 

# 🧑🏾‍💻🤝🏽🐳 Volumen

### 🏃🏾‍♂️ Ejecutar (en volumen)
```bash
docker run -it --name isis_local_env -v $(pwd)/isis_host:/isis_container isis_env_image /bin/bash
```
Esta acción: 
* crea un contenedor llamado `isis_local_env`
* crea una directorio local `isis_host`, que será el host local
* crea un directorio dentro del contenedor, llamado `isis_container`
* hace todo esto a partir de la imagen `isis_env_image`

Recapitulando: **`isis_host`** e **`isis_container`** comparten contenido, en local y en el contenedor respectivamente.

**Donde se encuentra isis_container?**
Al iniciar el contenedor te vas a encontrar en un directorio que contiene al paquete ISIS, pero aqui no es donde vamos a trabajar.
Desde donde iniciamos en el contenedor nos regresamos a `../../isis_container`.

## 🏗️ ISIS en el Volume
Si ya te encuentras en **`isis_container`** notarás que está vacío, hace falta añadir ahí el `ISIS2.2.tar` y extraerlo, esto lo puedes hacer ya desde el host o desde el container: 

Añadimos el ISIS y extraemos desde host 
```bash
tar -xvf ISIS2.2.tar # Unpack

# archivos de configuracion
xed install.csh short.h & 
```

## ✔️ Permisos
Puede que te encuentres con un problema de permisos al intentar modificar o acceder a un archivo desde del host local, esto se soluciona con el siguiente comando: 

```bash
# desde el isis_host:
sudo chmod -R a+rwx .
```

# ✨💼 Comandos y flujo de trabajo en el día a día
La mayoria de pasos descritos anteriormente solo se necesitan realizar una vez. 
Ya que tengas tu imagen y tu contenedor, unicamente sera necesario acceder a el para seguir trabajando. 

```bash
# listar los contenedores activos 
docker ps
# listar todos los contenedores
docker ps -a 

# iniciar un contenedor 
docker start <nombre_contenedor> 

# acceder a un contenedor en ejecucion	
docker exec -it <contenedor_id> /bin/bash
```

## Que podemos hacer desde el contenedor?
- 🔴 No corre el `./install` pero no es necesario ya que los ejecutables se encuentran desde un inicio en la carpeta `bin/`.
- ✅ Sí corre el `./process.csh` y el resto de instrucciones sin ningun problema. 
- ✅ Sí se pueden abrir los fits desde el host en xgterm.





