# Alembic Cheatsheet

Quick reference for Alembic database migrations with SQLAlchemy.
Last updated: 2026-07-15

---

## Qué es Alembic

Gestor de migraciones para SQLAlchemy. Una migración es un archivo Python que
describe un cambio en el esquema de la base de datos de forma incremental y reversible.

```
Sin Alembic: cambiar un modelo → tirar la DB y recrearla
Con Alembic: cambios incrementales, reversibles y versionados en Git
```

---

## Instalación

```
# requirements.txt
alembic==1.16.2
```

---

## Inicialización (una sola vez)

```bash
# Dentro del contenedor de la API
docker exec -it brewery-api bash
alembic init alembic

# Copiar archivos generados al host
docker cp brewery-api:/app/alembic ~/projects/brewery-app/backend/
docker cp brewery-api:/app/alembic.ini ~/projects/brewery-app/backend/

# Crear carpeta versions en el host (docker cp no copia carpetas vacías)
mkdir -p ~/projects/brewery-app/backend/alembic/versions
touch ~/projects/brewery-app/backend/alembic/versions/.gitkeep
```

---

## Estructura de archivos

```
backend/
├── alembic.ini
└── alembic/
    ├── env.py
    ├── script.py.mako
    └── versions/
        ├── .gitkeep
        └── abc123_create_tables.py
```

---

## Configuración — alembic.ini

```ini
sqlalchemy.url = postgresql+asyncpg://brewery:password@brewery-db:5432/brewery_db
```

---

## Configuración — env.py (async)

```python
import asyncio
import os
from logging.config import fileConfig
from sqlalchemy.ext.asyncio import create_async_engine
from alembic import context
from src.db.base import Base
from src.db.models import recipe, batch  # CRITICO: importar todos los modelos

config = context.config
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata
DATABASE_URL = os.getenv("DATABASE_URL")

def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=target_metadata)
    with context.begin_transaction():
        context.run_migrations()

async def run_async_migrations() -> None:
    connectable = create_async_engine(DATABASE_URL)
    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)
    await connectable.dispose()

def run_migrations_online() -> None:
    asyncio.run(run_async_migrations())

if context.is_offline_mode():
    pass
else:
    run_migrations_online()
```

> Error mas comun: olvidar importar los modelos en env.py.
> Sin los imports, autogenerate no detecta las tablas — genera migración vacía sin avisar.

---

## Flujo de trabajo

```bash
# 1. Modificar modelo SQLAlchemy

# 2. Generar migración
docker exec brewery-api alembic revision --autogenerate -m "descripción"

# 3. Copiar al host
docker cp brewery-api:/app/alembic/versions/. ~/projects/brewery-app/backend/alembic/versions/

# 4. Revisar el archivo generado

# 5. Aplicar
docker exec brewery-api alembic upgrade head

# 6. Commitear
git add backend/alembic/versions/ backend/src/db/models/
git commit -m "feat: descripción del cambio de esquema"
```

---

## Comandos principales

```bash
# Generar migración automática
docker exec brewery-api alembic revision --autogenerate -m "descripción"

# Aplicar todas las pendientes
docker exec brewery-api alembic upgrade head

# Revertir la última
docker exec brewery-api alembic downgrade -1

# Ver historial
docker exec brewery-api alembic history

# Ver estado actual
docker exec brewery-api alembic current
```

---

## alembic_version — control de estado

```sql
SELECT * FROM alembic_version;
-- version_num: 653336fca96a  (hash de la última migración aplicada)
```

Si `upgrade head` no muestra "Running upgrade" — ya está actualizado, correcto.

---

## Añadir modelos nuevos

Cada nuevo modelo debe importarse en `env.py`:

```python
from src.db.models import recipe, batch, fermentation, socio
```

---

## Integración con Dockerfile

```dockerfile
COPY src/ ./src/
COPY alembic/ ./alembic/
COPY alembic.ini .
```

---

## Troubleshooting

| Problema | Causa | Solución |
| --- | --- | --- |
| `--autogenerate` genera migración vacía | Modelos no importados en `env.py` | Añadir imports en `env.py` |
| Migración generada pero no en host | No se copió con `docker cp` | `docker cp brewery-api:/app/alembic/versions/. backend/alembic/versions/` |
| `ModuleNotFoundError` al ejecutar alembic | Archivos no copiados en imagen | Añadir `COPY alembic/` en Dockerfile y reconstruir |
| Error de conexión al generar migración | PostgreSQL no accesible | Verificar que `brewery-db` está corriendo |
