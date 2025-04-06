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

* **`isis_workspace`**: Aquí van a ir nuestros archivos `Dockerfile` e `ISIS2.2.tar`
* **`isis_host`**: Aquí es donde vamos a interactuar con el paquete **ISIS**

```bash
mkdir isis_workspace
cd isis_workspace # aqui se debe encontrar el ISIS2.2.tar
mkdir isis_host  # Aquí verás los archivos de ISIS desde el host
cp ISIS2.2.tar isis_host/ # Hacemos una copia del ISIS2.2.tar en el isis_host
```

## 2. 🐳 Dockerfile
Creamos nuestro archivo `Dockerfile` en `isis_workspace`: 
```bash
touch Dockerfile
```

Accedemos al Dockerfile y escribimos lo siguiente: 
```dockerfile
# Dockerfile para configuración de ISIS en CentOS 6
# Nota: Este sistema utiliza una versión antigua de CentOS para su compatibilidad con ISIS

# Utiliza la imagen oficial de CentOS 6
FROM centos:6

# Reemplaza los repositorios de CentOS 6 por los repos archivados (ya no existen de la manera usual)
RUN sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-Base.repo && \
    sed -i 's|^#baseurl=http://mirror.centos.org/centos/$releasever|baseurl=http://vault.centos.org/6.10|g' /etc/yum.repos.d/CentOS-Base.repo

# Instala paquetes y herramientas necesarias para nuestro flujo
RUN yum install -y \
    tcsh \
    nano \
    glibc.i686 \
    vim \
    awk \
    emacs && \
    yum update -y && \
    yum groupinstall -y "Development Tools" && \
    yum install -y \
    wget \
    csh

# Configuración del entorno ISIS
WORKDIR /isis
COPY ISIS2.2.tar /isis/

# Extracción del paquete ISIS
RUN #tar -xvf ISIS2.2.tar && \
    #cd package && \
    #csh install.csh && \
    chown -R root:root /isis

# Variables de entorno críticas (no modificar)
ENV PATH="/isis/package/bin:/isis/package/register:$PATH"

# Comando por defecto (shell interactiva)
CMD ["/bin/bash"]
```

A este punto, la estructura de tus directorios debería verse así: 

![isis_workspace_tree](/assets/img/isis_workspace_tree.png){: width="972" height="589" .w-50 .left}


## 3. 👷🏾‍♀️ BUILD: Primera ejecución

En el mismo directorio `isis_workspace` donde se deben encontrar el `Dockerfile` e `ISIS2.2.tar` vamos a construir la imagen con los siguientes comandos: 

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
docker run -it --name isis_local_env -v $(pwd)/isis_host:/isis isis_env_image /bin/bash
```
Esta acción: 
* crea un contenedor llamado `isis_local_env`
* convierte nuestro directorio local `isis_host`, al host local del contenedor
* Todo el contenido de `/isis` estará en nuestro host dentro de `isis_host`.
* hace todo esto a partir de la imagen `isis_env_image`

Recapitulando: **`isis_host`** e **`isis/`** comparten contenido, en local y en el contenedor respectivamente.

## 🏗️ ISIS en el Volume
Si ya te encuentras en **`/isis`** notarás que únicamente contiene el archivo `ISIS2.2.tar`, hace falta extraerlo y realizar la instalación, la extracción la puedes hacer ya sea desde el host o desde el container: 

Extraemos: 
```bash
tar -xvf ISIS2.2.tar # Unpack
```
Es muy importante que la instalación la hagamos desde el contenedor `/isis`: 
```bash
cd package 
yes | csh install.csh
```

## ✔️ Permisos
Puede que te encuentres con un problema de permisos al intentar modificar o acceder a un archivo desde del host local, esto se soluciona con el siguiente comando: 

```bash
# desde el isis_host:
sudo chmod -R a+rwx .
```
* Puede que te encuentres con avisos de permisos varias veces, pero siempre se soluciona con el comando anterior.


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

# salir del contenedor 
exit 

#detener el contenedor
docker stop <nombre_contenedor> 
```

### Resultado:

- Todo el contenido de `/isis` (incluyendo `package`, binarios y demás) estará **en tu host** dentro de `isis_host`.
- Los cambios que hagas dentro o fuera del contenedor estarán sincronizados.
- No vas a perder nada al hacer `exit`.

## Que podemos hacer desde el contenedor?
- ✅ Sí corre el `./install`.
- ✅ Sí corre el `./process.csh` y el resto de instrucciones sin ningun problema. 
- ✅ Sí se pueden abrir en el el host los **.fits** generados en el contenedor.





