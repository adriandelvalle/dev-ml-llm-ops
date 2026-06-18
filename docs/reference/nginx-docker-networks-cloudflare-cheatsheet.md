# Nginx + Docker Networks + Cloudflare Tunnel Cheatsheet

Quick reference for reverse proxy, container networking, and external access.
Last updated: 2026-06-19

---

## Conceptos clave

| Término | Qué es |
| --- | --- |
| Web server | Sirve archivos desde su propio disco |
| Reverse proxy | Reenvía la petición a otro servidor y devuelve su respuesta |
| Forward proxy | Está delante del cliente, oculta al cliente del exterior |
| Reverse proxy (de nuevo) | Está delante del servidor, oculta al servidor del exterior |

Nginx puede ser ambas cosas a la vez, en bloques `location` distintos.

---

## Redes Docker

```bash
# Crear red
docker network create brewery-network

# Conectar contenedor existente
docker network connect brewery-network brewery-api

# Ver redes a las que pertenece un contenedor
docker inspect brewery-api --format '{{json .NetworkSettings.Networks}}'

# Listar redes
docker network ls

# Desconectar
docker network disconnect brewery-network brewery-api
```

> Dentro de una red, los contenedores se resuelven **por nombre**, no por IP.
> `proxy_pass http://brewery-api:8000` funciona porque Docker tiene DNS interno.

---

## Volúmenes — cuándo usar cada uno

| Método | Cuándo |
| --- | --- |
| `COPY` en Dockerfile | Código de aplicación, versionado, parte de un release |
| Volumen (`-v host:contenedor`) | Contenido que cambia frecuentemente (static files, configs) |

```bash
-v ~/projects/brewery-app/static:/usr/share/nginx/html/static:ro
```

`:ro` = read-only. El contenedor solo lee, nunca escribe esos archivos.

---

## Dockerfile de Nginx

```dockerfile
FROM nginx:alpine
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

> `nginx.conf` se copia con `COPY` — cambios requieren rebuild + recreate.
> Los static files sí están en volumen — cambios instantáneos sin rebuild.

---

## nginx.conf — plantilla base

```nginx
server {
    listen 80;
    server_name _;
    charset utf-8;

    # Web server — sirve archivos directamente
    location /static/ {
        alias /usr/share/nginx/html/static/;
    }

    # Reverse proxy — reenvía a otro contenedor
    location / {
        proxy_pass http://brewery-api:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Por qué los proxy_set_header

Sin ellos, la API ve siempre la IP de Nginx como si fuera el cliente real.
Estas cabeceras preservan la IP y protocolo originales del visitante.

---

## Arrancar el stack completo

```bash
# Red
docker network create brewery-network
docker network connect brewery-network brewery-api

# Nginx
cd ~/projects/brewery-app/nginx
docker build -t brewery-nginx:v0.2 .
docker run -d \
  --name brewery-nginx \
  --network brewery-network \
  -p 80:80 \
  -v ~/projects/brewery-app/static:/usr/share/nginx/html/static:ro \
  --restart unless-stopped \
  brewery-nginx:v0.2

# Verificar
docker ps
curl http://localhost/health
curl -s -o /dev/null -w "%{http_code}\n" http://localhost/static/archivo.html
```

---

## Actualizar nginx.conf (requiere rebuild)

```bash
# Tras editar nginx.conf en VS Code:
cd ~/projects/brewery-app/nginx
docker build -t brewery-nginx:v0.X .   # incrementar versión
docker stop brewery-nginx && docker rm brewery-nginx
docker run -d --name brewery-nginx --network brewery-network -p 80:80 \
  -v ~/projects/brewery-app/static:/usr/share/nginx/html/static:ro \
  --restart unless-stopped brewery-nginx:v0.X
```

## Actualizar un static file (sin rebuild)

```bash
# Solo copiar el archivo nuevo — Nginx lo sirve al instante
cp /mnt/Win_Projects/Subidos/archivo.html ~/projects/brewery-app/static/
```

---

## Cloudflare Tunnel

### Quick Tunnel (sin cuenta, sin dominio)

```bash
docker run -d \
  --name brewery-cloudflared \
  --network brewery-network \
  --restart unless-stopped \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate \
  --url http://brewery-nginx:80
```

```bash
docker logs brewery-cloudflared
# Busca la línea: "Your quick Tunnel has been created!"
# URL: https://<random-words>.trycloudflare.com
```

**Limitación**: el subdominio cambia cada vez que el contenedor se reinicia.
No hay forma de fijarlo sin dominio propio registrado en Cloudflare.

### Túnel con nombre (requiere dominio propio en Cloudflare)

```bash
docker run -it \
  -v ~/.cloudflared:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest \
  login
# Requiere tener al menos un dominio añadido como "zona" en tu cuenta
```

> Sin una zona (dominio) en la cuenta, Cloudflare no permite asociar
> un hostname personalizado a un túnel. El error visible es:
> "No domains or subdomains found in any account."

---

## Arquitectura final

```
Internet
        ↓ HTTPS automático
Cloudflare (edge — ej. mad06 Madrid, protocolo QUIC)
        ↓ túnel cifrado SALIENTE (servidor nunca recibe conexiones entrantes)
jotasrv
    └── brewery-cloudflared
            ↓ red Docker "brewery-network"
        brewery-nginx :80
            ├── /static/  → archivos desde disco (volumen)
            └── /, /api/* → brewery-api:8000 (reverse proxy)
                    ↓
                FastAPI + Pydantic
```

---

## Troubleshooting

| Síntoma | Causa | Solución |
| --- | --- | --- |
| Caracteres especiales rotos (`sesiÃ³n`) | Falta charset UTF-8 | `<meta charset="UTF-8">` en HTML + `charset utf-8;` en nginx.conf |
| `localhost` no funciona desde otro dispositivo | `localhost` siempre es la máquina local | Usar la IP real del servidor |
| `mount error(115)` en SMB | IP del host cambió (DHCP) | Verificar con `ping`, actualizar `/etc/fstab` |
| Cambios en nginx.conf no se aplican | Está copiado en la imagen, no en volumen | Rebuild + recreate del contenedor |
| `cloudflared tunnel run` pide ID de túnel | Comando para túnel con nombre, no quick tunnel | Quitar `run`, usar solo `tunnel --url` |

---

## Diagnóstico de red SMB

```bash
# Ver si el mount está activo
df -h | grep Win_Projects

# Verificar conectividad al host Windows
ping -c 4 <IP_DEL_WINDOWS>

# Tras editar /etc/fstab manualmente
sudo systemctl daemon-reload
sudo mount -a
```

## Fijar IP en Windows (cuando el router no soporta DHCP Reservation)

```powershell
netsh interface ip set address "Ethernet" static <IP> 255.255.255.0 <GATEWAY>
netsh interface ip set dns "Ethernet" static 8.8.8.8
```
