# Práctica 1. Proxmox

[Descargar la práctica en PDF](assets/UT2_Practica1_Proxmox.pdf){ .md-button .md-button--primary }

## Objetivos y entrega

En esta práctica instalarás Proxmox VE, crearás una máquina virtual y un contenedor LXC, los pondrás en marcha y compararás cómo se comportan.

Al terminar deberías ser capaz de:

- Instalar un hipervisor de tipo 1 y administrarlo desde su consola web.
- Crear y usar una máquina virtual (KVM) y un contenedor (LXC).
- Explicar, con datos medidos por ti, en qué se diferencian y cuándo usar cada uno.

**Qué se entrega:** una memoria en PDF con portada, índice y un apartado por ejercicio. Cada paso importante lleva una captura de pantalla con una frase que explique qué se hace y por qué. No basta con "lo dejamos por defecto": si dejas un valor por defecto, indica qué significa.

**Capturas identificables:** el nombre del servidor Proxmox, de la máquina virtual y del contenedor deben incluir tu apellido (por ejemplo `pve-garcia`, `vm-garcia`, `ct-garcia`), de modo que se vea en las capturas.

## Preparación: la máquina de VirtualBox

Proxmox se instala dentro de una máquina de VirtualBox, y dentro de Proxmox crearás más máquinas: es virtualización anidada. Para que funcione, crea la máquina de VirtualBox con esta configuración:

| Ajuste | Valor |
| --- | --- |
| Tipo | Linux, Debian (64-bit) |
| Memoria | 8 GB recomendado (4 GB como mínimo) |
| Procesadores | 2 o más |
| Virtualización anidada | Sistema → Procesador → marcar "Habilitar VT-x/AMD-V anidado" |
| Disco | 50 GB, reservado dinámicamente |
| Red | NAT, con reenvío del puerto 8006 |

### Virtualización anidada

Sin la opción "VT-x/AMD-V anidado", las máquinas virtuales no arrancan dentro de Proxmox y aparece el error "KVM virtualisation configured, but not available".

!!! warning "Si la casilla aparece en gris"
    Si no se puede marcar, actívala por comando. Apaga del todo la máquina, abre el Símbolo del sistema de Windows y ejecuta, poniendo entre comillas el nombre exacto de tu máquina:

    ```
    cd "C:\Program Files\Oracle\VirtualBox"
    VBoxManage modifyvm "Proxmox" --nested-hw-virt on
    ```

    Al volver a abrir la configuración, la casilla aparecerá marcada.

### Red: NAT

La tarjeta de red de la máquina se deja en modo "NAT" (no confundir con "Red NAT"). Con NAT, Proxmox tiene salida a Internet pero queda detrás de VirtualBox, y hay que añadir un reenvío de puertos para llegar a su consola web: en Red → Avanzadas → Reenvío de puertos, crea una regla con protocolo TCP, puerto anfitrión 8006 y puerto invitado 8006, dejando las direcciones IP vacías.

Con esta configuración:

- La consola web de Proxmox se abre en `https://localhost:8006`.
- Proxmox recibe la dirección `10.0.2.15`, con puerta de enlace `10.0.2.2`.
- La máquina virtual y el contenedor se conectan a `vmbr0` con una IP estática distinta para cada uno, dentro de la red `10.0.2.0/24`. Tienen Internet, pero no son accesibles directamente desde el equipo anfitrión.

!!! danger "Con NAT no se puede usar DHCP dentro de Proxmox"
    VirtualBox entrega siempre la dirección `10.0.2.15`, la misma que ya tiene Proxmox, y la máquina o el contenedor que la reciban se quedan sin red. Usa estas direcciones fijas:

    | Equipo | Dirección IP | Puerta de enlace |
    | --- | --- | --- |
    | Proxmox | `10.0.2.15/24` | `10.0.2.2` |
    | Contenedor | `10.0.2.20/24` | `10.0.2.2` |
    | Máquina virtual | `10.0.2.30/24` | `10.0.2.2` |

    Como servidor DNS, usa el mismo que tiene Proxmox (lo verás en el nodo, en System → DNS) o `8.8.8.8`.

## Ejercicio 1. Instalación de Proxmox VE

Instala Proxmox VE en la máquina de VirtualBox y accede a su consola web desde el equipo anfitrión.

Descarga la ISO de la [página oficial de Proxmox](https://www.proxmox.com/en/downloads). Si estás en el instituto, se te facilitará la ISO para no sobrecargar la red.

1. Arranca la máquina con la ISO y elige la instalación gráfica.
2. Selecciona el disco de destino.
3. Indica país, zona horaria (Europe/Madrid) y teclado.
4. Define la contraseña de `root` y un correo.
5. Configura la red de gestión: nombre del servidor con tu apellido (por ejemplo `pve-garcia.local`) y los datos de red que propone el instalador: dirección `10.0.2.15/24`, puerta de enlace `10.0.2.2` y el DNS que aparezca.
6. Revisa el resumen e instala. Al reiniciar, retira la ISO.
7. Desde el navegador del anfitrión entra en `https://localhost:8006` con el usuario `root`. El aviso del certificado y el de "No valid subscription" son normales: acepta y continúa.

**Evidencias:**

- Captura del resumen de la instalación.
- Captura de la consola web con la sesión iniciada, donde se vea el nombre de tu nodo.

**Responde:**

- ¿Qué versión de Proxmox y qué versión del kernel has instalado? (Lo verás en el resumen del nodo.)
- ¿Por qué Proxmox se administra desde un navegador y no tiene escritorio propio?

## Ejercicio 2. Crear y usar una máquina virtual

Crea en Proxmox una máquina virtual con una distribución de Linux, completa la instalación y comprueba que funciona.

Puedes elegir la distribución que quieras, con o sin entorno gráfico. Si la quieres con escritorio, elige uno ligero (Xubuntu, Lubuntu o Debian con XFCE), porque al estar anidada irá más lenta de lo normal.

**Guías de apoyo (Somebooks):** [Almacenar una imagen ISO en Proxmox VE](http://somebooks.es/almacenar-una-imagen-iso-proxmox-ve/) y [Crear una máquina virtual en Proxmox VE](http://somebooks.es/crear-una-maquina-virtual-proxmox-ve/). Están hechas con una versión anterior de Proxmox, así que alguna pantalla puede no coincidir exactamente con la tuya.

1. **Sube la ISO.** En el almacenamiento `local` del nodo, entra en "Imágenes ISO" y pulsa "Cargar".
2. **Pulsa "Crear VM"** y recorre el asistente:
    - General: nombre con tu apellido (por ejemplo `vm-garcia`).
    - SO: selecciona la ISO que has subido.
    - Sistema: valores por defecto.
    - Discos: 20 GB en `local-lvm`.
    - CPU: 2 núcleos.
    - Memoria: 2048 MB (4096 MB si lleva escritorio y tu equipo lo permite).
    - Red: puente `vmbr0`, modelo VirtIO.
3. **Inicia la máquina** y abre su consola desde Proxmox.
4. **Completa la instalación** del sistema operativo hasta poder iniciar sesión.

**Red de la máquina:** una vez instalado el sistema, configúrale la IP estática de la tabla de la preparación (`10.0.2.30/24`, puerta de enlace `10.0.2.2`) y el servidor DNS. No uses DHCP.

!!! warning "Si la máquina no arranca o Proxmox deja de responder al iniciarla"
    Es probable que Windows tenga Hyper-V activo. Se reconoce porque, en la ventana de VirtualBox de Proxmox, aparece abajo a la derecha el icono de una tortuga verde.

    En ese caso, en la máquina virtual entra en "Options", pon "KVM hardware virtualization" en "No" y, en "Hardware" → "Processors", elige el tipo `qemu64`. La máquina funcionará emulada: puede tardar entre 5 y 15 minutos en arrancar y usará la CPU al 100 %. Con Hyper-V activo, usa una distribución sin entorno gráfico.

    Pulsa "Start" una sola vez y espera; si lo pulsas varias veces, o pulsas "Shutdown" mientras arranca, aparecerá un error de bloqueo.

**Evidencias:**

- Captura de la pestaña "Confirmar" del asistente con el resumen de la configuración.
- Captura de un terminal dentro de la máquina ya instalada con la salida de `hostname`, `ip a` y `uname -r`.
- Captura del "Resumen" de la máquina en Proxmox mientras está en marcha.

**Responde:**

- ¿Qué diferencia hay entre los almacenamientos `local` y `local-lvm`? ¿Qué se guarda en cada uno?
- ¿Qué es `vmbr0` y para qué sirve?
- El modelo de la tarjeta de red es "VirtIO (paravirtualizado)". ¿Qué ventaja tiene frente a emular una tarjeta real?

## Ejercicio 3. Crear y usar un contenedor LXC

Crea un contenedor LXC a partir de una plantilla, ponlo en marcha e instala en él un servidor web.

**Guía de apoyo (Somebooks):** [Crear contenedores Linux a partir de plantillas en Proxmox VE](http://somebooks.es/crear-contenedores-linux-partir-plantillas-proxmox-ve/). Está hecha con una versión anterior de Proxmox, así que alguna pantalla puede no coincidir exactamente con la tuya.

1. **Consigue la plantilla.** En el almacenamiento `local`, entra en "Plantillas de CT", pulsa "Plantillas" y descarga una de Debian o Ubuntu. Si Proxmox no tiene salida a Internet, se te facilitará el archivo para subirlo con "Cargar".
2. **Pulsa "Crear CT"** y recorre el asistente:
    - General: nombre con tu apellido (por ejemplo `ct-garcia`) y contraseña de `root`.
    - Plantilla: la que has descargado.
    - Discos: 8 GB.
    - CPU: 1 núcleo.
    - Memoria: 512 MB.
    - Red: puente `vmbr0`, IPv4 estática `10.0.2.20/24` y puerta de enlace `10.0.2.2`. No uses DHCP. Si dejas la IP vacía, el contenedor no tendrá red.
3. **Inicia el contenedor**, abre su consola e inicia sesión como `root`.
4. **Instala un servidor web:** `apt update && apt install -y nginx`
5. **Compruébalo:** desde la consola del nodo Proxmox ("Shell"), ejecuta `curl -I http://10.0.2.20`.

La respuesta debe ser "200 OK". Con NAT el contenedor no es accesible desde el equipo anfitrión, pero también puedes abrir esa dirección en el navegador de la máquina virtual del ejercicio 2 si tiene escritorio.

!!! tip "Si la consola del navegador no carga"
    Entra al contenedor desde la pantalla de Proxmox en VirtualBox: inicia sesión como `root` y ejecuta `pct enter 101`, cambiando 101 por el número de tu contenedor. Se sale con `exit`.

**Evidencias:**

- Captura de la pestaña "Confirmar" del asistente.
- Captura de la consola del contenedor con la salida de `hostname`, `ip a` y `uname -r`.
- Captura de la comprobación del servidor web en la que se vea la IP del contenedor: la respuesta "200 OK" de curl o la página de bienvenida de nginx en el navegador de la máquina virtual.

**Responde:**

- ¿Qué significa que el contenedor sea "sin privilegios"?
- Al crear la máquina virtual usaste una ISO y tuviste que instalar el sistema. Aquí has usado una plantilla y no has instalado nada. ¿Por qué?

## Ejercicio 4. Máquinas virtuales frente a contenedores

Compara, con datos medidos por ti, la máquina virtual del ejercicio 2 y el contenedor del ejercicio 3, y saca conclusiones.

Copia esta tabla en tu memoria y complétala. La RAM y el disco los verás en el "Resumen" de cada uno en Proxmox, con ambos en marcha y sin hacer nada.

| Medida | Máquina virtual | Contenedor LXC |
| --- | --- | --- |
| Tiempo desde que pulsas "Iniciar" hasta poder iniciar sesión | | |
| Memoria RAM en uso en reposo | | |
| Espacio de disco ocupado | | |
| Versión del kernel (`uname -r`) | | |
| Tiempo que tardó en estar listo para usar desde que lo creaste | | |

Ejecuta también `uname -r` en la consola del propio nodo Proxmox ("Shell") y anótalo.

**Responde a partir de lo que has observado:**

1. ¿Qué es un contenedor LXC?
2. Compara los tres resultados de `uname -r` (nodo, máquina virtual y contenedor). ¿Cuáles coinciden y por qué?
3. ¿Qué sistemas operativos se pueden usar en un contenedor LXC y cuáles no? Relaciónalo con la respuesta anterior.
4. ¿Cuáles son las principales ventajas de los contenedores frente a las máquinas virtuales? Apoya la respuesta en los datos de tu tabla.
5. ¿Cuáles son sus principales desventajas?
6. Pon un ejemplo de servicio para el que usarías una máquina virtual y otro para el que usarías un contenedor, y justifica la elección.

## Ampliación opcional. Snapshots

Un snapshot guarda el estado de una máquina o contenedor en un momento dado para poder volver a él. Compruébalo con tu contenedor:

1. Con nginx funcionando, crea un snapshot desde la pestaña "Snapshots" del contenedor.
2. Rompe algo a propósito: desinstala nginx con `apt remove -y nginx` y comprueba que el servidor web ya no responde.
3. Restaura el snapshot ("Revertir") y comprueba que vuelve a funcionar.

**Evidencias:** captura del snapshot creado, de la comprobación fallando y de la comprobación funcionando de nuevo tras restaurar.

**Responde:** ¿en qué situaciones reales harías un snapshot antes de tocar un servidor?

## Criterios de calificación

| Apartado | Puntos |
| --- | --- |
| Ejercicio 1. Instalación de Proxmox | 2 |
| Ejercicio 2. Máquina virtual instalada y funcionando | 2,5 |
| Ejercicio 3. Contenedor funcionando con el servidor web | 2,5 |
| Ejercicio 4. Tabla comparativa y respuestas razonadas | 2 |
| Presentación: portada, índice, capturas legibles y explicadas, ortografía | 1 |
| Ampliación opcional | +1 |

Un ejercicio sin sus evidencias, o con capturas en las que no aparezca tu apellido en el nombre del equipo, no puntúa.
