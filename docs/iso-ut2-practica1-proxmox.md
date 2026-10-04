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

## Preparación: el escenario en VirtualBox

Trabajarás con dos máquinas de VirtualBox conectadas a la misma red interna: el servidor Proxmox y un cliente Windows desde el que lo administrarás con el navegador. Dentro de Proxmox crearás además una máquina virtual y un contenedor, que estarán en esa misma red. La red interna no tiene salida a Internet, así que todo el material (ISO y plantilla) se te facilitará en clase.

| Equipo | Dirección IP | Dónde se ejecuta |
| --- | --- | --- |
| Servidor Proxmox | `192.168.100.2/24` | Máquina de VirtualBox |
| Cliente Windows | `192.168.100.3/24` | Máquina de VirtualBox |
| Contenedor LXC | `192.168.100.20/24` | Dentro de Proxmox |
| Máquina virtual | `192.168.100.30/24` | Dentro de Proxmox |

### Máquina de VirtualBox para Proxmox

| Ajuste | Valor |
| --- | --- |
| Tipo | Linux, Debian (64-bit) |
| Memoria | 8 GB recomendado (4 GB como mínimo) |
| Procesadores | 2 o más |
| Virtualización anidada | Sistema → Procesador → marcar "Habilitar VT-x/AMD-V anidado" |
| Disco | 50 GB, reservado dinámicamente |
| Red | Red interna, con nombre `intnet`. En Avanzadas, modo promiscuo: "Permitir todo" |

Dos ajustes son imprescindibles:

- **VT-x/AMD-V anidado.** Proxmox se ejecuta dentro de VirtualBox y a su vez ejecuta máquinas: es virtualización anidada. Sin esta opción, las máquinas virtuales no arrancan dentro de Proxmox y aparece el error "KVM virtualisation configured, but not available".
- **Modo promiscuo "Permitir todo".** Sin él, el cliente Windows puede comunicarse con Proxmox, pero no con la máquina virtual ni con el contenedor que crees dentro.

!!! warning "Si la casilla de VT-x/AMD-V anidado aparece en gris"
    Si no se puede marcar, actívala por comando. Apaga del todo la máquina, abre el Símbolo del sistema de Windows y ejecuta, poniendo entre comillas el nombre exacto de tu máquina:

    ```
    cd "C:\Program Files\Oracle\VirtualBox"
    VBoxManage modifyvm "Proxmox" --nested-hw-virt on
    ```

    Al volver a abrir la configuración, la casilla aparecerá marcada.

### Cliente Windows

Usa una máquina virtual de Windows que ya tengas. En su configuración de red, conéctala también a la red interna `intnet` y, dentro de Windows, asígnale la dirección `192.168.100.3` con máscara `255.255.255.0`.

## Ejercicio 1. Instalación de Proxmox VE

Instala Proxmox VE en la máquina de VirtualBox y accede a su consola web desde el cliente Windows.

1. Arranca la máquina con la ISO de Proxmox y elige la instalación gráfica.
2. Selecciona el disco de destino.
3. Indica país, zona horaria (Europe/Madrid) y teclado.
4. Define la contraseña de `root` y un correo.
5. Configura la red de gestión: nombre del servidor con tu apellido (por ejemplo `pve-garcia.local`), dirección `192.168.100.2/24`, puerta de enlace `192.168.100.1` y DNS `192.168.100.1`. La puerta de enlace y el DNS no existen en esta red, pero el instalador obliga a rellenarlos.
6. Revisa el resumen e instala. Al reiniciar, retira la ISO.
7. En el cliente Windows, abre el Símbolo del sistema y comprueba la conexión con `ping 192.168.100.2`.
8. En el navegador del cliente Windows entra en `https://192.168.100.2:8006` con el usuario `root`. El aviso del certificado y el de "No valid subscription" son normales: acepta y continúa.

**Evidencias:**

- Captura del resumen de la instalación.
- Captura del ping desde el cliente Windows a Proxmox.
- Captura de la consola web con la sesión iniciada, donde se vea el nombre de tu nodo.

**Responde:**

- ¿Qué versión de Proxmox y qué versión del kernel has instalado? (Lo verás en el resumen del nodo.)
- ¿Por qué Proxmox se administra desde un navegador y no tiene escritorio propio?

## Ejercicio 2. Crear y usar una máquina virtual

Crea en Proxmox una máquina virtual con una distribución de Linux, completa la instalación y comprueba que funciona.

Puedes elegir la distribución que quieras, con o sin entorno gráfico. Si la quieres con escritorio, elige uno ligero (Xubuntu, Lubuntu o Debian con XFCE), porque al estar anidada irá más lenta de lo normal.

**Guías de apoyo (Somebooks):** [Almacenar una imagen ISO en Proxmox VE](http://somebooks.es/almacenar-una-imagen-iso-proxmox-ve/) y [Crear una máquina virtual en Proxmox VE](http://somebooks.es/crear-una-maquina-virtual-proxmox-ve/). Están hechas con una versión anterior de Proxmox, así que alguna pantalla puede no coincidir exactamente con la tuya.

1. **Pon la ISO de Linux a disposición de Proxmox.** Con la máquina de Proxmox encendida, en el menú de su ventana de VirtualBox elige Dispositivos → Unidades ópticas → Seleccionar imagen de disco, y escoge la ISO de Linux. Proxmox la verá como un CD insertado en su lector.
2. **En la consola web, pulsa "Crear VM"** y recorre el asistente:
    - General: nombre con tu apellido (por ejemplo `vm-garcia`).
    - SO: marca "Usar lector físico de CD/DVD".
    - Sistema: valores por defecto.
    - Discos: 20 GB en `local-lvm`.
    - CPU: 2 núcleos.
    - Memoria: 2048 MB (4096 MB si lleva escritorio y tu equipo lo permite).
    - Red: puente `vmbr0`, modelo VirtIO.
3. **Inicia la máquina** y abre su consola desde Proxmox.
4. **Completa la instalación** del sistema operativo hasta poder iniciar sesión. No hay Internet, así que omite las actualizaciones durante la instalación.
5. **Configura la red** del sistema instalado con la dirección estática `192.168.100.30`, máscara `255.255.255.0`.
6. **Comprueba la conexión** desde el cliente Windows con `ping 192.168.100.30`.

!!! tip "Otra forma de cargar la ISO"
    Si prefieres tenerla en el almacén de Proxmox, con la ISO insertada como en el paso 1 ejecuta en la consola del nodo ("Shell"):

    ```
    dd if=/dev/sr0 of=/var/lib/vz/template/iso/linux.iso bs=4M
    ```

    Aparecerá en `local` → "Imágenes ISO" y podrás elegirla en la pestaña SO del asistente.

!!! warning "Si la máquina no arranca o Proxmox deja de responder al iniciarla"
    Es probable que Windows tenga Hyper-V activo. Se reconoce porque, en la ventana de VirtualBox de Proxmox, aparece abajo a la derecha el icono de una tortuga verde.

    En ese caso, en la máquina virtual entra en "Options", pon "KVM hardware virtualization" en "No" y, en "Hardware" → "Processors", elige el tipo `qemu64`. La máquina funcionará emulada y muy lenta, así que usa una distribución sin entorno gráfico.

    Pulsa "Start" una sola vez y espera; si lo pulsas varias veces, o pulsas "Shutdown" mientras arranca, aparecerá un error de bloqueo.

**Evidencias:**

- Captura de la pestaña "Confirmar" del asistente con el resumen de la configuración.
- Captura de un terminal dentro de la máquina ya instalada con la salida de `hostname`, `ip a` y `uname -r`.
- Captura del ping desde el cliente Windows a la máquina virtual.
- Captura del "Resumen" de la máquina en Proxmox mientras está en marcha.

**Responde:**

- ¿Qué diferencia hay entre los almacenamientos `local` y `local-lvm`? ¿Qué se guarda en cada uno?
- ¿Qué es `vmbr0` y para qué sirve?
- El modelo de la tarjeta de red es "VirtIO (paravirtualizado)". ¿Qué ventaja tiene frente a emular una tarjeta real?

## Ejercicio 3. Crear y usar un contenedor LXC

Crea un contenedor LXC a partir de una plantilla, ponlo en marcha y sirve desde él una página web que se vea en el cliente Windows.

**Guía de apoyo (Somebooks):** [Crear contenedores Linux a partir de plantillas en Proxmox VE](http://somebooks.es/crear-contenedores-linux-partir-plantillas-proxmox-ve/). Está hecha con una versión anterior de Proxmox, así que alguna pantalla puede no coincidir exactamente con la tuya.

1. **Sube la plantilla.** Se te facilitará una plantilla de Ubuntu. Cópiala al cliente Windows (por ejemplo, con una carpeta compartida de VirtualBox) y, en la consola web, entra en el almacenamiento `local` → "Plantillas de CT" y pulsa "Cargar".
2. **Pulsa "Crear CT"** y recorre el asistente:
    - General: nombre con tu apellido (por ejemplo `ct-garcia`) y contraseña de `root`.
    - Plantilla: la que has subido.
    - Discos: 8 GB.
    - CPU: 1 núcleo.
    - Memoria: 512 MB.
    - Red: puente `vmbr0` e IPv4 estática `192.168.100.20/24`. Si dejas la IP vacía, el contenedor no tendrá red.
3. **Inicia el contenedor**, abre su consola e inicia sesión como `root`.
4. **Crea una página web** con tu apellido y sírvela con el servidor web que incluye Python:

    ```
    mkdir /srv/web
    cd /srv/web
    echo "<h1>Contenedor de TU-APELLIDO</h1>" > index.html
    python3 -m http.server 80
    ```

5. **Compruébalo:** en el navegador del cliente Windows entra en `http://192.168.100.20`. Debe aparecer tu página. El servidor se detiene con Ctrl+C.

!!! tip "Si la consola del navegador no carga"
    Entra al contenedor desde la pantalla de Proxmox en VirtualBox: inicia sesión como `root` y ejecuta `pct enter 101`, cambiando 101 por el número de tu contenedor. Se sale con `exit`.

**Evidencias:**

- Captura de la pestaña "Confirmar" del asistente.
- Captura de la consola del contenedor con la salida de `hostname`, `ip a` y `uname -r`.
- Captura del navegador del cliente Windows mostrando tu página, con la IP del contenedor visible en la barra de direcciones.

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

## Criterios de calificación

| Apartado | Puntos |
| --- | --- |
| Ejercicio 1. Instalación de Proxmox | 2 |
| Ejercicio 2. Máquina virtual instalada y funcionando | 2,5 |
| Ejercicio 3. Contenedor funcionando con la página web | 2,5 |
| Ejercicio 4. Tabla comparativa y respuestas razonadas | 2 |
| Presentación: portada, índice, capturas legibles y explicadas, ortografía | 1 |

Un ejercicio sin sus evidencias, o con capturas en las que no aparezca tu apellido en el nombre del equipo, no puntúa.
