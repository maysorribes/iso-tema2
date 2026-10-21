# Principales acciones con Docker

En la [vista rápida del comando `docker`](index.md#vista-rapida-del-comando-docker) vimos cómo buscar y descargar imágenes y cómo crear, listar y borrar contenedores. En este apartado profundizamos en esas acciones y añadimos otras nuevas: dónde guarda Docker la información, cómo parar, inspeccionar y renombrar contenedores, ejecutar comandos dentro de ellos, copiar ficheros, conectarse a sus procesos y consultar sus logs. Al terminarlo ya podrás usar Docker con cierta soltura.

!!! note "¿Y una interfaz gráfica?"
    Existen herramientas para gestionar Docker desde una interfaz gráfica, que hacen la tarea más visual e intuitiva. Aunque pueden ser muy útiles, mientras aprendemos pueden hacer que se nos escape cómo funciona Docker por dentro. Por eso **trabajaremos siempre desde la línea de comandos**.

## Imágenes y contenedores

### ¿Qué es una imagen y un contenedor?

Antes de empezar conviene tener claros estos dos conceptos:

- **Imágenes**
    - Una imagen es una **plantilla de solo lectura** que se utiliza para crear contenedores. A partir de una misma imagen pueden crearse muchos contenedores.
    - Además de su sistema de ficheros, una imagen incluye una serie de parámetros predefinidos (comando de arranque, variables de entorno, etc.) con valores por defecto que se pueden personalizar al crear el contenedor.
    - Docker permite crear imágenes nuevas a partir de otras. Una imagen está formada por un conjunto de **capas** que han ido modificando una imagen base: al crear una imagen nueva, simplemente añadimos una capa encima de la anterior.
- **Contenedores**
    - Son **instancias de una imagen**.
    - Se pueden crear, arrancar, parar y eliminar.
    - Cada contenedor tiene un **identificador único de 64 caracteres**, aunque normalmente se usa la versión corta con los 12 primeros. Los comandos de Docker aceptan las dos.

!!! tip "Un símil"
    Piensa en la instalación de una distribución de Linux desde un DVD: el DVD sería la **imagen** y el sistema operativo ya instalado sería el **contenedor**. Con el mismo DVD puedes instalar tantos equipos como quieras.

### ¿Dónde se almacenan imágenes, contenedores y datos?

El lugar donde se guardan imágenes y contenedores depende del sistema operativo, del driver de almacenamiento y de la versión de Docker. Para consultarlo usamos:

```bash
docker info
```

Este comando muestra información sobre el estado de Docker. Para saber dónde se guarda todo nos interesan dos datos: el **driver de almacenamiento** y el **directorio de Docker**.

Driver de almacenamiento:

```text
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
```

En este ejemplo el driver es `overlay2` y funciona sobre un sistema de ficheros `extfs` (en concreto, `ext4`). Puedes saber más sobre los distintos drivers en la [documentación de Docker](https://docs.docker.com/engine/storage/drivers/select-storage-driver/).

Directorio de Docker:

```text
Docker Root Dir: /var/lib/docker
```

Es el directorio donde se almacena todo lo relacionado con Docker. Con el driver `overlay2`:

- Las **capas de las imágenes** están en `/var/lib/docker/overlay2`.
    - Recuerda que una imagen se compone de capas.
    - Los cambios que hacemos dentro de un contenedor se guardan como una capa más, de lectura y escritura, encima de las capas de la imagen.
- La **configuración de los contenedores** está en `/var/lib/docker/containers`.
- Para los datos compartidos o persistentes existen los **volúmenes**, que veremos más adelante.

!!! warning "Instalaciones recientes de Docker"
    En las versiones más nuevas de Docker Engine, las instalaciones desde cero usan por defecto el *containerd image store*. En ese caso `docker info` puede mostrar otro driver (por ejemplo `overlayfs` con `driver-type: io.containerd.snapshotter.v1`) y las capas de las imágenes se guardan bajo `/var/lib/containerd` en vez de en `/var/lib/docker/overlay2`. La idea de imágenes formadas por capas es exactamente la misma.

## Registro: Docker Hub

[Docker Hub](https://hub.docker.com/) es la plataforma de registro de Docker. Sus servicios básicos son gratuitos y permite publicar imágenes Docker, públicas o privadas.

Contiene un gran ecosistema de imágenes ya creadas, normalmente con instrucciones de instalación y uso, y un [buscador](https://hub.docker.com/search) para encontrarlas según distintos criterios.

Docker utiliza Docker Hub como **registro por defecto**, aunque se puede usar otro registro e incluso montar uno privado (por ejemplo, con la imagen oficial [`registry`](https://hub.docker.com/_/registry)).

## Crear y arrancar contenedores con `docker run`

### ¿Qué hace `docker run`?

Es seguramente el comando de Docker más utilizado: **crea un contenedor a partir de una imagen y lo arranca**.

!!! note "Importante"
    Un error muy común es pensar que `docker run` solo arranca contenedores. **Cada vez que haces `docker run` estás creando un contenedor nuevo**, no arrancando otra vez el mismo.

La descripción completa del comando está en la [documentación de `docker run`](https://docs.docker.com/reference/cli/docker/container/run/). Más abajo veremos sus parámetros más importantes.

### Crear contenedores sin arrancarlos

Para crear un contenedor sin arrancarlo existe el comando `docker create` ([documentación](https://docs.docker.com/reference/cli/docker/container/create/)).

### Repasando el «Hello World»

Al [instalar Docker](index.md#instalacion-de-docker-en-ubuntu) comprobamos que funcionaba con `docker run hello-world`. Veamos con detalle qué pasó en ese momento. La documentación de esta imagen está en [Docker Hub](https://hub.docker.com/_/hello-world).

#### Parte 1: se obtiene la imagen

La primera vez que se ejecuta, la salida empieza así:

```text
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:89b647c604b2a436fc3aa56ab1ec515c26b085ac0c15b0d105bc475be15738fb
Status: Downloaded newer image for hello-world:latest
```

Nos indican que la imagen `hello-world:latest` no está en nuestro sistema, así que se descarga del registro por defecto (Docker Hub) y se guarda localmente. Si volvemos a ejecutar `docker run hello-world`, **este texto ya no aparece**, porque la imagen ya está almacenada en local.

Aunque solo hemos escrito `hello-world`, se ha descargado `hello-world:latest`. Cada imagen tiene una **etiqueta** (*tag*) que indica su versión: si no indicamos ninguna, o indicamos `latest`, se usa la última. Para usar una versión concreta se escribe `imagen:etiqueta`.

#### Parte 2: se crea el contenedor y ejecuta un comando

Con la imagen descargada, se crea el contenedor, se arranca y ejecuta un proceso. Ese proceso lo podemos indicar en la propia orden `docker run`; si no lo indicamos, como en este caso, se ejecuta el comando predefinido de la imagen.

Aquí se ejecuta un programa llamado `hello`, que muestra por la salida estándar (en inglés) cómo ha generado Docker el mensaje: el cliente de Docker se ha conectado con el servicio (*daemon*), este ha descargado la imagen de Docker Hub (o la ha usado en local si ya la tenía), ha creado un contenedor cuyo comando por defecto genera el texto que leemos y, por último, ha enviado esa salida a nuestra terminal.

!!! tip "Curiosidad"
    El código fuente del programa `hello` está disponible en [GitHub](https://github.com/docker-library/hello-world/blob/master/hello.c).

## Listar contenedores con `docker ps`

Como vimos en la [vista rápida](index.md#8-consultar-los-contenedores-en-ejecucion), `docker ps` lista los contenedores **en ejecución** y `docker ps -a` lista **todos**, también los parados. Por ejemplo:

```text
CONTAINER ID   IMAGE         COMMAND       CREATED              STATUS                      PORTS     NAMES
434d318b3771   ubuntu        "bash"        About a minute ago   Up About a minute                     stupefied_colden
b0619341b82c   ubuntu        "/bin/bash"   3 minutes ago        Exited (0) 2 minutes ago              intelligent_ritchie
3fb53852ce99   hello-world   "/hello"      37 minutes ago       Exited (0) 37 minutes ago             recursing_meninsky
77dc9f7f80e5   hello-world   "/hello"      41 minutes ago       Exited (0) 41 minutes ago             priceless_pare
```

Veamos qué significa cada columna:

- **CONTAINER ID**: identificador único del contenedor (versión corta de 12 caracteres).
- **IMAGE**: imagen a partir de la que se creó.
- **COMMAND**: comando que se lanza al arrancar el contenedor.
- **CREATED**: cuándo se creó.
- **STATUS**: si está en marcha o no, y desde hace cuánto.
- **PORTS**: redirección de puertos del contenedor (la veremos más adelante).
- **NAMES**: nombre del contenedor. Se puede indicar al crearlo; si no, Docker genera uno aleatorio.

Documentación: [`docker ps`](https://docs.docker.com/reference/cli/docker/container/ls/).

## Parar y arrancar contenedores con `docker start`, `stop` y `restart`

Ya conocemos `docker start` para arrancar un contenedor **ya creado** (recuerda: `docker run` crea uno nuevo). Junto a él están `docker stop`, para pararlo, y `docker restart`, para reiniciarlo. Los tres se usan igual, seguidos del identificador o del nombre del contenedor:

```bash
docker stop 434d318b3771
docker restart stupefied_colden
```

Documentación: [`docker start`](https://docs.docker.com/reference/cli/docker/container/start/), [`docker stop`](https://docs.docker.com/reference/cli/docker/container/stop/) y [`docker restart`](https://docs.docker.com/reference/cli/docker/container/restart/).

## Inspeccionar contenedores con `docker inspect`

`docker inspect` muestra en detalle la configuración de un contenedor: identificador completo de 64 caracteres, almacenamiento, red, imagen en la que se basa, etc.

```bash
docker inspect IDENTIFICADOR/NOMBRE
```

Documentación: [`docker inspect`](https://docs.docker.com/reference/cli/docker/inspect/).

## Ejecutar comandos en un contenedor con `docker exec`

`docker exec` ejecuta un comando dentro de un contenedor que **esté en ejecución**:

```bash
docker exec [OPCIONES] IDENTIFICADOR/NOMBRE COMANDO [ARGUMENTOS]
```

En el [ejemplo del servidor Apache](index.md#modificar-el-contenido-del-servidor-web) ya lo usamos con `-it` para abrir una shell `bash` dentro del contenedor. Veamos otras dos opciones, suponiendo un contenedor en marcha llamado `contenedor`:

```bash
docker exec -d contenedor touch /tmp/prueba
```

Se ejecuta en segundo plano gracias a `-d`. Simplemente crea con `touch` el fichero `prueba` en `/tmp`.

```bash
docker exec -it -e VAR=1 contenedor bash
```

Abre una shell dentro del contenedor, como en el ejemplo de Apache, pero con `-e` se establece la variable de entorno `VAR` con valor `1`. Lo puedes comprobar dentro de esa shell con `echo $VAR`.

Documentación: [`docker exec`](https://docs.docker.com/reference/cli/docker/container/exec/).

## Copiar ficheros entre anfitrión y contenedor con `docker cp`

`docker cp` copia ficheros y directorios del anfitrión a un contenedor o al revés. **No permite copiar directamente de un contenedor a otro.**

```bash
docker cp idcontainer:/tmp/prueba ./
```

Copia el fichero `/tmp/prueba` del contenedor `idcontainer` (identificador o nombre) al directorio actual del anfitrión.

```bash
docker cp ./miFichero idcontainer:/tmp
```

Copia el fichero `miFichero` del directorio actual al directorio `/tmp` del contenedor.

Documentación: [`docker cp`](https://docs.docker.com/reference/cli/docker/container/cp/).

## Conectarse a un proceso en ejecución con `docker attach`

A veces queremos enlazar nuestra terminal a la entrada y salida estándar del proceso principal de un contenedor que está en segundo plano:

```bash
docker attach [OPCIONES] IDENTIFICADOR/NOMBRE
```

Para probarlo, crearemos un contenedor cuyo proceso escribe la fecha cada segundo de forma indefinida. El comando llama a `sh` con el parámetro `-c`, que indica que la cadena siguiente es un pequeño *script* que debe ejecutar la shell. Este *script* lo podrías probar en cualquier terminal:

```bash
sh -c "while true; do date; sleep 1; done"
```

Lo aplicamos creando un contenedor a partir de la imagen `busybox`:

```bash
docker run -d --name=muchotexto busybox sh -c "while true; do date; sleep 1; done"
```

!!! warning "Atención"
    Los parámetros de este `docker run` (`-d`, `--name`) se explican más abajo.

Con el contenedor en marcha, nos conectamos a su salida y veremos la fecha aparecer cada segundo:

```bash
docker attach muchotexto
```

!!! warning "Cómo salir de `attach`"
    Al estar conectados al proceso principal, `Ctrl+C` le envía la señal de parada y el contenedor se puede detener. La combinación `Ctrl+P` seguida de `Ctrl+Q` permite desconectarse sin pararlo, pero solo funciona en contenedores creados con `-it`.

Documentación: [`docker attach`](https://docs.docker.com/reference/cli/docker/container/attach/).

## Consultar los logs con `docker logs`

`docker logs` muestra lo que ha escrito el contenedor por su salida estándar y de error:

```bash
docker logs [OPCIONES] IDENTIFICADOR/NOMBRE
```

Ya lo usamos para ver el [log del servidor Apache](index.md#consultar-el-log). Es parecido a `docker attach`, pero sin conectarse al proceso y con opciones pensadas para tratar esa información como un log. Con el mismo contenedor `muchotexto` del ejemplo anterior:

```bash
docker logs -f muchotexto
```

Muestra todo lo generado hasta ahora y sigue mostrando lo nuevo (`-f`, *follow*). A diferencia de `attach`, aquí `Ctrl+C` solo deja de mostrar el log: el contenedor sigue en marcha.

```bash
docker logs -n 5 muchotexto
```

Muestra solo las 5 últimas líneas.

Documentación: [`docker logs`](https://docs.docker.com/reference/cli/docker/container/logs/).

## Renombrar contenedores con `docker rename`

`docker rename` cambia el nombre de un contenedor:

```bash
docker rename contenedor1 contenedor2
```

Cambia el nombre de `contenedor1` a `contenedor2`.

Documentación: [`docker rename`](https://docs.docker.com/reference/cli/docker/container/rename/).

## Principales parámetros de `docker run`

Como hemos visto, `docker run` crea un contenedor a partir de una imagen y lo arranca. Su estructura es:

```bash
docker run [PARAMETROS] IMAGEN [COMANDO AL ARRANCAR] [ARGUMENTOS]
```

Veamos algunos ejemplos.

### Ejemplo 1: lanzar Ubuntu y acceder a una terminal

En la [vista rápida](index.md#7-ejecutar-contenedores) ya lanzamos un Ubuntu interactivo con nombre. Ahora indicamos además el comando que debe ejecutarse al arrancar:

```bash
docker run -it --name=nuestroUbuntu1 ubuntu /bin/bash
```

Veamos por separado qué hace cada uno de los parámetros:

- **`-i`**: modo interactivo; mantiene enlazada la entrada estándar del proceso.
- **`-t`**: asigna una pseudoterminal al proceso, para poder usarlo desde nuestra terminal.
- **`--name`**: pone nombre al contenedor. Si no lo indicamos, Docker genera uno aleatorio.

El comando `/bin/bash`, junto con `-it`, hace que justo después de lanzar la orden estemos en una shell dentro del contenedor. Al salir de esa shell (con `exit`, `Ctrl+D`, etc.) **el contenedor se para**.

Los cambios que hagamos desde la shell, como crear un directorio, se guardan en la capa de escritura del contenedor, así que **siguen ahí cuando lo volvemos a arrancar**. Se pierden si eliminamos el contenedor.

### Ejemplo 1 extra: volver a la terminal de un contenedor parado

En el ejemplo anterior, al salir de la shell el contenedor se paraba. ¿Cómo volvemos a él y a su shell? Con `docker start`.

Al arrancar no indicamos ningún comando: se ejecuta el que pusimos en `docker run` o `docker create` (o el de la imagen, si no indicamos ninguno).

```bash
docker start [PARAMETROS] IDENTIFICADOR/NOMBRE
```

Primero obtenemos el identificador o el nombre:

```bash
docker ps -a
```

Y después arrancamos el contenedor:

```bash
docker start -ai IDENTIFICADOR
```

- **`-a`**: enlaza la salida estándar y de error del contenedor a nuestra terminal.
- **`-i`**: modo interactivo; enlaza la entrada estándar del contenedor a nuestra terminal.

### Ejemplo 2: usar una versión concreta de una imagen y autoeliminar el contenedor

```bash
docker run -it --rm ubuntu:22.04 /bin/bash
```

Creamos un contenedor con la versión de `ubuntu` etiquetada como `22.04` en Docker Hub y lo arrancamos como en el ejemplo anterior. El parámetro nuevo es:

- **`--rm`**: en cuanto el contenedor se para, se elimina del sistema.

### Ejemplo 3: servidor web en segundo plano con puertos mapeados

Volvamos al [servidor Apache](index.md#consolidar-los-cambios-de-un-contenedor) de la teoría:

```bash
docker run -d --name my-apache-app -p 8080:80 httpd
```

Ahora que conocemos más parámetros, veamos en detalle los dos que aparecían ahí:

- **`-d`** (*detached*): lanza el contenedor en segundo plano. No vemos su entrada ni su salida; solo se muestra el identificador del contenedor creado.
- **`-p`**: con el formato `pAnf:pCont`, enlaza el puerto `pAnf` del anfitrión con el puerto `pCont` del contenedor. En el ejemplo, el puerto 8080 del anfitrión con el 80 del contenedor.
    - Si solo indicamos un puerto, por ejemplo `-p 80`, se toma como puerto del contenedor y Docker le asocia un puerto libre aleatorio del anfitrión. Lo podemos consultar con `docker ps` o con el comando específico `docker port`.

!!! warning "Atención"
    El mapeo de puertos solo se puede hacer **al crear el contenedor**. No se puede cambiar en un contenedor que ya existe.

!!! tip "Otra forma de cambiar el `index.html`"
    En la teoría cambiamos la página del servidor con `docker exec`. También se puede preparar el `index.html` en el anfitrión y copiarlo con `docker cp`:

    ```bash
    docker cp ./index.html my-apache-app:/usr/local/apache2/htdocs/
    ```

### Ejemplo 4: establecer variables de entorno

Vamos a establecer una variable de entorno y mostrar su valor:

```bash
docker run -it -e MENSAJE=HOLA ubuntu bash
```

Al crear el contenedor hemos definido la variable `MENSAJE` y abierto una shell. Lo comprobamos con:

```bash
echo $MENSAJE
```

El parámetro nuevo es:

- **`-e`**: establece una o varias variables de entorno.

Las imágenes también pueden traer variables de entorno con valores por defecto. Esos valores se mantienen salvo que los sobrescribamos con `-e`.

## Bibliografía

- [Documentación oficial de Docker](https://docs.docker.com/)

---

<small>Adaptado de *Introducción a Docker. UD 03. Principales acciones con Docker*, de Sergi García Barea (CEFIRE, Generalitat Valenciana, 2021), con licencia [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es). Esta adaptación se distribuye con la misma licencia.</small>
