# Parte 2: Primer contenedor

[← Volver al índice](README.md)

## Objetivo

Ejecutar mi primer contenedor a partir de una imagen de Docker Hub y entender qué pasa en cada paso.

---

## Comando ejecutado

```bash
docker run hello-world
```

### Explicación

`docker run` hace varias cosas en un solo comando:

1. Busca la imagen `hello-world` localmente.
2. Como no la tenía, la descarga de Docker Hub (equivale a un `docker pull` automático).
3. Crea un contenedor nuevo a partir de esa imagen.
4. Lo inicia, ejecuta su programa (que imprime un mensaje) y el contenedor termina.

### Resultado obtenido

```text
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
d5e71e642bf5: Download complete 
Digest: sha256:5e23090353324d887c48ad5e5c56d294eab81588df9605b07d1afe895f9cc8f8
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### ¿Qué ocurrió si la imagen no estaba descargada?

La primera línea de la salida fue *"Unable to find image 'hello-world:latest' locally"*. Después Docker la descargó (`Pulling from library/hello-world`, `Pull complete`) y mostró el *digest* de la imagen. Si vuelvo a ejecutar el mismo comando, ese bloque ya no aparece porque la imagen quedó guardada en mi máquina.

### Reflexión

Me llamó la atención que el mismo mensaje de *hello-world* explica los cuatro pasos que hizo Docker. Es una buena forma de ver el flujo cliente → daemon → registro → contenedor.

---

## Comando ejecutado

```bash
docker ps
```

### Explicación

Lista los contenedores que están **en ejecución** en este momento.

### Resultado obtenido

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### Reflexión

La lista salió vacía aunque acababa de ejecutar un contenedor. Al principio pensé que algo había fallado, pero tiene sentido: el contenedor ya había terminado.

---

## Comando ejecutado

```bash
docker ps -a
```

### Explicación

La opción `-a` (*all*) muestra **todos** los contenedores, incluidos los que están detenidos.

### Resultado obtenido

```text
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
d108b720a077   hello-world   "/hello"   39 seconds ago   Exited (0) 38 seconds ago             competent_ardinghelli
```

### Reflexión

Aquí sí apareció el contenedor, con estado `Exited (0)`. El `0` es el código de salida e indica que el programa terminó sin errores. También vi que Docker le asignó un nombre aleatorio (dos palabras unidas por un guion bajo) porque yo no le puse uno.

---

## Diferencia entre `docker ps` y `docker ps -a`

| `docker ps` | `docker ps -a` |
|---|---|
| Solo contenedores en ejecución (`Up`) | Todos: en ejecución, detenidos (`Exited`), creados (`Created`) |
| Útil para ver qué está corriendo ahora | Útil para encontrar contenedores viejos que siguen ocupando espacio |

## Preguntas de reflexión

**1. ¿Qué es la imagen `hello-world`?**

Es una imagen oficial y muy pequeña (unos pocos KB) que contiene un único programa cuyo trabajo es imprimir un mensaje de bienvenida. Sirve para comprobar que toda la cadena funciona: que el cliente habla con el daemon, que el daemon puede descargar de Docker Hub y que puede crear y ejecutar contenedores.

**2. ¿El contenedor quedó ejecutándose después de imprimir el mensaje?**

No. Un contenedor vive mientras vive su proceso principal. El programa de *hello-world* imprime el texto y termina, así que el contenedor pasa inmediatamente al estado `Exited`.

**3. ¿Por qué aparece en `docker ps -a` pero no necesariamente en `docker ps`?**

Porque `docker ps` solo filtra los que están corriendo. Un contenedor detenido no se borra automáticamente: sigue existiendo con su configuración y su sistema de archivos hasta que lo elimine con `docker rm` (o use `--rm` al ejecutarlo). Por eso solo aparece con `-a`.

**4. ¿Qué demuestra este primer ejemplo sobre Docker?**

Que con un solo comando se puede pasar de no tener nada a ejecutar software empaquetado por otra persona, sin instalar dependencias en mi sistema. También muestra que imagen y contenedor son cosas distintas: la imagen quedó descargada, y el contenedor es una instancia que se creó, corrió y terminó.
