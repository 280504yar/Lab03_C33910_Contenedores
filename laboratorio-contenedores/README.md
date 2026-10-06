# Laboratorio 2: Introducción práctica a contenedores con Docker

**Universidad de Costa Rica** · Escuela de Ingeniería Eléctrica
**Curso:** IE0417 - Diseño de Software para Ingeniería
**Docente:** Rafael Esteban Badilla Alvarado
**Estudiante:** _(Yara Izaguirre)_ · **Carné:** _C33910_
**Fecha:** _6/10/26_

---

## Índice

| Archivo | Contenido | Secciones del enunciado |
|---|---|---|
| [parte1-verificacion.md](parte1-verificacion.md) | Verificación de la instalación de Docker | 7 |
| [parte2-comandos-basicos.md](parte2-comandos-basicos.md) | Primer contenedor: `hello-world`, `docker ps` | 8 |
| [parte3-imagenes-y-contenedores.md](parte3-imagenes-y-contenedores.md) | Imágenes vs. contenedores, modo interactivo y administración básica | 9, 10 |
| [parte4-dockerfile.md](parte4-dockerfile.md) | Aplicación Flask y construcción de la imagen con Dockerfile | 11, 12 |
| [parte5-puertos.md](parte5-puertos.md) | Publicación de puertos, logs e inspección, variables de entorno | 13, 14, 15 |
| [parte6-volumenes.md](parte6-volumenes.md) | Persistencia con volúmenes y bind mounts | 16, 17 |
| [parte7-redes.md](parte7-redes.md) | Redes de Docker y comunicación entre servicios (Nginx, Redis) | 18, 19 |
| [parte8-limpieza.md](parte8-limpieza.md) | Limpieza del ambiente | 20 |
| [app/](app/) | Código de la aplicación: `app.py`, `requirements.txt`, `Dockerfile` | 11, 12 |

## Entorno utilizado

- **Sistema operativo:** _windows 11_
- **Docker:** _29.8.1_**
- **Editor:** Visual Studio Code
- **Terminal:** _PowerShell_

## Cómo ejecutar la aplicación

```bash
cd app
docker build -t laboratorio-flask:1.0 .
docker run -d --name app-lab -p 5000:5000 laboratorio-flask:1.0
# abrir http://localhost:5000 y http://localhost:5000/info
docker stop app-lab && docker rm app-lab
```

Para cambiar el mensaje de la página principal sin reconstruir la imagen:

```bash
docker run -d --name app-lab -p 5000:5000 -e MENSAJE="Otro mensaje" laboratorio-flask:1.0
```

---

## Reflexión final

**1. ¿Qué es un contenedor?**

Es un proceso (o grupo de procesos) que corre en mi computadora, pero aislado del resto del sistema: tiene su propio sistema de archivos, su propia red y su propia lista de procesos, y trae dentro todo lo que la aplicación necesita para funcionar. No tiene un sistema operativo propio, sino que usa el kernel del host. Por eso arranca en segundos y ocupa poco, pero desde adentro parece una máquina aparte.

**2. ¿Qué problema resuelve Docker?**

El de "en mi máquina sí funciona". Como se vio en clase, Docker es una plataforma para desarrollar, distribuir y ejecutar aplicaciones separándolas de la infraestructura. Docker permite empaquetar una aplicación con su versión exacta de lenguaje, librerías y configuración en una imagen, y esa imagen se ejecuta igual en mi computadora, en la de un compañero o en un servidor. También facilita levantar servicios como bases de datos sin instalarlos directamente en el sistema y borrarlos sin dejar rastro.

**3. ¿Qué diferencia hay entre una imagen y un contenedor?**

La imagen es la plantilla: de solo lectura, construida por capas a partir de un Dockerfile o descargada de un registro. El contenedor es una instancia de esa imagen que se está ejecutando o que se detuvo, con una capa propia donde se guardan sus cambios. De una imagen pueden salir muchos contenedores. Lo comprobé con `laboratorio-flask:1.0`, de la que creé `app-lab`, `app-puertos`, `app-logs`, `app-env` y otros, cada uno con una configuración distinta.

**4. ¿Qué diferencia hay entre un contenedor y una máquina virtual?**

Una máquina virtual simula hardware completo y ejecuta su propio sistema operativo con su propio kernel, por eso pesa GB y tarda en arrancar. Un contenedor comparte el kernel del host y solo empaqueta la aplicación y sus dependencias, por eso pesa MB y arranca en menos de un segundo. La VM ofrece un aislamiento más fuerte y puede correr otro sistema operativo; el contenedor es mucho más liviano y rápido. En el laboratorio lo noté al ver que la imagen de Ubuntu pesaba menos de 100 MB y que no traía ni `curl`.

**5. ¿Qué aprendí sobre puertos?**

Que una aplicación que escucha dentro de un contenedor no es accesible desde afuera hasta que publico el puerto con `-p HOST:CONTENEDOR`, y que `EXPOSE` en el Dockerfile solo documenta, no publica. Que la app debe escuchar en `0.0.0.0` y no en `localhost`. Y que el puerto del host lo elijo yo, lo que permite correr la misma app en el 8080 sin tocar el código, aunque dos contenedores no pueden usar el mismo puerto del host al mismo tiempo.

**6. ¿Qué aprendí sobre volúmenes?**

Que los datos escritos dentro de un contenedor se pierden al eliminarlo, como me pasó con `mensaje.txt`, y que los volúmenes resuelven eso guardando la información fuera del contenedor: `archivo.txt` sobrevivió al borrar el primer contenedor. Aprendí la diferencia con los bind mounts, que conectan una carpeta de mi máquina y son ideales para desarrollar. Y aprendí que borrar un volumen es irreversible, por lo que hay que hacerlo con cuidado.

**7. ¿Qué aprendí sobre redes?**

Que los contenedores conectados a una misma red creada por el usuario pueden comunicarse entre sí usando el nombre del contenedor como si fuera un nombre de dominio, gracias al DNS interno de Docker. Y que esa comunicación no necesita publicar puertos hacia el host. Lo vi con `curl http://servidor-web` y con `redis-cli -h redis-lab`, que respondió `PONG`. Esto permite armar aplicaciones de varios servicios sin escribir direcciones IP y sin exponer la base de datos.

**8. ¿En qué casos usaría Docker en un proyecto de software?**

- Para que todo el equipo de un proyecto del curso trabaje con el mismo entorno, sin problemas de versiones.
- Para levantar una base de datos (PostgreSQL, Redis) en desarrollo con un solo comando, sin instalarla en mi computadora.
- Para desplegar una aplicación web en un servidor o en la nube con la misma imagen que probé localmente.
- En integración continua, para ejecutar las pruebas en un ambiente limpio y reproducible.
- Para probar herramientas o versiones distintas sin "ensuciar" mi sistema.


**9. ¿Qué parte del laboratorio me pareció más útil?**

Las variables de entorno. Me pareció muy práctico que, con la misma imagen `laboratorio-flask:1.0`, pudiera cambiar el mensaje de la página solo agregando `-e MENSAJE="..."` al ejecutar el contenedor, sin tocar el código ni reconstruir nada. Así entendí cómo una misma aplicación puede usarse en distintos ambientes, como desarrollo o producción, cambiando únicamente la configuración. También me hizo pensar en por qué no se deben escribir contraseñas directamente en el código.


**10. ¿Qué parte me pareció más confusa?**

La diferencia entre `docker run`, `docker start` y `docker exec`. Al principio pensaba que hacían lo mismo, porque los tres "ejecutan" algo. Con el ejemplo de `mi-ubuntu` entendí que `run` crea un contenedor nuevo desde la imagen, `start` vuelve a iniciar uno que ya existía con sus archivos, y `exec` abre otro proceso dentro de un contenedor que ya está corriendo. También me costó entender por qué, al salir de la terminal abierta con `exec`, el contenedor seguía activo y había que detenerlo con `docker stop`.