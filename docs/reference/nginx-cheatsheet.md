# Nginx Cheatsheet

Quick reference for Nginx as reverse proxy and web server.
Last updated: 2026-06-19

---

## Conceptos clave

| Término | Qué es |
| --- | --- |
| Web server | Sirve archivos desde su propio disco |
| Reverse proxy | Reenvía la petición a otro servidor y devuelve su respuesta |
| Forward proxy | Está delante del cliente, oculta al cliente del exterior |
| Reverse proxy | Está delante del servidor, oculta al servidor del exterior |

Nginx puede ser ambas cosas — web server y reverse proxy — a la vez, en
bloques `location` distintos dentro del mismo `server {}`.

```
Forward proxy:   Cliente → Proxy → Internet      (oculta al cliente)
Reverse proxy:   Internet → Proxy → Servidor      (oculta al servidor)
```

---

## Por qué Nginx y no Apache (contexto desde WebLogic/Tomcat)

Nginx **no es un application server** — no ejecuta lógica de negocio como
WebLogic o Tomcat. Es la misma categoría que Apache HTTP Server: recibe
peticiones HTTP y decide qué hacer con ellas, sin interpretar código de la app.

```
Internet → Nginx (puerto 80) → reenvía → uvicorn (puerto 8000) → FastAPI
```

Apache: modelo de procesos/hilos por conexión.
Nginx: modelo de eventos asíncrono — pocos workers gestionan miles de
conexiones (resuelve el "C10k problem").

Razón para este path: Nginx es el estándar en Kubernetes
(`nginx-ingress-controller`) — la sintaxis aprendida aquí se reutiliza en Fase 3.

---

## Dockerfile de Nginx

```dockerfile
FROM nginx:alpine
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

> `alpine` — distribución mínima, ~30MB total vs cientos de MB de otras bases.

---

## nginx.conf — plantilla base

```nginx
server {
    listen 80;
    server_name _;
    charset utf-8;

    # Web server — sirve archivos directamente desde disco
    location /static/ {
        alias /usr/share/nginx/html/static/;
    }

    # Reverse proxy — reenvía a otro contenedor/servicio
    location / {
        proxy_pass http://brewery-api:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Por qué `charset utf-8;`

Sin esta línea, Nginx no declara el encoding de la respuesta y el navegador
puede interpretar UTF-8 como otro charset — síntoma típico: `sesión` se ve
como `sesiÃ³n`. También hay que declarar `<meta charset="UTF-8">` en el HTML.

### Por qué los `proxy_set_header`

Sin ellos, la app detrás del proxy ve siempre la IP de Nginx como si fuera
el cliente real. Estas cabeceras preservan IP y protocolo originales:

| Header | Qué preserva |
| --- | --- |
| `Host` | El dominio/host original de la petición |
| `X-Real-IP` | La IP real del visitante |
| `X-Forwarded-For` | Cadena de IPs si hay múltiples proxies |
| `X-Forwarded-Proto` | Si la petición original era http o https |

---

## Servir múltiples rutas con comportamientos distintos

```nginx
location /static/ {
    alias /usr/share/nginx/html/static/;     # web server
}

location /api/ {
    proxy_pass http://brewery-api:8000;      # reverse proxy
}

location / {
    proxy_pass http://brewery-frontend:3000; # reverse proxy a otro servicio
}
```

Nginx evalúa los `location` por especificidad de la ruta — la más específica
gana sobre la genérica `/`.

---

## Build y deploy

```bash
cd ~/projects/brewery-app/nginx
docker build -t brewery-nginx:v0.X .
docker run -d \
  --name brewery-nginx \
  --network brewery-network \
  -p 80:80 \
  -v ~/projects/brewery-app/static:/usr/share/nginx/html/static:ro \
  --restart unless-stopped \
  brewery-nginx:v0.X
```

**Importante**: `nginx.conf` se copia con `COPY` en el build — cualquier
cambio en ese archivo requiere `docker build` + recrear el contenedor.
No se actualiza solo. Ver el cheatsheet de Docker para la diferencia con
volúmenes.

---

## Verificación rápida

```bash
# Pasa por reverse proxy
curl -s http://localhost/health

# Web server — sirve archivo directamente
curl -s -o /dev/null -w "%{http_code}\n" http://localhost/static/archivo.html

# Ver configuración activa dentro del contenedor
docker exec brewery-nginx cat /etc/nginx/conf.d/default.conf

# Test de sintaxis antes de aplicar cambios
docker exec brewery-nginx nginx -t

# Recargar configuración sin reiniciar el contenedor (si se monta como volumen)
docker exec brewery-nginx nginx -s reload
```

---

## Troubleshooting

| Síntoma | Causa | Solución |
| --- | --- | --- |
| `sesiÃ³n` en vez de `sesión` | Falta charset UTF-8 | `charset utf-8;` en nginx.conf + `<meta charset="UTF-8">` en HTML |
| Cambios en nginx.conf no se aplican | Está copiado en la imagen (COPY), no en volumen | Rebuild + recrear contenedor |
| 502 Bad Gateway | El backend (`proxy_pass`) no está accesible | Verificar que ambos contenedores están en la misma red Docker |
| `localhost` no funciona desde otro dispositivo | `localhost` siempre es la máquina local | Usar la IP real del servidor |
