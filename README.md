# Taller: Volúmenes y Bind Mounts en Docker

Este repositorio documenta la realización del taller de Docker sobre volúmenes y bind mounts, incluyendo capturas de pantalla de los ejercicios completados.

## Ejercicio 1 — Bind Mount en Modo Lectura con Nginx

**Objetivo:** Entender cómo el host controla lo que el contenedor sirve.

### Comandos ejecutados:
```bash
# Crear carpeta y archivo
mkdir -p ~/web
echo "<h1>Hola desde bind mount</h1>" > ~/web/index.html

# Levantar Nginx con bind mount de solo lectura
docker run -d --name web-ro -p 8080:80 \
  -v ~/web:/usr/share/nginx/html:ro \
  nginx:alpine

# Intentar crear archivo dentro del contenedor (debe fallar)
docker exec -it web-ro sh -c 'echo test > /usr/share/nginx/html/test.txt'
```

### Capturas:

![Screenshot 1](PUNTO%201/Screenshot%202025-09-03%20at%2011.01.28%E2%80%AFAM.png)

![Screenshot 2](PUNTO%201/Screenshot%202025-09-03%20at%2011.01.33%E2%80%AFAM.png)

![Screenshot 3](PUNTO%201/Screenshot%202025-09-03%20at%2011.01.41%E2%80%AFAM.png)

![Screenshot 4](PUNTO%201/Screenshot%202025-09-03%20at%2011.02.35%E2%80%AFAM.png)

### Lo aprendido:
- Los bind mounts permiten compartir archivos entre host y contenedor
- El flag `:ro` (read-only) previene modificaciones desde el contenedor
- Los cambios en el host se reflejan inmediatamente en el contenedor

---

## Ejercicio 2 — Named Volume con PostgreSQL

**Objetivo:** Comprobar persistencia de datos.

### Comandos ejecutados:
```bash
# Crear volumen
docker volume create pgdata

# Ejecutar PostgreSQL
docker run -d --name pg -e POSTGRES_PASSWORD=postgres -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16-alpine

# Crear tabla y agregar datos
docker exec -it pg psql -U postgres -c "CREATE TABLE test(id serial, nombre text);"
docker exec -it pg psql -U postgres -c "INSERT INTO test(nombre) VALUES ('Ada'),('Linus');"
docker exec -it pg psql -U postgres -c "SELECT * FROM test;"

# Eliminar contenedor
docker rm -f pg

# Volver a levantar y verificar persistencia
docker run -d --name pg -e POSTGRES_PASSWORD=postgres -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16-alpine
```

### Capturas:

![Screenshot 1](PUNTO%202/Screenshot%202025-09-03%20at%2011.08.16%E2%80%AFAM.png)

![Screenshot 2](PUNTO%202/Screenshot%202025-09-03%20at%2011.15.12%E2%80%AFAM.png)

![Screenshot 3](PUNTO%202/Screenshot%202025-09-03%20at%2011.15.20%E2%80%AFAM.png)

![Screenshot 4](PUNTO%202/Screenshot%202025-09-03%20at%2011.17.36%E2%80%AFAM.png)

![Screenshot 5](PUNTO%202/Screenshot%202025-09-03%20at%2011.21.55%E2%80%AFAM.png)

### Lo aprendido:
- Los named volumes persisten datos independientemente del ciclo de vida del contenedor
- PostgreSQL mantiene su estado completo en el volumen
- Los datos sobreviven a la eliminación y recreación del contenedor

---

## Ejercicio 3 — Volumen Compartido entre Dos Contenedores

**Objetivo:** Producir y consumir datos simultáneamente.

### Comandos ejecutados:
```bash
# Crear volumen compartido
docker volume create sharedlogs

# Productor (escribe timestamps cada segundo)
docker run -d --name writer -v sharedlogs:/data \
  alpine:3.20 sh -c 'while true; do date >> /data/log.txt; sleep 1; done'

# Consumidor (lee en tiempo real)
docker run -it --rm --name reader -v sharedlogs:/data \
  alpine:3.20 tail -f /data/log.txt

# Reiniciar productor y verificar continuidad
docker rm -f writer
docker run -d --name writer -v sharedlogs:/data \
  alpine:3.20 sh -c 'while true; do date >> /data/log.txt; sleep 1; done'

# Verificar contenido
docker run --rm -v sharedlogs:/data alpine:3.20 sh -c 'tail -n 3 /data/log.txt'
```

### Capturas:

![Screenshot 1](PUNTO%203/1.png)

![Screenshot 2](PUNTO%203/2.png)

![Screenshot 3](PUNTO%203/3.png)

![Screenshot 4](PUNTO%203/4.png)

![Screenshot 5](PUNTO%203/5.png)

![Screenshot 6](PUNTO%203/6.png)

![Screenshot 7](PUNTO%203/7.png)

### Lo aprendido:
- Un volumen puede ser compartido simultáneamente entre múltiples contenedores
- Los cambios son visibles en tiempo real entre contenedores
- La persistencia del volumen es independiente de contenedores individuales

---

## Ejercicio 4 — Backup y Restauración de un Volumen

**Objetivo:** Aprender a respaldar y restaurar datos.

### Comandos ejecutados:
```bash
# Crear volumen y añadir archivo
docker volume create appdata
docker run --rm -v appdata:/data alpine:3.20 sh -c 'echo "backup-$(date +%F)" > /data/info.txt'

# Hacer backup a tar en el host
mkdir -p ~/backups
docker run --rm -v appdata:/data:ro -v ~/backups:/backup \
  alpine:3.20 sh -c 'cd /data && tar czf /backup/appdata.tar.gz .'

# Restaurar en nuevo volumen
docker volume create appdata_restored
docker run --rm -v appdata_restored:/data -v ~/backups:/backup \
  alpine:3.20 sh -c 'cd /data && tar xzf /backup/appdata.tar.gz'

# Verificar contenido restaurado
docker run --rm -v appdata_restored:/data alpine:3.20 cat /data/info.txt
```

### Capturas:

![Screenshot 1](PUNTO%204/8.png)

![Screenshot 2](PUNTO%204/9.png)

![Screenshot 3](PUNTO%204/10.png)

### Lo aprendido:
- Los volúmenes se pueden respaldar usando contenedores temporales
- El backup se realiza montando el volumen como read-only y un directorio del host
- La restauración crea un volumen idéntico al original

---

## Reflexión General

Este taller permitió comprender las diferencias fundamentales entre bind mounts y named volumes en Docker:

- **Bind mounts:** Útiles para desarrollo, permiten control directo desde el host
- **Named volumes:** Ideales para datos de producción, gestionados por Docker
- **Persistencia:** Ambos mecanismos mantienen datos independientes del contenedor
- **Compartición:** Los volúmenes facilitan la comunicación entre contenedores
- **Backup:** Docker proporciona mecanismos nativos para respaldo y restauración

### Problemas encontrados y soluciones:
- **Comandos confusos:** La verdad algunos comandos de Docker son bien largos y confusos, con tantos parámetros y flags que a veces uno no sabe ni qué está escribiendo jejeje
- **Práctica necesaria:** Como los comandos pueden ser complicados, toca seguir practicando para que no se olviden
- **Permisos:** Algunos comandos requirieron ajustar permisos de archivos
- **Puertos:** Verificar que los puertos no estuvieran ocupados antes de levantar contenedores
- **Limpieza:** Importancia de limpiar contenedores y volúmenes después de las pruebas

## Autor
Sebastián Cañón Castellanos

## Fecha
3 de Septiembre, 2025
