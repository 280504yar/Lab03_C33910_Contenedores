# Parte 7: Redes de Docker y comunicación entre servicios

[← Volver al índice](README.md)

Este archivo cubre dos secciones del enunciado: redes de Docker (parte 12) y el ejemplo con aplicación y base de datos simulada (parte 13).

---

## Redes de Docker

### Comandos ejecutados

```bash
docker network create red-lab
docker network ls

docker run -d --name servidor-web --network red-lab nginx
docker run -it --name cliente --network red-lab ubuntu bash
# dentro del contenedor cliente:
apt update
apt install -y curl
curl http://servidor-web
exit

docker stop servidor-web
docker rm servidor-web
docker rm cliente
docker network rm red-lab
```

### Qué es una red en Docker

Es una red virtual, creada por Docker dentro del host, a la que se conectan contenedores. Cada contenedor conectado recibe una IP en esa red y puede comunicarse con los demás contenedores de la misma red, mientras queda aislado de los contenedores de otras redes. Al instalarse, Docker crea tres redes: `bridge` (la red por defecto), `host` y `none`.

### Qué hace `docker network create`

Crea una red nueva. Si no se indica el tipo, es de tipo **bridge**: un switch virtual dentro del host con su propia subred (por ejemplo `172.18.0.0/16`). Docker además le activa un **servidor DNS interno**, algo que la red `bridge` por defecto no tiene.

```text
docker network create red-lab                                                   
4d71f0428f26290bea4b6e91150075a3004d4fe083d9cbab442c32a417496f86

docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
e4485d274a40   bridge    bridge    local
bda8049fbd52   host      host      local
cd09687128c5   none      null      local
4d71f0428f26   red-lab   bridge    local
```

### Qué significa conectar contenedores a la misma red

Con `--network red-lab` le indico a Docker que conecte el contenedor a esa red en vez de a la red por defecto. Los contenedores que comparten red:

- quedan en la misma subred y pueden enviarse tráfico directamente, en **cualquier** puerto, sin necesidad de `-p`;
- se pueden encontrar por su nombre de contenedor;
- quedan aislados de los contenedores que están en otras redes.

### Qué ocurrió al ejecutar `curl http://servidor-web`

Se mostró el HTML de la página de bienvenida de Nginx (*"Welcome to nginx!"*). Es decir, el contenedor `cliente` hizo una petición HTTP al contenedor `servidor-web` y recibió su respuesta.

```text
root@920d3546bf99:/# curl http://servidor-web
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Algo que noté: tuve que instalar `curl` con `apt` porque la imagen `ubuntu` no lo trae. Además, esa instalación solo existe en el contenedor `cliente`; al borrarlo se pierde. Es otra prueba de lo que vi en la parte 3.

### Por qué se pudo usar el nombre `servidor-web`

Porque en las redes creadas por el usuario Docker ofrece **resolución de nombres por DNS**. Cuando `cliente` preguntó por `servidor-web`, el DNS interno de Docker (en la dirección `127.0.0.11` dentro del contenedor) respondió con la IP que tenía el contenedor de Nginx en `red-lab`. No tuve que buscar ni escribir ninguna IP.

En clase se vio que Docker crea una red por defecto de tipo *bridge* y que los contenedores en la misma red pueden comunicarse por nombre. Al hacer el laboratorio, y revisando la documentación de Docker, encontré un detalle que complementa esa idea: la resolución por nombre funciona en las redes **creadas por el usuario** (como `red-lab`), pero **no** en la red `bridge` por defecto, donde los contenedores solo se pueden comunicar por IP. Por eso hizo falta crear `red-lab` con `docker network create`, igual que en el ejemplo de la clase con `red-demo`.


### Reflexión

Fue la parte que más se pareció a una aplicación real: dos programas separados, cada uno en su contenedor, hablando entre sí por nombre. También entendí que Nginx no necesitó `-p` porque el acceso fue desde otro contenedor de la red, no desde mi navegador.

### Preguntas de reflexión

**1. ¿Por qué los contenedores necesitan redes?**

Porque cada contenedor está aislado, también a nivel de red, pero casi ninguna aplicación real es un solo proceso. Normalmente hay un frontend, una API, una base de datos o una caché que tienen que comunicarse. Las redes de Docker permiten esa comunicación de forma controlada: los servicios que deben hablarse comparten red, y los demás quedan separados.

**2. ¿Qué ventaja tiene usar nombres de contenedor en lugar de direcciones IP?**

Las IPs las asigna Docker dinámicamente y pueden cambiar cada vez que un contenedor se recrea. Si la app tuviera la IP escrita en su configuración, dejaría de funcionar en cuanto se reinicie el otro servicio. El nombre es estable y yo lo elijo, así que la configuración se puede escribir de antemano (por ejemplo `DB_HOST=redis-lab`) y además es más legible.

**3. ¿Qué diferencia hay entre publicar un puerto hacia el host y comunicarse dentro de una red Docker?**

Publicar un puerto (`-p`) abre el servicio **hacia afuera**: hacia mi computadora y potencialmente hacia la red local o internet. La comunicación dentro de una red Docker es **interna** entre contenedores: no necesita `-p`, se puede usar cualquier puerto y nada queda expuesto fuera del host. Una buena práctica es publicar solo lo que el usuario necesita (por ejemplo el servidor web) y dejar la base de datos accesible solo por la red interna.

**4. ¿Qué ejemplos reales podrían usar una red Docker?**

- Una aplicación web que se conecta a una base de datos (Flask + PostgreSQL).
- Una API que usa Redis como caché o para manejar sesiones.
- Un proxy reverso (Nginx) que reparte las peticiones entre varios contenedores de backend.
- Microservicios que se llaman entre sí por nombre.
- Un stack de monitoreo (Prometheus recolectando métricas de varios contenedores y Grafana mostrándolas).

---

## Comunicación entre servicios

### Comandos ejecutados

```bash
docker network create red-app
docker run -d --name redis-lab --network red-app redis
docker ps

docker run -it --name cliente-redis --network red-app redis redis-cli -h redis-lab
# dentro del cliente Redis:
ping
set curso IE0417
get curso
exit

docker stop redis-lab
docker rm redis-lab
docker rm cliente-redis
docker network rm red-app
```

### Resultado obtenido

```text
docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS      NAMES
13546b73036d   redis     "docker-entrypoint.s…"   9 seconds ago   Up 9 seconds   6379/tcp   redis-lab
```

```text
docker run -it --name cliente-redis --network red-app redis redis-cli -h redis-lab
redis-lab:6379> ping
PONG
redis-lab:6379> set curso IE0417
OK
redis-lab:6379> get curso
"IE0417"
redis-lab:6379> exit
```

### Qué es Redis en este ejemplo

Redis es una base de datos en memoria de tipo clave-valor, muy usada como caché o para guardar sesiones. Aquí cumple el papel de **"la base de datos" de la aplicación**: es el servicio al que otro programa se conecta para guardar y leer información.

### Qué representa `redis-lab`

Es el nombre del contenedor que ejecuta el **servidor** de Redis y, gracias al DNS de la red, también es su **nombre de host** dentro de `red-app`. Es lo que una aplicación real pondría en su configuración, por ejemplo `REDIS_HOST=redis-lab`.

### Cómo se conectó el cliente al servidor

Creé un segundo contenedor (`cliente-redis`) usando **la misma imagen** `redis`, pero en lugar de iniciar el servidor le pasé como comando `redis-cli -h redis-lab`. Es decir, reemplacé el comando por defecto de la imagen por el cliente de línea de comandos, indicándole el host al que debía conectarse. Como ambos contenedores están en `red-app`, el nombre `redis-lab` se resolvió a la IP del servidor y el cliente se conectó a su puerto 6379, el puerto por defecto de Redis.

### Qué significa recibir `PONG`

`PING` es el comando de Redis para verificar la conexión, y `PONG` es la respuesta del servidor. Recibirla confirma tres cosas: que el nombre `redis-lab` se resolvió por DNS, que hay conectividad de red entre los contenedores y que el servidor Redis está funcionando y aceptando comandos. Después, `set` respondió `OK` y `get curso` devolvió `"IE0417"`, lo que muestra que el dato quedó guardado en el **otro** contenedor.

### Qué enseñanza deja este ejemplo sobre aplicaciones con varios contenedores

Que una aplicación puede dividirse en servicios independientes, cada uno en su propio contenedor con su propia imagen, y comunicarlos solo con una red compartida y nombres. El cliente no necesitó saber nada del servidor más allá de su nombre y su puerto.

Es la misma idea del ejemplo de la clase, donde en la red `red-demo` se levantaban un contenedor `backend` con PostgreSQL y un contenedor `web` con la aplicación Django. En este laboratorio Redis cumple el papel de la base de datos y `redis-cli` el de la aplicación que se conecta a ella. También mostró que la base de datos no necesitaba ningún puerto publicado hacia el host para ser útil, lo cual es más seguro.

### Preguntas de reflexión

**1. ¿Por qué una aplicación web podría necesitar comunicarse con una base de datos?**

Porque la mayoría de las aplicaciones necesitan guardar información que sobreviva entre peticiones y reinicios: usuarios, productos, pedidos, sesiones, etc. El servidor web procesa las peticiones, pero delega el almacenamiento y la consulta de datos a un servicio especializado en eso.

**2. ¿Por qué ambos contenedores deben estar en la misma red?**

Porque las redes de Docker aíslan el tráfico: un contenedor en `red-app` no puede alcanzar a uno que está en otra red. Además, la resolución por nombre (`redis-lab`) solo funciona entre contenedores de la misma red definida por el usuario. Si el cliente hubiera estado en la red por defecto, `redis-cli -h redis-lab` habría fallado porque no podría resolver el nombre.

**3. ¿Qué ventaja tiene separar servicios en contenedores distintos?**

- Cada servicio usa su propia imagen oficial y su versión, sin conflictos de dependencias.
- Se pueden actualizar, reiniciar o reemplazar por separado (por ejemplo, actualizar la app sin tocar la base de datos).
- Se pueden escalar de forma independiente (más copias de la app, una sola base de datos).
- Si un servicio falla, es más fácil identificar cuál y los demás no se caen con él.
- La responsabilidad de cada contenedor es clara: un proceso, una función.

**4. ¿Qué limitación tiene hacerlo manualmente con varios comandos `docker run`?**

Hay que recordar y escribir muchos comandos largos en el orden correcto: crear la red, levantar primero la base de datos, luego la app, con las mismas opciones cada vez. Es fácil equivocarse en un nombre o en una opción, es tedioso de repetir, es difícil de compartir con el equipo y la limpieza también es manual (detener, borrar y quitar la red, uno por uno). Herramientas como **Docker Compose** resuelven esto describiendo todos los servicios, redes y volúmenes en un solo archivo y levantándolos con un solo comando.
