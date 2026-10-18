# Limitar uso de memoria
Definir la cantidad máxima de memoria RAM que el contenedor puede utilizar a un valor en cierta unidad, en donde m para megabytes, g para gigabytes, etc.
Si configuras esta opción, el valor mínimo permitido es 6m (6 megabytes).
_Se puede usar –-memory o -m_
```
--memory=<valor><unidad>
```
La opción --memory-swap se usar para definir la cantidad total de memoria que un contenedor puede utilizar, es decir se incluye tanto la memoria RAM como la memoria swap
```
--memory=<valor><unidad> --memory-swap=<valorUnidad>
```
Por lo tanto, la memoria swap máxima disponible para el contenedor se calcula como:

Memoria swap máxima = *memory-swap − memory

**Considerar:** el parámetro --memory-swap siempre se utiliza en conjunto con --memory para definir un límite total de memoria que incluye tanto la memoria RAM como la memoria swap. Al establecer solo --memory-swap sin --memory, Docker no tiene un punto de referencia para calcular la memoria swap máxima, lo que causará un error.

### Ejemplos
Limitar la memoria RAM que el contenedor puede utilizar a 10 megabytes
```
docker run -d --name server-nginx --memory=10m nginx:alpine
```

Limitar la memoria RAM que el contenedor puede utilizar a 300 megabytes y que el límite total de memoria y memoria swap combinados que el contenedor puede utilizar sea 1 gigabyte
```
docker run -d --name server-nginx --memory=300m --memory-swap=1g nginx:alpine
```
**¿Cuántos megabytes de memoria swap puede utilizar el contenedor creado anteriormente?**
El contenedor puede utilizar 724 MB de memoria swap, ya que el límite total (memory-swap) es 1g = 1024 MB y la memoria RAM (memory) es 300 MB: 1024 − 300 = 724 MB.
