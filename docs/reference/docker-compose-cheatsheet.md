# Docker Compose Cheatsheet

Quick reference for Docker Compose — declarative multi-container stack management.
Last updated: 2026-07-15

---

## Por qué Docker Compose

Sin Compose: 4 comandos `docker run` con flags, redes y volúmenes que hay que
recordar y ejecutar en orden correcto cada vez.

Con Compose: un archivo declarativo + un solo comando.

```bash
docker compose up -d      # levanta todo el stack en background
docker compose down       # para y elimina contenedores (no volúmenes)
docker compose ps         # estado de todos los servicios
docker compose logs       # logs de todos los servicios
docker compose logs -f nombre  # logs en tiempo real de un servicio
```

---

## Estructura del docker-compose.yml

```yaml
services:
  nombre-servicio:
    container_name: nombre-fijo    # evita prefijo y sufijo numérico
    image: imagen:version          # imagen de Docker Hub
    build: ./directorio            # o construir desde Dockerfile local
    environment:
      VARIABLE: ${VARIABLE}        # lee del .env automáticamente
    volumes:
      - nombre-volumen:/ruta/contenedor     # named volume
      - ./ruta/host:/ruta/contenedor:ro     # bind mount read-only
    networks:
      - nombre-red
    ports:
      - "host:contenedor"
    depends_on:
      - otro-servicio
    restart: unless-stopped

volumes:
  nombre-volumen:

networks:
  nombre-red:
    driver: bridge
```

---

## Variables de entorno — .env

Compose lee automáticamente el `.env` del mismo directorio:

```bash
# .env (gitignored)
POSTGRES_USER=brewery
POSTGRES_PASSWORD=mi_password_real
POSTGRES_DB=brewery_db
DATABASE_URL=postgresql+asyncpg://brewery:mi_password_real@brewery-db:5432/brewery_db

# .env.example (commiteado — plantilla pública)
POSTGRES_USER=brewery
POSTGRES_PASSWORD=your_password_here
POSTGRES_DB=brewery_db
DATABASE_URL=postgresql+asyncpg://brewery:your_password_here@brewery-db:5432/brewery_db
```

En `docker-compose.yml` se referencian como `${VARIABLE}` — nunca hardcodeadas.

---

## container_name — nombres fijos

Sin `container_name`, Compose genera `proyecto-servicio-1` (con prefijo y número).
Con `container_name`, el nombre es fijo y predecible:

```yaml
services:
  brewery-api:
    container_name: brewery-api    # siempre "brewery-api", nunca "brewery-app-brewery-api-1"
```

Esencial para aliases, scripts y referencias entre servicios que no cambien.

---

## depends_on — orden de arranque

```yaml
brewery-api:
  depends_on:
    - brewery-db
```

Garantiza que `brewery-db` arranca antes que `brewery-api`.
**No garantiza** que PostgreSQL esté listo para aceptar conexiones —
solo que el contenedor está iniciado. Para eso se añade un healthcheck.

---

## Build vs image

```yaml
# Usar imagen de Docker Hub
brewery-db:
  image: postgres:16

# Construir desde Dockerfile local
brewery-api:
  build: ./backend         # directorio que contiene el Dockerfile
```

---

## Comandos de build

```bash
# Reconstruir todas las imágenes
docker compose build

# Reconstruir un servicio específico
docker compose build brewery-api

# Forzar rebuild sin caché (cuando requirements.txt cambia)
docker compose build --no-cache brewery-api

# Reconstruir y reiniciar
docker compose build brewery-api && docker compose up -d brewery-api
```

---

## Gestión del stack

```bash
# Levantar todo
docker compose up -d

# Levantar solo un servicio (y sus dependencias)
docker compose up -d brewery-api

# Parar todo (sin borrar volúmenes)
docker compose down

# Parar y borrar volúmenes — CUIDADO: borra datos de PostgreSQL
docker compose down -v

# Reiniciar un servicio
docker compose restart brewery-api

# Ver logs
docker compose logs brewery-api
docker compose logs -f brewery-api    # en tiempo real
docker compose logs --tail 50 brewery-api
```

---

## Stack del proyecto brewery-app

```yaml
services:
  brewery-db:
    container_name: brewery-db
    image: postgres:16
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - brewery-db-data:/var/lib/postgresql/data
    networks:
      - brewery-network
    restart: unless-stopped

  brewery-api:
    container_name: brewery-api
    build: ./backend
    environment:
      DATABASE_URL: ${DATABASE_URL}
    networks:
      - brewery-network
    ports:
      - "8000:8000"
    depends_on:
      - brewery-db
    restart: unless-stopped

  brewery-nginx:
    container_name: brewery-nginx
    build: ./nginx
    networks:
      - brewery-network
    ports:
      - "80:80"
    volumes:
      - ./static:/usr/share/nginx/html/static:ro
    depends_on:
      - brewery-api
    restart: unless-stopped

  brewery-cloudflared:
    container_name: brewery-cloudflared
    image: cloudflare/cloudflared:latest
    command: tunnel --no-autoupdate --url http://brewery-nginx:80
    networks:
      - brewery-network
    restart: unless-stopped
    depends_on:
      - brewery-nginx

volumes:
  brewery-db-data:

networks:
  brewery-network:
    driver: bridge
```

---

## Troubleshooting

| Problema | Causa | Solución |
| --- | --- | --- |
| Cambios en código no se aplican | Imagen cacheada | `docker compose build --no-cache` |
| Contenedor con nombre `proyecto-servicio-1` | Falta `container_name` | Añadir `container_name` en el servicio |
| Variables de entorno no se leen | `.env` no existe o está en otro directorio | Verificar que `.env` está en la misma carpeta que `docker-compose.yml` |
| Servicio arranca antes que su dependencia | `depends_on` no garantiza readiness | Añadir healthcheck al servicio dependido |
| Datos de PostgreSQL perdidos | `docker compose down -v` borró el volumen | Nunca usar `-v` salvo que quieras resetear la DB |
