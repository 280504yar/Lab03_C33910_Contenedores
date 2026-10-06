# Parte 8: Limpieza del ambiente

[← Volver al índice](README.md)

## Objetivo

Revisar qué recursos dejó el laboratorio en mi máquina y eliminar los que ya no se usan, entendiendo qué borra cada comando antes de ejecutarlo.

---

## Inventario antes de limpiar

### Comandos ejecutados

```bash
docker ps -a
docker images
docker volume ls
docker network ls
docker system df
```

### Resultado obtenido

```text
docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED       STATUS                   PORTS     NAMES
da6ba5fc8048   ubuntu        "bash"     4 hours ago   Exited (0) 4 hours ago             amazing_margulis
d108b720a077   hello-world   "/hello"   4 hours ago   Exited (0) 4 hours ago             competent_ardinghelli

docker images                                                         i Info →   U  In Use
IMAGE                   ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest      5e2309035332       25.9kB         9.49kB    U   
laboratorio-flask:1.0   af52160a4333        222MB         54.3MB        
nginx:latest            abe47724e466        242MB         66.3MB        
redis:latest            6f81e8915c60        213MB         57.6MB        
ubuntu:latest           f144425ff09b        162MB         45.6MB    U   

docker volume ls
DRIVER    VOLUME NAME
local     datos-lab

docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
e4485d274a40   bridge    bridge    local
bda8049fbd52   host      host      local
cd09687128c5   none      null      local

docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         2         603.5MB   324.3MB (53%)
Containers      2         0         16.38kB   16.38kB (100%)
Local Volumes   1         0         33B       33B (100%)
Build Cache     11        0         222.7MB   28.67kB
```

### Qué recursos quedaron creados

Como fui deteniendo y eliminando cada contenedor al final de cada parte, lo que quedó fue principalmente:

| Tipo | Recursos | Origen |
|---|---|---|
| Contenedores | Los de `hello-world` y el primer `ubuntu` (parte 2 y 3), en estado `Exited`, porque nunca los borré | Partes 2 y 3 |
| Imágenes | `hello-world`, `ubuntu`, `laboratorio-flask:1.0`, `nginx`, `redis` | Varias partes |
| Volúmenes | `datos-lab` | Parte 6 |
| Redes | Solo las de fábrica (`bridge`, `host`, `none`), porque `red-lab` y `red-app` ya las había borrado | — |

---

## Comandos de limpieza ejecutados

### `docker container prune`

Elimina **todos los contenedores detenidos**. Pide confirmación y al final muestra los IDs borrados y el espacio recuperado. No toca los contenedores que están corriendo.

```text
docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
da6ba5fc804887da146edd8a697d06af144b70677814f42bda480932fc5ffd06
d108b720a07740a1550922e796a5dc678e7b8fd5444a623ddf6426f8f4cc8e70

Total reclaimed space: 16.38kB
```

### `docker image prune`

Por defecto elimina solo las imágenes **colgantes** (*dangling*): las que no tienen nombre ni etiqueta (aparecen como `<none>`), que suelen quedar cuando se reconstruye una imagen con el mismo tag. **No** borra `ubuntu`, `nginx`, etc., aunque no tengan contenedores. Para eso existe `docker image prune -a`, que borra toda imagen sin contenedores asociados.

```text
docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```

### `docker volume prune`

Elimina volúmenes que no están siendo usados por ningún contenedor. Algo importante que descubrí: en las versiones actuales de Docker (desde la 23), este comando **solo borra volúmenes anónimos**; los volúmenes con nombre, como `datos-lab`, se conservan a menos que se use `docker volume prune -a`. Por eso, si `datos-lab` seguía apareciendo, lo borré explícitamente:

```bash
docker volume rm datos-lab
```

```text
docker volume prune
WARNING! This will remove anonymous local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B

docker volume rm datos-lab
datos-lab
```

### `docker system df`

Muestra cuánto espacio en disco usa Docker, separado por tipo: imágenes, contenedores, volúmenes locales y caché de construcción. Para cada tipo indica el total, cuántos están activos, el tamaño y cuánto es **recuperable** (lo que se podría liberar porque no está en uso).

```text
docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         0         603.5MB   486MB (80%)
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     11        0         222.7MB   28.67kB
```

### `docker system prune` (revisado)

Junta varias limpiezas en una: borra contenedores detenidos, redes sin usar, imágenes colgantes y la caché de construcción. Antes de ejecutar, muestra una advertencia con la lista de lo que va a eliminar.

```text
docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - unused build cache

Are you sure you want to continue? [y/N] y
Deleted build cache objects:
g324rdu7a0he6r1p0bhdxzrmx
wk11ryqcykp6i0c2et3l15otp
vg2gicmh89oaz300eibc8ykn3

Total reclaimed space: 28.67k
```

`docker system prune -a` es más agresivo: además borra **todas** las imágenes que no estén en uso por algún contenedor. Eso significa que tendría que volver a descargar `ubuntu`, `python`, `nginx`, `redis`, etc. la próxima vez. No lo ejecuté, siguiendo la indicación del enunciado.

---

## Diferencia entre limpiar contenedores, imágenes y volúmenes

| Limpiar... | Qué se pierde | ¿Se puede recuperar? |
|---|---|---|
| **Contenedores** | Las instancias detenidas y lo escrito en su capa (como `mensaje.txt`) | Se crea otro igual con `docker run`, pero los cambios internos se pierden |
| **Imágenes** | Las plantillas guardadas | Sí: se vuelven a descargar (`pull`) o construir (`build`), solo cuesta tiempo |
| **Volúmenes** | **Los datos persistentes** | **No.** Si no hay respaldo, se pierden para siempre |

Los contenedores y las imágenes son reproducibles a partir de una imagen o un Dockerfile. Los volúmenes contienen datos que no se pueden reproducir.

## Reflexión

Al comparar `docker system df` antes y después se nota cuánto espacio dejan acumulado unas pocas pruebas. La sorpresa fue que `docker volume prune` no borró `datos-lab`. Al principio pensé que algo había fallado, pero leyendo la ayuda del comando entendí que es una protección para no borrar datos con nombre por accidente.

---

## Preguntas de reflexión

**1. ¿Por qué Docker puede consumir mucho espacio en disco?**

Porque nada se borra solo: cada imagen descargada se queda guardada (y algunas pesan cientos de MB), los contenedores detenidos siguen existiendo con sus capas, cada reconstrucción puede dejar imágenes colgantes y caché de construcción, los volúmenes persisten aunque nadie los use y los logs de contenedores que corren mucho tiempo también crecen. Con el tiempo, sin limpiar, eso suma varios GB.

**2. ¿Qué diferencia hay entre eliminar un contenedor y eliminar una imagen?**

Eliminar un contenedor (`docker rm`) borra una instancia específica y su capa de escritura; la imagen queda intacta y puedo crear otro contenedor cuando quiera. Eliminar una imagen (`docker rmi`) borra la plantilla, así que no puedo crear nuevos contenedores de ella hasta volver a descargarla o construirla. Docker no deja borrar una imagen que todavía usa algún contenedor (aunque esté detenido) sin forzarlo, por eso normalmente se borran primero los contenedores.

**3. ¿Por qué se debe tener cuidado al eliminar volúmenes?**

Porque los volúmenes existen precisamente para guardar lo que no se debe perder (por ejemplo, la información de una base de datos), y su eliminación es permanente: no hay papelera. Además, `prune` borra todo lo que no esté en uso **en ese momento**. Un volumen de un proyecto cuyo contenedor simplemente borré podría tener datos importantes y desaparecer con un solo comando.

**4. ¿Qué buenas prácticas aplicaría para mantener limpio su ambiente local?**

- Usar `--rm` en contenedores de prueba (`docker run --rm -it ubuntu bash`), para que se borren solos al salir.
- Ponerles nombre a contenedores y volúmenes, para saber qué es cada cosa y qué se puede borrar.
- Revisar `docker system df` de vez en cuando.
- Ejecutar `docker container prune` y `docker image prune` periódicamente, ya que son seguros.
- Borrar volúmenes uno por uno, sabiendo qué contienen, en lugar de hacerlo de forma masiva.
- Usar etiquetas de versión en mis imágenes y borrar las versiones viejas que ya no uso.
- Agregar un archivo `.dockerignore` en mis proyectos, para no copiar archivos innecesarios a las imágenes.
