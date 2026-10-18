# Mi aprendizaje

## Antes de la práctica
Antes de esta práctica usaba imágenes que ya existían en Docker Hub, pero nunca había creado una propia. Tampoco sabía que se podía limitar cuánta memoria o CPU usa un contenedor, ni que Docker podía revisar si un contenedor está "sano" o reiniciarlo solo cuando se cae.

## Lo que aprendí
Aprendí a limitar los recursos de un contenedor con `--memory`, `--memory-swap`, `--cpus` y `--cpuset-cpus`. Me ayudó entender que la memoria swap disponible se calcula restando memory-swap menos memory; por ejemplo, con 300m y 1g quedan 724 MB de swap. También aprendí a ver cuántos procesadores tiene mi máquina (en mi caso 8).

Entendí qué es un healthcheck: un comando que Docker ejecuta cada cierto tiempo dentro del contenedor para saber si está funcionando, y que con opciones como el intervalo, el timeout y los reintentos se decide cuándo marcarlo como no saludable.

Lo que más me gustó fue crear mi propia imagen con un Dockerfile. Entendí para qué sirve cada instrucción (FROM, RUN, COPY, EXPOSE, CMD) y vi en la práctica cómo funciona la caché: la primera construcción tardó casi un minuto, pero cuando solo cambié el index.html con mi nombre, Docker reutilizó los pasos anteriores (aparecían como CACHED) y solo volvió a ejecutar el COPY, así que tardó segundos. También entendí que EXPOSE y CMD no crean pasos de construcción, y qué son las imágenes huérfanas.

Con las políticas de reinicio pude ver la diferencia entre cada una probándolas: con `no` el contenedor terminó y se quedó apagado; con `always` y `unless-stopped` se reiniciaba una y otra vez aunque el script terminara bien; y con `on-failure` se reiniciaba solo porque el script terminaba con error (exit 1). Revisar el contador de reinicios con `docker inspect` me ayudó a comprobarlo.

## Problemas que tuve y cómo los resolví
- **CentOS 7 ya no tiene soporte:** al construir la imagen, el paso `yum -y update` falló con el error *"Cannot find a valid baseurl for repo"*. Investigué y era porque los repositorios de CentOS 7 se movieron a `vault.centos.org`. Lo solucioné agregando en el mismo RUN unos comandos `sed` que cambian la dirección de los repositorios antes de actualizar, así no cambió la cantidad de pasos del Dockerfile.
- **Scripts con formato de Windows:** los archivos `inicio.sh` tenían saltos de línea de Windows (CRLF), que Linux no interpreta bien dentro del contenedor. Los convertí a formato Linux (LF) para que se ejecutaran correctamente.
- **Mismo nombre de contenedor:** varios ejemplos usan `server-nginx`, así que tuve que eliminar el contenedor anterior con `docker rm -f server-nginx` antes de crear otro.

## Otros comandos que usé
- `echo $env:NUMBER_OF_PROCESSORS` y `docker info --format "{{.NCPU}}"` para ver los procesadores.
- `docker image inspect mi-apache:1.0` para revisar la información de la imagen.
- `docker ps -a --filter "name=cont-always"` para ver solo un contenedor.
- `docker inspect -f "{{.RestartCount}}" cont-always` para ver cuántas veces se reinició.
- `docker stop cont-always` para detener un contenedor con política de reinicio.
