# Políticas de reinicio

## Para esta parte de la práctica
1. Usa el archivo Dockerfile que se encuentra en cada carpeta con el nombre de la política de reinicio para crear una imagen. Analiza las instrucciones de cada Dockerfile 
2. Usa la imagen para ejecutar un contenedor agregando la política respectiva y observar lo que sucede verificando el cumplimiento de la política de reinicio

### no
No reinicia el contenedor bajo ninguna razón. Esta es la política por default
```
docker build -t politica-no ./no
docker run -d --name cont-no politica-no
```
**Observación:** el contenedor ejecutó `/bin/bash`, terminó enseguida y quedó en estado `Exited (0)`. Docker no lo volvió a iniciar porque con la política `no` nunca se reinicia.

### always
Reinicia siempre el contenedor si se detiene. Si se detiene manualmente, sólo se reiniciará cuando se reinicie el demonio Docker o cuando se reinicie manualmente el propio contenedor
```
docker build -t politica-always ./always
docker run -d --name cont-always --restart always politica-always
docker inspect -f "{{.RestartCount}}" cont-always
```
**Observación:** el script `inicio.sh` termina después de 6 segundos con `exit 1`, y Docker vuelve a iniciar el contenedor cada vez. Al revisarlo, el contenedor llevaba más de un minuto creado pero mostraba `Up 1 second`, y el contador de reinicios marcaba 9. Al detenerlo manualmente con `docker stop cont-always`, el contenedor quedó detenido y ya no se reinició.

### unless-stopped

Similar a always, excepto que cuando el contenedor se detiene (manualmente o de otra manera), no se reinicia incluso después de reiniciar el demonio Docker.
```
docker build -t politica-unless ./unless-stopped
docker run -d --name cont-unless --restart unless-stopped politica-unless
docker inspect -f "{{.RestartCount}}" cont-unless
```
**Observación:** en este caso el script `inicio.sh` no tiene `exit 1`, por lo que termina correctamente (código 0). Aun así, Docker reinició el contenedor varias veces: tenía más de un minuto creado, mostraba `Up Less than a second` y el contador de reinicios pasó de 8 a 9. Esto demuestra que la política `unless-stopped` reinicia el contenedor sin importar el código de salida. Al detenerlo manualmente con `docker stop cont-unless` quedó detenido y no volvió a iniciar.

### on-failure
Se reinicia únicamente cuando hay una falla en la ejecución del código que se manifiesta con un código diferente de 0. No se reinicia si el contenedor se detiene manualmente.

```
docker build -t politica-onfailure ./on-failure
docker run -d --name cont-onfailure --restart on-failure politica-onfailure
docker inspect -f "{{.RestartCount}}" cont-onfailure
```
**Observación:** el script `inicio.sh` termina con `exit 1`, es decir, con un código de error. Por eso Docker reinició el contenedor varias veces: tenía 49 segundos creado, mostraba `Up 5 seconds` y el contador de reinicios pasó de 7 a 8. Si el script terminara con código 0 (sin error), el contenedor no se reiniciaría. Al detenerlo manualmente con `docker stop cont-onfailure` tampoco se reinició.