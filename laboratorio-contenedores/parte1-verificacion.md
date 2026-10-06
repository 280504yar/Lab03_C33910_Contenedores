# Parte 1: Verificación de la instalación de Docker

[← Volver al índice](README.md)

## Objetivo

Antes de empezar quise confirmar que Docker estaba instalado y, sobre todo, que el servicio estaba corriendo. Si eso falla, ninguno de los comandos de las partes siguientes funciona.

## Datos del entorno

| Elemento | Valor |
|---|---|
| Sistema operativo | _Windows 11_ |
| Versión de Docker | _29.8.1_ |
| Forma de instalación | _Docker Desktop_ |

---

## Comando ejecutado

```bash
docker --version
```

### Explicación

Muestra la versión del cliente de Docker. Solo confirma que el programa `docker` existe en el sistema y se puede llamar desde la terminal; no dice nada sobre si el daemon está activo.

### Resultado obtenido

```text
Docker version 29.8.1, build 4a63305
```


### Reflexión

Es la verificación más rápida, pero se observó  que no es suficiente: este comando responde aunque el servicio de Docker esté apagado.

---

## Comando ejecutado

```bash
docker info
```

### Explicación

Este comando le pregunta al daemon (el servidor de Docker) por su estado. Por eso la salida se divide en dos bloques: **Client** (el programa que yo uso en la terminal) y **Server** (el servicio que realmente crea y administra los contenedores). Si el daemon no está corriendo, la sección *Server* muestra un error de conexión.

### Resultado obtenido (parcial)

```text
Client:
 Version:    29.8.1
 Context:    desktop-linux
 Debug Mode: false
 Plugins:
  agent: Docker AI Agent Runner (Docker Inc.)
    Version:  v1.141.0
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-agent.exe
  ai: Docker AI Agent - Ask Gordon (Docker Inc.)
    Version:  v1.31.0
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-ai.exe
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.37.1
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-buildx.exe
  compose: Docker Compose (Docker Inc.)
    Version:  v5.5.1
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-compose.exe
  debug: Get a shell into any image or container (Docker Inc.)
    Version:  0.0.47
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-debug.exe
  desktop: Docker Desktop commands (Docker Inc.)
    Version:  v0.4.4
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-desktop.exe
  dhi: CLI for managing Docker Hardened Images (Docker Inc.)
    Version:  v0.0.7
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-dhi.exe
  extension: Manages Docker extensions (Docker Inc.)
    Version:  v0.2.31
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-extension.exe
  init: Creates Docker-related starter files for your project (Docker Inc.)
    Version:  v1.4.0
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-init.exe
  mcp: Docker MCP Plugin (Docker Inc.)
    Version:  v0.43.3
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-mcp.exe
  model: Docker Model Runner (Docker Inc.)
    Version:  v1.2.6
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-model.exe
  offload: Docker Offload (Docker Inc.)
    Version:  v0.6.33
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-offload.exe
  pass: Docker Pass Secrets Manager Plugin (beta) (Docker Inc.)
    Version:  v0.2.2
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-pass.exe
  sandbox: "docker sandbox" is deprecated, use Docker Sandboxes instead (Docker Inc.)
    Version:  v0.13.0
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-sandbox.exe
  scout: Docker Scout (Docker Inc.)
    Version:  v1.24.0
    Path:     C:\Users\YARADEYENISAIZAGUIRR\.docker\cli-plugins\docker-scout.exe

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 29.8.1
```

### Qué información muestra `docker info`


- **Containers / Running / Paused / Stopped:** cuántos contenedores existen y en qué estado están.
- **Images:** cuántas imágenes hay descargadas o construidas.
- **Server Version:** versión del daemon, que puede ser distinta a la del cliente.
- **Storage Driver**: cómo guarda Docker las capas de las imágenes en disco.
- **Operating System / Kernel Version / Architecture:** sobre qué sistema y kernel corren los contenedores. En Windows o macOS aparece un kernel Linux, porque Docker Desktop usa una máquina virtual ligera por debajo.
- **CPUs y Total Memory:** los recursos que Docker tiene disponibles.
- **Docker Root Dir:** la carpeta donde Docker guarda imágenes, contenedores y volúmenes.

### Reflexión

`docker info` sí demuestra que el sistema completo funciona, porque necesita respuesta del daemon. Se usa como primera prueba cuando algo falle.

---

## Comando ejecutado

```bash
docker help
```

### Explicación

Lista los subcomandos disponibles (`run`, `ps`, `build`, `images`, `volume`, `network`, etc.) con una descripción corta de cada uno. También se puede usar `docker <comando> --help` para ver las opciones de un comando específico.

### Resultado obtenido (parcial)

```text

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Common Commands:
  run         Create and run a new container from an image
  exec        Execute a command in a running container
  ps          List containers
  build       Build an image from a Dockerfile
  bake        Build from a file
  pull        Download an image from a registry
  push        Upload an image to a registry
  images      List images
  login       Authenticate to a registry
  logout      Log out from a registry
  search      Search Docker Hub for images
  version     Show the Docker version information
  info        Display system-wide information

Management Commands:
  agent*      Docker AI Agent Runner
  ai*         Docker AI Agent - Ask Gordon
  builder     Manage builds
```

### Reflexión

Sirvió para ver que los comandos se agrupan por tipo de objeto (`docker container ...`, `docker image ...`, `docker volume ...`, `docker network ...`). Por lo tanto, ayuda a ordenar mentalmente lo que viene en el resto del laboratorio.

---

## ¿Por qué es importante verificar la instalación antes de continuar?

Si no lo verifico al inicio y algo falla en la parte 6 (por ejemplo, al construir la imagen), no sabría si el problema está en mi Dockerfile o en la instalación. Revisar primero elimina una variable: si `docker info` responde bien, sé que cualquier error posterior viene de lo que yo hice y no del entorno.

## Preguntas de reflexión

**1. ¿Qué diferencia hay entre instalar Docker y tener Docker ejecutándose correctamente?**

Instalar Docker solo deja los programas en el disco. Para que funcione, el daemon (`dockerd`) tiene que estar corriendo y el usuario tiene que tener permiso para comunicarse con él. Por eso puede pasar que `docker --version` funcione y `docker ps` dé un error como *"Cannot connect to the Docker daemon"*. En Linux también es común el error *"permission denied"* cuando el usuario no está en el grupo `docker`.

**2. ¿Qué información útil muestra el comando `docker info`?**

El estado del daemon, la cantidad de contenedores (activos y detenidos) y de imágenes, la versión del servidor, el storage driver, el sistema operativo y kernel sobre el que corren los contenedores, y los recursos disponibles (CPUs y memoria). Es como un resumen de salud del entorno Docker.

**3. ¿Por qué Docker necesita un servicio o daemon ejecutándose en segundo plano?**

Porque, como se vio en clase, Docker sigue una arquitectura cliente-servidor con varios componentes: el **Docker Client** (el comando `docker`), el **Docker Daemon** (`dockerd`), las **imágenes**, los **contenedores** y el **registro** (Docker Hub). El comando `docker` que escribo solo envía peticiones a una API. Quien realmente descarga imágenes, crea los contenedores, configura redes y volúmenes y mantiene vivos los procesos es el daemon. Además, los contenedores tienen que seguir funcionando aunque yo cierre la terminal, y eso solo es posible si hay un proceso permanente que los administre.
