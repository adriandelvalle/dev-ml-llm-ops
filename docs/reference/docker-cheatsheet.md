# Docker Cheatsheet

Quick reference for Docker fundamentals and day-to-day operations.
Last updated: 2026-05-27

---

## Instalación (Ubuntu 24.04 — repositorio oficial)

```bash
# 1. Añadir clave GPG oficial de Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 2. Añadir repositorio oficial
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 3. Instalar
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 4. Añadir usuario al grupo docker (evita sudo)
sudo usermod -aG docker $USER
# Cerrar sesión y reconectar para que surta efecto

# 5. Verificar
docker --version
docker run hello-world
```

> Nunca usar `snap install docker` ni `apt install docker.io` en producción.
> Siempre instalar desde el repositorio oficial de Docker.

---

## Conceptos clave

```
Dockerfile  →  docker build  →  Imagen  →  docker run  →  Contenedor
(receta)        (construir)    (plantilla)  (ejecutar)    (instancia viva)
```

**Imagen** — plantilla inmutable. Se descarga de Docker Hub o se construye con un Dockerfile.
**Contenedor** — instancia en ejecución de una imagen.
**Dockerfile** — archivo que define cómo construir una imagen propia.

---

## Dockerfile — brewery-app

```dockerfile
# Imagen base slim — menos peso, menos superficie de ataque
FROM python:3.12-slim

# Directorio de trabajo dentro del contenedor — lo crea Docker automáticamente
WORKDIR /app

# Copiar requirements ANTES del código — aprovecha caché de capas
COPY requirements.txt .

# Instalar dependencias
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Copiar código DESPUÉS de dependencias
COPY src/ ./src/

# Exponer puerto
EXPOSE 8000

# Comando de arranque — 0.0.0.0 obligatorio dentro de Docker
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Por qué el orden requirements → código importa

Docker cachea cada instrucción como una capa. Si el código cambia pero
`requirements.txt` no, Docker reutiliza la capa de `pip install` y el
build es instantáneo. Orden incorrecto = pip install en cada build.

---

## Construir imagen

```bash
# Desde el directorio que contiene el Dockerfile
docker build -t nombre:version .

# Ejemplos
docker build -t brewery-app:v0.1 .
docker build -t brewery-app:v0.2 .

# Ver imágenes construidas
docker images
```

---

## Gestionar contenedores

```bash
# Arrancar contenedor
docker run -d \
  --name brewery-api \
  -p 8000:8000 \
  --restart unless-stopped \
  brewery-app:v0.1

# Ver contenedores corriendo
docker ps

# Ver todos (incluyendo parados)
docker ps -a

# Parar contenedor
docker stop brewery-api

# Arrancar contenedor parado
docker start brewery-api

# Parar y eliminar
docker stop brewery-api && docker rm brewery-api

# Eliminar imagen
docker rmi brewery-app:v0.1
```

---

## Flags de docker run

| Flag | Significado | Ejemplo |
| --- | --- | --- |
| `-d` | Detached — corre en segundo plano | `-d` |
| `--name` | Nombre del contenedor | `--name brewery-api` |
| `-p` | Mapeo de puertos `host:contenedor` | `-p 8000:8000` |
| `--restart` | Política de reinicio | `--restart unless-stopped` |
| `-e` | Variable de entorno | `-e DATABASE_URL=...` |
| `-v` | Montar volumen | `-v /data:/app/data` |
| `--network` | Red Docker | `--network brewery-network` |

---

## Políticas de restart

| Política | Comportamiento | Cuándo usar |
| --- | --- | --- |
| `no` | Nunca reinicia (default) | Contenedores de un solo uso |
| `on-failure` | Solo si falla con error | Jobs, scripts |
| `always` | Siempre, incluso tras `docker stop` | Producción estricta |
| `unless-stopped` | Reinicia tras reboot, respeta stop manual | ✅ Desarrollo y producción flexible |

---

## Mapeo de puertos

```
Red local / Internet
        ↓
  Puerto HOST (jotasrv:8000)
        ↓  -p 8000:8000
  Puerto CONTENEDOR (:8000)
        ↓
  uvicorn 0.0.0.0:8000
```

> `--host 0.0.0.0` en uvicorn es obligatorio dentro de Docker.
> Sin él, uvicorn escucha solo en localhost del contenedor — inaccesible desde fuera.

---

## Logs y monitorización

```bash
# Ver logs
docker logs brewery-api

# Logs en tiempo real
docker logs -f brewery-api

# Últimas N líneas
docker logs --tail 50 brewery-api

# Uso de recursos en tiempo real
docker stats brewery-api

# Información detallada del contenedor
docker inspect brewery-api
```

---

## Debugging — entrar dentro del contenedor

```bash
# Abrir shell interactiva dentro del contenedor
docker exec -it brewery-api bash

# Ejecutar un comando puntual
docker exec brewery-api ls /app
docker exec brewery-api cat /app/src/main.py
```

---

## Limpieza

```bash
# Eliminar contenedores parados
docker container prune

# Eliminar imágenes sin usar
docker image prune

# Eliminar todo lo no usado (contenedores, imágenes, redes, caché)
docker system prune

# Ver uso de disco
docker system df
```

---

## .dockerignore

Archivo en la raíz del contexto de build — evita copiar archivos innecesarios
a la imagen, igual que `.gitignore` en Git.

```
# .dockerignore
venv/
__pycache__/
*.pyc
.pytest_cache/
tests/
*.md
.git/
.gitignore
requirements-dev.txt
```

> Sin `.dockerignore`, Docker copia el `venv/` entero al contexto de build
> — cientos de MB innecesarios que ralentizan el build.

---

## requirements — separación producción / desarrollo

```
backend/
├── requirements.txt          # producción — lo que usa el Dockerfile
└── requirements-dev.txt      # desarrollo — incluye requirements.txt + extras
```

`requirements-dev.txt` empieza con `-r requirements.txt`:
```
-r requirements.txt
pytest==9.0.3
pytest-asyncio==1.3.0
pre_commit==4.5.1
...
```

| Contexto | Archivo |
| --- | --- |
| `docker build` (producción) | `requirements.txt` |
| `pip install` local (venv) | `requirements-dev.txt` |
| GitHub Actions (CI/CD) | `requirements-dev.txt` |

---

## Flujo de trabajo habitual

```bash
# 1. Modificar código
# 2. Reconstruir imagen
docker build -t brewery-app:v0.2 .

# 3. Parar y eliminar contenedor anterior
docker stop brewery-api && docker rm brewery-api

# 4. Arrancar con nueva imagen
docker run -d --name brewery-api -p 8000:8000 --restart unless-stopped brewery-app:v0.2

# 5. Verificar
docker ps
curl http://localhost:8000/health
```

---

## Stack actual del proyecto

```
jotasrv (Ubuntu 24.04)
└── Docker Engine 29.5.2
    └── brewery-api (brewery-app:v0.1)
        └── uvicorn → FastAPI → Pydantic
            └── :8000 → 0.0.0.0:8000
```

**Próximo paso (Semana 4 continuación):**
```
jotasrv
└── Docker Engine
    ├── brewery-nginx    ← Nginx proxy inverso
    │   └── :80/:443 → brewery-api:8000
    └── brewery-api      ← FastAPI (solo escucha en localhost)
```
