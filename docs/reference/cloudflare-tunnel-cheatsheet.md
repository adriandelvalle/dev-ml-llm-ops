# Cloudflare Tunnel Cheatsheet

Quick reference for exposing local services to the internet securely.
Last updated: 2026-06-19

---

## Qué resuelve

Expone una app que corre en un servidor doméstico a internet, sin abrir
puertos en el router ni exponer la IP pública de casa.

```
Port forwarding tradicional:        Cloudflare Tunnel:
Internet → IP pública expuesta      Internet → Cloudflare
  → Router (puerto abierto)           ↓ túnel cifrado SALIENTE
  → Servidor                        Servidor (cloudflared) → inicia la conexión
```

La diferencia crítica: con Cloudflare Tunnel el servidor **nunca recibe
conexiones entrantes**. `cloudflared` abre una conexión saliente permanente
hacia Cloudflare. Cloudflare mete el tráfico externo por ese túnel ya
establecido — tu IP doméstica nunca aparece en ningún sitio.

---

## Quick Tunnel — sin cuenta, sin dominio

Para pruebas y demos. Gratis, sin registro previo.

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
```

Busca en el log:
```
Your quick Tunnel has been created!
https://<random-words>.trycloudflare.com
```

### Limitación importante

El subdominio es **aleatorio y temporal** — cambia cada vez que el
contenedor se reinicia (manual, reboot del servidor, actualización de imagen).
No hay forma de fijarlo en el modo quick tunnel.

---

## Túnel con nombre — requiere dominio propio

Para tener un subdominio fijo y elegido (`app.tudominio.com`), Cloudflare
exige que el dominio esté añadido como "zona" en tu cuenta.

```bash
# Login — necesario una sola vez
docker run -it \
  -v ~/.cloudflared:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest \
  login
```

Abre la URL que aparece en el log, autoriza desde el navegador.

> **Error común**: "No domains or subdomains found in any account" —
> significa que no tienes ningún dominio registrado como zona en Cloudflare.
> Sin zona, no se puede crear un hostname personalizado. No hay alternativa
> gratuita para esto sin poseer al menos un dominio.

Tras el login, crear el túnel con nombre:

```bash
docker run --rm \
  -v ~/.cloudflared:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest \
  tunnel create brewery-tunnel

docker run --rm \
  -v ~/.cloudflared:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest \
  tunnel route dns brewery-tunnel app.tudominio.com
```

Y correrlo como servicio:

```bash
docker run -d \
  --name brewery-cloudflared \
  --network brewery-network \
  -v ~/.cloudflared:/home/nonroot/.cloudflared \
  --restart unless-stopped \
  cloudflare/cloudflared:latest \
  tunnel run brewery-tunnel
```

---

## Decisión de diseño para este proyecto

Quick Tunnel hasta tener contenido real que justifique comprar un dominio
propio (`trestigris.beer`, ~10€/año). Cuando se compre, se migra a túnel
con nombre con subdominio fijo. La migración es sencilla — solo cambia
el comando de arranque de `cloudflared`, nada en Nginx ni en la API.

---

## Verificación

```bash
docker logs brewery-cloudflared        # ver la URL actual asignada
docker ps                              # confirmar que está corriendo
```

Probar desde fuera de la red local (datos móviles, no WiFi de casa):
```
https://<subdominio>.trycloudflare.com/health
https://<subdominio>.trycloudflare.com/static/archivo.html
```

---

## Por qué el servicio apunta a Nginx y no directamente a la API

```bash
--url http://brewery-nginx:80    # correcto
--url http://brewery-api:8000    # incorrecto para este proyecto
```

Cloudflare Tunnel reenvía tráfico a **una sola URL/puerto**. Apuntando a
Nginx, aprovechas todo lo que Nginx ya resuelve — servir static files y
hacer reverse proxy a la API según la ruta. Apuntar directo a la API
saltaría por completo esa capa.

---

## Troubleshooting

| Síntoma | Causa | Solución |
| --- | --- | --- |
| `"cloudflared tunnel run" requires the ID or name of the tunnel` | Comando de túnel con nombre usado sin haber creado uno | Usar `tunnel --url ...` (sin `run`) para quick tunnel |
| "No domains or subdomains found" | No hay dominio registrado como zona en la cuenta | Comprar/añadir un dominio, o seguir con quick tunnel |
| URL pública no responde | Túnel recién creado, propagación tarda unos segundos | Esperar 10-30s y reintentar |
| URL cambia tras reinicio | Comportamiento esperado del quick tunnel | Revisar `docker logs` tras cada restart, o migrar a túnel con nombre |
