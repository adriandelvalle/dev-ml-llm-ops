# Docker Networks & Volumes Cheatsheet

Quick reference for container networking and persistent/shared storage.
Complements docker-cheatsheet.md (build, run, lifecycle).
Last updated: 2026-06-19

---

## Por qué existen las redes Docker

Cada contenedor está aislado por defecto — no puede hablar con otros
contenedores a menos que estén conectados a la misma red. Sin red compartida,
`brewery-nginx` no podría reenviar peticiones a `brewery-api`.

```bash
# Crear red
docker network create brewery-network

# Conectar un contenedor existente (no hace falta recrearlo)
docker network connect brewery-network brewery-api

# Listar redes
docker network ls

# Ver detalle de una red (qué contenedores tiene conectados)
docker network inspect brewery-network

# Desconectar
docker network disconnect brewery-network brewery-api
```

---

## Resolución de nombres dentro de una red

Docker gestiona un DNS interno — los contenedores se encuentran **por
nombre**, no por IP. La IP interna puede cambiar si se recrea el contenedor;
el nombre no.

```bash
docker inspect brewery-api --format '{{json .NetworkSettings.Networks}}'
```

```json
{
  "brewery-network": {
    "IPAddress": "172.18.0.2",
    "DNSNames": ["brewery-api", "<container_id>"]
  }
}
```

Esto permite que en `nginx.conf` se use:
```nginx
proxy_pass http://brewery-api:8000;
```
en lugar de hardcodear una IP que puede cambiar.

---

## Tipos de red Docker

| Driver | Cuándo se usa |
| --- | --- |
| `bridge` | Default para un solo host — la que usamos en este proyecto |
| `host` | El contenedor comparte la red del host directamente, sin aislamiento |
| `none` | Sin red — aislamiento total |
| `overlay` | Múltiples hosts (Docker Swarm / Kubernetes) — no aplica aún |

---

## Volúmenes — persistencia y compartición de archivos

### El problema que resuelven

Sin volúmenes, todo lo que escribe un contenedor se pierde al eliminarlo —
los contenedores son efímeros por diseño. Los volúmenes permiten que datos
sobrevivan al ciclo de vida del contenedor, o se compartan entre el host
y el contenedor en tiempo real.

### Bind mount — host directory → contenedor

```bash
-v /ruta/en/el/host:/ruta/en/el/contenedor:ro
```

El `:ro` (read-only) es opcional pero recomendado cuando el contenedor
solo necesita leer, nunca escribir.

```bash
docker run -d \
  -v ~/projects/brewery-app/static:/usr/share/nginx/html/static:ro \
  brewery-nginx:v0.2
```

Cambios en el host se reflejan instantáneamente dentro del contenedor —
sin rebuild, sin redeploy.

### COPY (en Dockerfile) vs volumen — cuándo usar cada uno

| Método | Cuándo |
| --- | --- |
| `COPY` en Dockerfile | Código de aplicación versionado, parte del release |
| Volumen (`-v`) | Contenido que cambia frecuentemente sin republicar la imagen |

Ejemplo del proyecto: el código Python (`src/`) se copia con `COPY` porque
es parte del release versionado. Los static files de Tres Tigris usan
volumen porque cambian cada semana con nuevas cards de Instagram, y nadie
quiere reconstruir la imagen de Nginx por eso.

### Named volume — gestionado por Docker

```bash
docker volume create brewery-db-data
docker run -d -v brewery-db-data:/var/lib/postgresql/data postgres:16
```

A diferencia del bind mount, Docker gestiona dónde vive físicamente el
volumen — útil para datos de bases de datos donde no necesitas acceder
directamente desde el host. Se usará en Semana 5 con PostgreSQL.

```bash
docker volume ls
docker volume inspect brewery-db-data
docker volume rm brewery-db-data
```

---

## Arquitectura de red del proyecto

```
brewery-network (bridge)
    ├── brewery-api        172.18.0.2   (FastAPI, puerto interno 8000)
    ├── brewery-nginx      172.18.0.3   (puerto host 80 → 80)
    └── brewery-cloudflared 172.18.0.4  (sin puerto expuesto — solo saliente)
```

Solo `brewery-nginx` tiene `-p` hacia el host — los demás contenedores
solo son alcanzables dentro de la red interna por nombre.

---

## Comandos de diagnóstico

```bash
# Ver todas las redes y qué contenedores tienen
docker network ls
docker network inspect brewery-network

# Verificar que un contenedor resuelve a otro por nombre
docker exec brewery-nginx ping -c 2 brewery-api

# Ver volúmenes montados de un contenedor
docker inspect brewery-nginx --format '{{json .Mounts}}'
```

---

## Troubleshooting

| Síntoma | Causa | Solución |
| --- | --- | --- |
| 502 Bad Gateway en Nginx | Contenedores no comparten red | `docker network connect brewery-network <contenedor>` |
| Cambios en archivo montado no se ven | Ruta del volumen incorrecta o typo | Verificar con `docker inspect --format '{{json .Mounts}}'` |
| `nginx: [emerg] host not found` | El backend no está en la misma red o el nombre está mal escrito | Verificar `docker network inspect brewery-network` |
