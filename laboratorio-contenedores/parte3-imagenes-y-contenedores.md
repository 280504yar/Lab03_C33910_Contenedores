# Parte 3: Imágenes y contenedores

[← Volver al índice](README.md)

## Objetivo

Entender en la práctica la diferencia entre una imagen (la plantilla) y un contenedor (la instancia creada a partir de ella).

---

## Comando ejecutado

```bash
docker pull ubuntu
```

### Explicación

`docker pull` descarga una imagen desde un registro (por defecto Docker Hub) sin crear ningún contenedor. Como no indiqué etiqueta, Docker usó `ubuntu:latest`.

### Resultado obtenido

```text
Using default tag: latest
latest: Pulling from library/ubuntu
06ad70e463aa: Pull complete 
4e07a0f12b2c: Pull complete 
8f70d2bfe91a: Download complete 
Digest: sha256:f144425ff09be612d6d9ad965196e9cdc23dae1f42110a8a11a3e9a8198759f7
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

### Reflexión

La descarga se hizo por capas (cada línea con un hash y `Pull complete`). Ubuntu tiene muy pocas capas porque es una imagen base. Más adelante, al construir mi propia imagen, entendí mejor por qué Docker trabaja así.

---

## Comando ejecutado

```bash
docker images
```

### Explicación

Lista las imágenes guardadas localmente con su repositorio, etiqueta (*tag*), ID, fecha de creación y tamaño.

### Resultado obtenido

```text
 i Info →   U  In Use
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   5e2309035332       25.9kB         9.49kB    U   
ubuntu:latest        f144425ff09b        162MB         45.6MB  
```

### Reflexión

La imagen de Ubuntu pesa alrededor de 80 MB. Una ISO de Ubuntu Desktop pesa varios GB. Esa diferencia ya me dio una pista de que no es un sistema operativo completo.

---

## Comando ejecutado

```bash
docker run -it ubuntu bash
```

### Explicación

- `-i` (*interactive*): mantiene abierta la entrada estándar, para poder escribir comandos.
- `-t` (*tty*): asigna una terminal, para que el prompt y la salida se vean como en una consola normal.
- `bash`: es el comando que se ejecuta dentro del contenedor y reemplaza al comando por defecto de la imagen.

En resumen, **modo interactivo** significa que mi terminal queda conectada al proceso dentro del contenedor, como si hubiera abierto una sesión en otra máquina.

### Comandos dentro del contenedor y resultado

```bash
ls
pwd
cat /etc/os-release
```

```text
root@da6ba5fc8048:/# ls
bin   etc   lib64  opt   run   sys  var
boot  home  media  proc  sbin  tmp
dev   lib   mnt    root  srv   usr
root@da6ba5fc8048:/# pwd
/
root@da6ba5fc8048:/# cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
VERSION_CODENAME=resolute
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=resolute
LOGO=ubuntu-logo

```

### Qué observé dentro del contenedor

- El prompt cambió a `root@<id>:/#`. El texto después de `@` es el ID corto del contenedor, que hace las veces de *hostname*.
- Soy usuario `root` y estoy en `/` (eso mostró `pwd`).
- `ls` muestra la estructura típica de Linux: `bin`, `etc`, `home`, `usr`, `var`, etc.
- `/etc/os-release` dice que es Ubuntu, con su versión y nombre en clave.
- Faltan muchas herramientas que normalmente doy por sentadas (por ejemplo `ping`, `curl` o `nano`). La imagen es mínima.

### Salida del contenedor

```bash
exit
```

Al escribir `exit` terminó el proceso `bash`, que era el proceso principal, y con él se detuvo el contenedor.

---

## Comando ejecutado

```bash
docker ps -a
```

### Resultado obtenido

```text
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                      PORTS     NAMES
da6ba5fc8048   ubuntu        "bash"     About a minute ago   Exited (0) 11 seconds ago             amazing_margulis
d108b720a077   hello-world   "/hello"   7 minutes ago        Exited (0) 7 minutes ago              competent_ardinghelli
```

### Reflexión

El contenedor de Ubuntu quedó como `Exited`, igual que el de *hello-world*. No se borró: si lo reinicio, conserva lo que tenía. Eso lo comprobé en la siguiente sección.

---

## Preguntas de reflexión

**1. ¿La imagen Ubuntu es lo mismo que una máquina virtual Ubuntu?**

No. Una máquina virtual incluye su propio kernel y simula hardware (CPU, disco, red), por lo que arranca un sistema operativo completo. La imagen `ubuntu` solo trae el sistema de archivos y las herramientas básicas de la distribución (el *userland*), sin kernel y sin servicios como `systemd`. Por eso pesa decenas de MB y el contenedor arranca en menos de un segundo.

Esto coincide con el diagrama de la clase *"Contenedores vs máquinas virtuales"*: en las VMs, cada aplicación lleva su propio **Guest OS** encima de un **hipervisor**; en los contenedores, las aplicaciones solo llevan sus binarios y librerías, y todas corren sobre un único **Container Engine** y el sistema operativo del host.

**2. ¿Por qué el contenedor puede parecer un sistema Linux si no es una máquina virtual completa?**

Porque desde adentro ve un sistema de archivos con la estructura de Ubuntu y usa las mismas herramientas (`ls`, `bash`, `apt`). Además, el kernel lo aísla con *namespaces*: el contenedor tiene su propio árbol de procesos (el `bash` es el PID 1), su propio hostname y su propia red. Todo eso da la sensación de estar en una máquina aparte, aunque en realidad es un proceso más del host.

**3. ¿Qué significa que el contenedor comparta el kernel con el host?**

Que no hay un segundo sistema operativo corriendo. Las llamadas al sistema que hacen los procesos del contenedor las atiende directamente el kernel del host, el mismo que usan mis otros programas. Esto hace que los contenedores sean livianos y rápidos. La desventaja es que un contenedor Linux necesita un kernel Linux (en Windows o macOS, Docker Desktop lo resuelve con una VM ligera) y que el aislamiento es menor que el de una VM.

**4. ¿Qué diferencia hay entre una imagen descargada y un contenedor creado?**

La imagen es una plantilla de solo lectura: no cambia y puede usarse para crear muchos contenedores. El contenedor es una instancia de esa imagen con una capa de escritura propia encima, una configuración (nombre, puertos, variables) y un estado (corriendo o detenido). Usando una analogía de programación: la imagen es la clase y el contenedor es el objeto.

---

## Administración de contenedores

### Secuencia de comandos ejecutada

```bash
docker run -it --name mi-ubuntu ubuntu bash
# dentro del contenedor:
echo "Hola desde el contenedor" > mensaje.txt
cat mensaje.txt
exit

docker ps -a
docker start mi-ubuntu
docker exec -it mi-ubuntu bash
# dentro del contenedor:
cat mensaje.txt
exit

docker stop mi-ubuntu
docker rm mi-ubuntu
docker ps -a
```

### Resultados obtenidos

```text

1. cat mensaje.txt
Hola desde el contenedor
2. docker ps -a 
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
8fda3f0b4475   ubuntu        "bash"     39 seconds ago   Exited (0) 18 seconds ago             mi-ubuntu
da6ba5fc8048   ubuntu        "bash"     2 minutes ago    Exited (0) 58 seconds ago             amazing_margulis
d108b720a077   hello-world   "/hello"   7 minutes ago    Exited (0) 7 minutes ago              competent_ardinghelli
3. docker start mi-ubuntu 
mi-ubuntu
4. cat mensaje.txt 
Hola desde el contenedor
5. docker stop / docker rm 
mi-ubuntu
6. docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                          PORTS     NAMES
da6ba5fc8048   ubuntu        "bash"     3 minutes ago   Exited (0) About a minute ago             amazing_margulis
d108b720a077   hello-world   "/hello"   8 minutes ago   Exited (0) 8 minutes ago                  competent_ardinghelli
```

### Uso de `--name`

`--name mi-ubuntu` le asigna un nombre fijo al contenedor. Sin esta opción, Docker inventa uno aleatorio. Con el nombre puedo usar `docker start mi-ubuntu` o `docker stop mi-ubuntu` sin buscar el ID. Los nombres tienen que ser únicos: si intento crear otro contenedor con el mismo nombre, Docker da un error de conflicto.

### Diferencia entre `docker start` y `docker run`

- `docker run` **crea un contenedor nuevo** desde una imagen y lo inicia. Cada `run` produce un contenedor distinto, con un sistema de archivos limpio.
- `docker start` **inicia un contenedor que ya existe** y está detenido. Conserva su configuración y los cambios que se hicieron en su sistema de archivos.

### Uso de `docker exec`

`docker exec` ejecuta un comando **adicional** dentro de un contenedor que ya está corriendo. Con `docker exec -it mi-ubuntu bash` abrí una segunda shell dentro del contenedor. Si salgo de esa shell, el contenedor sigue corriendo, porque su proceso principal es el `bash` original, no el que abrí con `exec`. Por eso después tuve que usar `docker stop`.

### Diferencia entre detener y eliminar un contenedor

- **Detener** (`docker stop`): envía una señal al proceso principal para que termine. El contenedor sigue existiendo, aparece en `docker ps -a` y se puede volver a iniciar con todos sus datos.
- **Eliminar** (`docker rm`): borra el contenedor junto con su capa de escritura. Ya no aparece en ninguna lista y sus datos se pierden. Docker no permite borrar un contenedor que está corriendo, salvo con `-f`.

### Qué pasó con el archivo creado dentro del contenedor

- Después de salir y volver a iniciar el contenedor con `start`, **el archivo seguía ahí**: `cat mensaje.txt` mostró *"Hola desde el contenedor"*. Detener un contenedor no borra su sistema de archivos.
- Después de `docker rm mi-ubuntu`, **el archivo se perdió junto con el contenedor**. Si ahora hago `docker run -it --name mi-ubuntu ubuntu bash` de nuevo, el archivo no existe, porque es un contenedor nuevo creado a partir de la imagen original, que nunca se modificó.

### Preguntas de reflexión

**1. ¿Qué ventaja tiene asignar nombres a los contenedores?**

Hace que los comandos sean legibles y fáciles de repetir (`docker logs app-logs` en lugar de `docker logs 3f2a9c1b7e44`). También permite que otros contenedores de la misma red los encuentren por nombre, como vi en la parte de redes. Y evita duplicados accidentales, porque Docker no deja crear dos contenedores con el mismo nombre.

**2. ¿Qué diferencia hay entre crear un contenedor nuevo y reiniciar uno existente?**

Crear uno nuevo (`run`) parte de la imagen original: estado limpio, sin cambios previos. Reiniciar uno existente (`start`) retoma el mismo contenedor, con los archivos que se crearon o modificaron dentro. Por eso `mensaje.txt` seguía ahí después de `start`.

**3. ¿Qué sucede con los datos creados dentro de un contenedor si este se elimina?**

Se pierden. Los cambios se guardan en una capa de escritura que pertenece solo a ese contenedor, y `docker rm` la borra. La imagen no cambia. Para conservar datos hay que usar volúmenes o bind mounts (parte 6).

**4. ¿Por qué se dice que los contenedores son desechables?**

Porque están pensados para crearse y destruirse sin miedo. Toda la configuración necesaria está en la imagen y los datos importantes deberían vivir fuera del contenedor (en volúmenes). Así, si un contenedor falla o hay que actualizarlo, se borra y se crea otro idéntico en segundos, en lugar de "repararlo" a mano como se haría con un servidor tradicional.
