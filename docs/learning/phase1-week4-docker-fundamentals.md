# Fase 1, Semana 4: Docker Fundamentals & Containerización

## Fecha
2026-05-27

## Objetivo
Aprender los fundamentos de Docker, containerizar la API FastAPI, y resolver
la deuda técnica de persistencia del servicio tras reinicios del servidor.

## Estado al inicio de la sesión
- API FastAPI funcionando en local con arranque manual
- Sin Docker instalado
- Deuda técnica activa: servicio se perdía tras reinicio del servidor

## Estado al final de la sesión
- Docker 29.5.2 instalado desde repositorio oficial
- Imagen `brewery-app:v0.1` construida
- Contenedor `brewery-api` corriendo con `--restart unless-stopped`
- Servicio persiste tras reinicios — deuda técnica cerrada ✅
- requirements.txt separado en producción y desarrollo
- 36 actualizaciones de seguridad del sistema aplicadas

---

## Conceptos aprendidos

### ¿Por qué existe Docker?

El problema clásico antes de Docker: *"en mi máquina funciona"*.
Diferencias de versiones de Python, librerías, y configuración entre
entornos de desarrollo y producción causaban fallos difíciles de reproducir.

La solución anterior eran las máquinas virtuales — emulaban hardware completo,
pesaban gigabytes y tardaban minutos en arrancar.

Docker resuelve el mismo problema de forma mucho más eficiente: comparte
el kernel del sistema operativo del host y solo aísla lo que necesita
la aplicación.

### VM vs Contenedor

```
VM:                          Contenedor Docker:
┌─────────────────────┐      ┌─────────────────────┐
│   Tu aplicación     │      │   Tu aplicación     │
│   Librerías         │      │   Librerías         │
│   Python/Node/etc   │      │   Python/Node/etc   │
│   SO completo       │      └──────────┬──────────┘
│   Hipervisor        │                 │ comparte
│   Hardware          │      ┌──────────▼──────────┐
└─────────────────────┘      │   Kernel del host   │
                             │   Hardware          │
                             └─────────────────────┘
```

VM: gigabytes, minutos en arrancar, SO completo duplicado.
Contenedor: megabytes, segundos en arrancar, solo empaqueta la app.

### Los tres conceptos fundamentales

**Imagen** — la receta. Inmutable, se descarga de un registro (Docker Hub).
Describe exactamente qué hay dentro. No cambia.

**Contenedor** — la instancia en ejecución de una imagen. Puedes crear
N contenedores de la misma imagen — N instancias independientes.

**Dockerfile** — el archivo donde defines tu propia imagen. Describes
paso a paso cómo construirla.

```
Dockerfile  →  docker build  →  Imagen  →  docker run  →  Contenedor
(receta)        (construir)    (plantilla)  (ejecutar)    (instancia viva)
```

### Por qué Docker importa a nivel profesional

**Reproducibilidad** — el contenedor que pruebas en local es byte a byte
idéntico al de producción. No hay sorpresas.

**Aislamiento** — cada contenedor tiene su propio entorno. Tu app no
interfiere con otras apps del mismo servidor.

**Portabilidad** — si funciona en Docker, funciona en cualquier servidor
Linux, en cualquier cloud. Sin cambios.

---

## Instalación de Docker — por qué el repositorio oficial

Ubuntu ofrece tres opciones, solo una es correcta para producción:

| Opción | Problema |
| --- | --- |
| `snap install docker` | Limitaciones de permisos, no estándar |
| `apt install docker.io` | Versión del repo de Ubuntu, siempre desactualizada |
| **Repositorio oficial Docker** | ✅ Siempre actualizado, estándar de la industria |

El proceso de instalación desde repositorio oficial requiere:
1. Añadir la clave GPG de Docker — verifica que los paquetes son auténticos
2. Añadir el repositorio de Docker a las fuentes de apt
3. Instalar `docker-ce`, `docker-ce-cli`, `containerd.io`, plugins

```bash
# Añadir usuario al grupo docker — evita sudo en cada comando
sudo usermod -aG docker $USER
# Requiere cerrar sesión y reconectar para que surta efecto
```

---

## El Dockerfile de brewery-app

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt
COPY src/ ./src/
EXPOSE 8000
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Por qué este orden importa — caché de capas

Docker construye imágenes en capas. Cada instrucción es una capa cacheada.

Si copias el código antes de instalar dependencias:
- Cualquier cambio de código invalida la caché de pip install
- pip install se ejecuta en cada build aunque requirements.txt no cambió

Si instalas dependencias antes de copiar el código:
- pip install se cachea y solo se ejecuta cuando requirements.txt cambia
- Builds hasta 3 minutos más rápidos en proyectos con muchas dependencias

### Por qué `--host 0.0.0.0` en el CMD

Dentro de Docker, `localhost` es el propio contenedor — no el host.
Si uvicorn escucha solo en `127.0.0.1`, nadie puede acceder desde fuera
del contenedor. `0.0.0.0` significa "escucha en todas las interfaces"
— necesario para que el mapeo de puertos funcione.

### Por qué no necesitas venv dentro de Docker

venv existe para aislar proyectos en una máquina donde conviven varios
proyectos Python. Dentro de un contenedor solo hay una app — el contenedor
ya es el entorno aislado. pip install directo es lo correcto.

---

## requirements.txt — separación producción / desarrollo

Mezclar dependencias de producción y desarrollo en un solo archivo es
un error típico con dos consecuencias:

1. La imagen pesa más de lo necesario
2. Mayor superficie de ataque — cada dependencia extra es un vector potencial

### Solución: dos archivos

```
backend/
├── requirements.txt          # solo producción — lo que usa Docker
└── requirements-dev.txt      # desarrollo = -r requirements.txt + extras
```

`requirements-dev.txt` empieza con `-r requirements.txt` — incluye todo
lo de producción y añade las herramientas de desarrollo encima.

| Contexto | Archivo |
| --- | --- |
| Docker (producción) | `requirements.txt` |
| Desarrollo local (venv) | `requirements-dev.txt` |
| CI/CD (GitHub Actions) | `requirements-dev.txt` |

---

## Comandos Docker esenciales

```bash
# Construir imagen
docker build -t nombre:version .

# Listar imágenes
docker images

# Arrancar contenedor
docker run -d --name nombre -p host:contenedor --restart unless-stopped imagen:version

# Ver contenedores corriendo
docker ps

# Ver logs
docker logs nombre-contenedor
docker logs -f nombre-contenedor    # en tiempo real

# Ver uso de recursos
docker stats nombre-contenedor

# Parar y eliminar contenedor
docker stop nombre-contenedor
docker rm nombre-contenedor

# Entrar dentro del contenedor (debugging)
docker exec -it nombre-contenedor bash
```

---

## Mapeo de puertos — `-p host:contenedor`

```
Internet / red local
        ↓
  Puerto 8000 del HOST (jotasrv)
        ↓  -p 8000:8000
  Puerto 8000 del CONTENEDOR
        ↓
  uvicorn escuchando en 0.0.0.0:8000
```

Sin `-p`, el contenedor está completamente aislado — inaccesible desde fuera.

---

## Políticas de restart

| Política | Comportamiento | Cuándo usar |
| --- | --- | --- |
| `no` | Nunca reinicia (default) | Contenedores de un solo uso |
| `on-failure` | Solo si falla con error | Jobs, scripts |
| `always` | Siempre, incluso tras `docker stop` | Producción estricta |
| `unless-stopped` | Reinicia tras reboot, respeta stop manual | ✅ Desarrollo y producción flexible |

**`unless-stopped`** es la elección correcta para este proyecto — el
contenedor arranca automáticamente tras un reboot del servidor, pero
`docker stop brewery-api` lo para y lo deja parado hasta que lo
arranques manualmente.

---

## Deuda técnica cerrada

| Item | Estado |
| --- | --- |
| No service persistence (FastAPI not auto-starting) | ✅ Cerrado — Docker `unless-stopped` |

---

## Verificación post-reboot

Tras reiniciar el servidor, `docker ps` muestra el contenedor corriendo
sin intervención manual. El campo `CREATED` muestra cuándo se creó el
contenedor y `STATUS` cuánto lleva activo.

```
CONTAINER ID   IMAGE              STATUS         PORTS                    NAMES
792f2af93f7d   brewery-app:v0.1   Up 4 minutes   0.0.0.0:8000->8000/tcp   brewery-api
```

---

## Pendiente — Semana 4 continuación

| Item | Estado |
| --- | --- |
| Nginx como proxy inverso | ⏳ Pendiente |
| Cloudflare Tunnel para acceso externo | ⏳ Pendiente |
| Static files (Tres Tigris HTML) | ⏳ Pendiente |
| .dockerignore | ⏳ Pendiente |
