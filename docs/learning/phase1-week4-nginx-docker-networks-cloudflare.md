# Fase 1, Semana 4 (Parte 2): Nginx, Redes Docker y Cloudflare Tunnel

## Fecha
2026-06-18 / 2026-06-19

## Objetivo
Montar un reverse proxy con Nginx delante de la API, servir contenido estático
de Tres Tigris, y exponerlo a internet de forma segura con Cloudflare Tunnel.

## Estado al inicio de la sesión
- `brewery-api` corriendo en Docker, solo accesible en red local puerto 8000
- Sin reverse proxy, sin acceso externo, sin servir static files

## Estado al final de la sesión
- Nginx como reverse proxy + web server en el mismo contenedor
- Red Docker propia conectando `brewery-api` y `brewery-nginx`
- Static files de Tres Tigris servidos vía volumen montado
- Cloudflare Tunnel exponiendo la app a internet sin abrir puertos
- Bug de encoding UTF-8 resuelto
- IP fija del Windows configurada tras detectar cambio de DHCP

---

## Conceptos aprendidos

### Nginx no es un application server

Diferencia clave con el background de WebLogic/Tomcat: esos son **application
servers** — ejecutan tu lógica de negocio directamente (Servlets, EJBs).
Nginx **no ejecuta código de la aplicación** — es un servidor web / proxy,
la misma categoría que Apache HTTP Server.

```
Internet
    ↓
Nginx (puerto 80/443) — recibe la petición, decide qué hacer
    ↓ reenvía internamente
uvicorn (puerto 8000) — ejecuta el código FastAPI real
```

### Web server vs Reverse proxy — mismo software, dos funciones

No son categorías excluyentes. El mismo Nginx hace ambas cosas según la ruta:

- **Web server**: sirve contenido que tiene en su propio disco (`/static/`)
- **Reverse proxy**: no tiene el contenido, reenvía la petición a otro
  servidor (`brewery-api:8000`) y devuelve su respuesta como propia

```nginx
location /static/ {
    alias /usr/share/nginx/html/static/;   # web server
}
location / {
    proxy_pass http://brewery-api:8000;    # reverse proxy
}
```

### Forward proxy vs Reverse proxy

```
Forward proxy:   Cliente → Proxy → Internet      (oculta al cliente)
Reverse proxy:   Internet → Proxy → Servidor      (oculta al servidor)
```

Un proxy corporativo que oculta empleados navegando hacia fuera es forward
proxy. Nginx delante de tu API ocultando que existe uvicorn es reverse proxy.

### Por qué Nginx y no Apache para este stack

Apache usa modelo de procesos/hilos por conexión (prefork/worker). Nginx usa
modelo de eventos asíncrono — pocos workers gestionan miles de conexiones sin
overhead de proceso por conexión (resuelve el "C10k problem").

Razón principal para este path: Nginx es el estándar de facto en Kubernetes
(`nginx-ingress-controller`) — la sintaxis aprendida aquí se reutiliza
directamente en Fase 3.

---

## Redes Docker

### El problema que resuelven

Cada contenedor está aislado por defecto. Para que `brewery-nginx` pueda
reenviar peticiones a `brewery-api`, ambos deben compartir una red Docker.

```bash
docker network create brewery-network
docker network connect brewery-network brewery-api
```

### Resolución de nombres automática

Dentro de una red Docker, los contenedores se encuentran **por nombre**, no
por IP. Docker gestiona un DNS interno:

```bash
docker inspect brewery-api --format '{{json .NetworkSettings.Networks}}'
# "DNSNames": ["brewery-api", "<container_id>"]
```

Por eso en `nginx.conf` se usa `proxy_pass http://brewery-api:80` y nunca
una IP — la IP interna del contenedor puede cambiar si se recrea, el nombre no.

---

## Volúmenes — por qué no usar COPY para static files

Dos formas de meter archivos en un contenedor:

**COPY en el Dockerfile** — los archivos quedan dentro de la imagen. Cualquier
cambio requiere rebuild + redeploy. Correcto para código versionado.

**Volumen montado (`-v`)** — el archivo vive en el host, Docker lo "asoma"
dentro del contenedor en tiempo real. Cambios instantáneos sin rebuild.
Correcto para contenido que cambia frecuentemente (cards de Instagram,
configuración).

```bash
-v ~/projects/brewery-app/static:/usr/share/nginx/html/static:ro
```

El flag `:ro` (read-only) es importante — Nginx solo necesita leer, nunca
escribir esos archivos. Limita el daño si el contenedor fuera comprometido.

---

## El Dockerfile de Nginx

```dockerfile
FROM nginx:alpine
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

`alpine` — distribución mínima de Linux usada para imágenes Docker pequeñas
(~30MB total vs cientos de MB de otras bases).

**Importante**: `nginx.conf` se copia con `COPY`, no se monta como volumen.
Cualquier cambio en `nginx.conf` requiere `docker build` + recrear el
contenedor — a diferencia de los static files que sí usan volumen.

---

## nginx.conf final

```nginx
server {
    listen 80;
    server_name _;
    charset utf-8;

    location /static/ {
        alias /usr/share/nginx/html/static/;
    }

    location / {
        proxy_pass http://brewery-api:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Los `proxy_set_header` preservan información del cliente original que se
perdería al pasar por el proxy — sin ellos, la API vería siempre la IP de
Nginx en lugar de la IP real del visitante.

---

## Bug real: encoding UTF-8

### Síntoma
`sesión` se mostraba como `sesiÃ³n` en el navegador.

### Causa
El HTML era un fragmento sin `<head>` ni `<meta charset="UTF-8">`, y Nginx
no declaraba el charset en su configuración — el navegador asumía un
encoding distinto al real (UTF-8 interpretado como Latin-1).

### Solución
Dos partes:
1. Envolver el fragmento en un HTML completo con `<meta charset="UTF-8">`
2. Añadir `charset utf-8;` en el bloque `server` de `nginx.conf`

### Lección
`nginx.conf` vive dentro de la imagen — cualquier cambio requiere rebuild
de la imagen y recrear el contenedor. Los static files en cambio se
actualizan solos por estar en volumen.

---

## Comandos Docker — arrancar el stack completo

```bash
# 1. Red compartida
docker network create brewery-network
docker network connect brewery-network brewery-api

# 2. Nginx
cd ~/projects/brewery-app/nginx
docker build -t brewery-nginx:v0.2 .
docker run -d \
  --name brewery-nginx \
  --network brewery-network \
  -p 80:80 \
  -v ~/projects/brewery-app/static:/usr/share/nginx/html/static:ro \
  --restart unless-stopped \
  brewery-nginx:v0.2
```

---

## Cloudflare Tunnel

### Por qué no port forwarding tradicional

```
Port forwarding:              Cloudflare Tunnel:
Usuario → IP pública           Usuario → Cloudflare
  expuesta → Router            (servidor inicia conexión SALIENTE)
  (puerto abierto) → jotasrv   jotasrv → cloudflared → túnel cifrado
```

Con Cloudflare Tunnel, el servidor **nunca recibe conexiones entrantes** —
`cloudflared` establece una conexión saliente permanente hacia Cloudflare.
Tu IP doméstica nunca se expone, no se abre ningún puerto en el router,
y Cloudflare absorbe ataques antes de que lleguen a ti.

### Quick Tunnel (sin cuenta / sin dominio)

```bash
docker run -d \
  --name brewery-cloudflared \
  --network brewery-network \
  --restart unless-stopped \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate \
  --url http://brewery-nginx:80
```

Genera un subdominio aleatorio en `trycloudflare.com`. **Limitación
importante**: ese subdominio cambia cada vez que el contenedor se reinicia
— no es estable ni elegible.

### Túnel con nombre (requiere dominio propio)

Intentamos crear un túnel con nombre vía `cloudflared login` sin tener un
dominio registrado en Cloudflare. Resultado: error — Cloudflare requiere
una "zona" (dominio) para asociar un hostname personalizado a un túnel.
**No existe forma de tener un subdominio fijo y elegido sin poseer al
menos un dominio.**

### Decisión tomada
Seguir con Quick Tunnel (gratis, subdominio aleatorio y temporal) hasta
tener contenido real que justifique comprar el dominio `trestigris.beer`.
Cuando se compre, se migra a túnel con nombre con subdominio fijo
(`app.trestigris.beer`).

---

## Bug de infraestructura: IP del Windows cambió por DHCP

### Síntoma
`sudo mount -a` del SMB fallaba con error 115 ("Operation now in progress").
`ping 192.168.0.10` devolvía "Destination Host Unreachable".

### Causa
El Windows tenía IP dinámica por DHCP. En algún momento el router le asignó
una IP distinta (`.10` → `.15`). El `/etc/fstab` seguía apuntando a la IP vieja.

### Diagnóstico
```bash
ping -c 4 192.168.0.10        # confirma que la IP vieja no responde
# desde Windows: ipconfig      # confirma la IP real actual
```

### Solución aplicada
1. Actualizar `/etc/fstab` con la IP nueva
2. `sudo systemctl daemon-reload` (necesario tras editar fstab manualmente)
3. `sudo mount -a`
4. Fijar la IP en Windows para que no vuelva a cambiar:
```powershell
netsh interface ip set address "Ethernet" static 192.168.0.15 255.255.255.0 192.168.0.1
netsh interface ip set dns "Ethernet" static 8.8.8.8
```

### Nota sobre el router
El router (Sercom, típico de Movistar/O2) no ofrece DHCP Reservation en su
interfaz — solo permite renombrar dispositivos. Por eso se optó por IP
estática configurada directamente en Windows en lugar de reserva en el router.

### Lección
La IP fija que se había configurado anteriormente era la de **jotasrv**
(`192.168.0.21`), no la del Windows. Cualquier dispositivo cuya IP usen
otros servicios (SMB, scripts, configuraciones) debería tener IP fija,
no solo el servidor principal.

---

## Stack completo verificado

```
Internet (cualquier red, incluso datos móviles)
        ↓ HTTPS automático
Cloudflare (mad06 — Madrid, protocolo QUIC)
        ↓ túnel cifrado saliente
jotasrv
    └── brewery-cloudflared (cloudflared)
            ↓ red Docker "brewery-network"
        brewery-nginx (puerto 80)
            ├── /static/ → static files (volumen, lectura)
            └── /api/*, /health → brewery-api:8000 (reverse proxy)
                    ↓
                brewery-api (uvicorn + FastAPI + Pydantic)
```

Verificado funcionando desde red externa (datos móviles):
```
https://<subdominio-aleatorio>.trycloudflare.com/health
https://<subdominio-aleatorio>.trycloudflare.com/static/tres_tigris_hojas_v2.html
https://<subdominio-aleatorio>.trycloudflare.com/api/v1/recipes/
```

---

## Pendiente

| Item | Estado |
| --- | --- |
| Dominio propio `trestigris.beer` | ⏳ Pendiente — cuando haya contenido real |
| Túnel con nombre (subdominio fijo) | ⏳ Pendiente — depende del dominio |
| Panel admin con login (exportar JPG, gestión socios) | ⏳ Backlog — Fase 3 |
| Modelo `Socio` + registro con RGPD | ⏳ Backlog — Semana 5 (PostgreSQL) |
| DHCP Reservation o IP fija revisada periódicamente | ⏳ Mitigado con IP estática en Windows |
