# Fase 2, Semana 5: PostgreSQL, Docker Compose, SQLAlchemy & Alembic

## Fecha
2026-06-24 / 2026-07-15

## Objetivo
Sustituir `mock_data.py` por persistencia real con PostgreSQL, montar el stack
completo con Docker Compose, y definir el esquema de base de datos con SQLAlchemy
y Alembic.

## Estado al inicio
- API con mock data en memoria — datos se pierden al reiniciar el contenedor
- Contenedores arrancados individualmente con `docker run`
- Sin ORM, sin migraciones, sin base de datos real

## Estado al final
- PostgreSQL 16 corriendo en Docker con volumen persistente
- Docker Compose gestiona todo el stack en un solo archivo
- SQLAlchemy 2 con modelos `Recipe` y `Batch` definidos
- Alembic con primera migración aplicada — tablas creadas en PostgreSQL
- `.env` + `.env.example` como patrón pre-Vault para credenciales

---

## SQL puro con psql — conceptos fijados con las manos

### Tablas y tipos

Una tabla SQL es conceptualmente idéntica a un modelo Pydantic — columnas = campos,
filas = instancias. La diferencia es que los datos viven en disco, no en memoria.

```sql
CREATE TABLE recipes (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    style VARCHAR(20) NOT NULL,
    batch_size_liters NUMERIC(5,2) NOT NULL,
    target_og NUMERIC(5,3) NOT NULL,
    target_fg NUMERIC(5,3) NOT NULL,
    target_ibu INTEGER,
    target_abv NUMERIC(4,2),
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

**`SERIAL PRIMARY KEY`** — contador autoincrementado gestionado por PostgreSQL.
Equivalente al `next_recipe_id` que llevábamos a mano en `mock_data.py`.

**`NUMERIC(5,2)`** — nunca `FLOAT` para medidas de precisión. 5 cifras totales,
2 decimales. Exacto, sin errores de redondeo binario.

**`DEFAULT NOW()`** — el equivalente a `datetime.now().isoformat()` pero gestionado
por la base de datos, no por el código.

### Foreign keys e integridad referencial

Una foreign key apunta a **una fila concreta** de otra tabla (identificada por su
primary key), no a la tabla en abstracto.

```sql
CREATE TABLE batches (
    id SERIAL PRIMARY KEY,
    recipe_id INTEGER NOT NULL REFERENCES recipes(id),
    ...
);
```

**Protección en inserción**: no puedes crear un batch con `recipe_id` que no exista.
**Protección en borrado**: no puedes borrar una receta que tenga batches apuntando a ella.

Comportamiento por defecto: `ON DELETE RESTRICT` — bloquea el borrado. Correcto
para datos de negocio donde el historial importa.

### Secuencias — nunca retroceden

Cuando un INSERT falla (por foreign key u otro motivo), la secuencia ya ha consumido
el número. El siguiente INSERT exitoso recibe el número siguiente, no el fallido.
Consecuencia: los IDs pueden tener huecos. **Nunca asumir que los IDs son consecutivos**.

### UTC vs hora local

PostgreSQL opera en UTC por defecto (`SHOW timezone` → `Etc/UTC`). El servidor
puede estar en CEST (UTC+2) pero los timestamps se guardan en UTC. Diferencia
de 2 horas entre `created_at` y la hora local — comportamiento correcto y deliberado.

Regla: guardar siempre en UTC, convertir a hora local solo en la capa de presentación.

### Soft delete — patrón empresarial

En aplicaciones reales de negocio, no se borran filas — se marcan como inactivas:

```sql
deleted_at TIMESTAMP  -- NULL = activo, timestamp = borrado
is_active BOOLEAN DEFAULT TRUE
```

Ventaja: historial preservado, recuperación posible, sin problemas de foreign key.
A implementar cuando lleguemos al diseño completo del modelo.

### Comandos psql útiles

```sql
\dt                    -- listar tablas
\d nombre_tabla        -- describir estructura de una tabla
\q                     -- salir
SHOW timezone;         -- ver timezone activo
SELECT * FROM alembic_version;  -- ver estado de migraciones
```

---

## Docker Compose

### Por qué sustituye a los docker run individuales

Antes: 4 comandos `docker run` con flags, redes y volúmenes que hay que recordar.
Ahora: un solo archivo declarativo + un solo comando.

```bash
docker compose up -d      # levanta todo el stack
docker compose down       # para y elimina contenedores (no volúmenes)
docker compose build      # reconstruye imágenes
docker compose logs       # logs de todos los servicios
```

### container_name — evitar nombres con prefijo y número

Sin `container_name`, Compose genera nombres como `brewery-app-brewery-api-1`.
Con `container_name: brewery-api`, el nombre es fijo y predecible — esencial
para aliases, scripts y referencias entre servicios.

### depends_on — orden de arranque

```yaml
brewery-api:
  depends_on:
    - brewery-db
```

Garantiza que `brewery-db` arranca antes que `brewery-api`. No garantiza que
PostgreSQL esté listo para aceptar conexiones — solo que el contenedor está
iniciado. Para casos críticos se añade un healthcheck (pendiente).

### Variables de entorno desde .env

Compose lee automáticamente el `.env` del mismo directorio y sustituye
`${VARIABLE}` en el archivo. Las credenciales nunca van hardcodeadas en
`docker-compose.yml`.

### .env vs .env.example

```
.env          ← credenciales reales, gitignored (600)
.env.example  ← plantilla pública, commiteada, valores ficticios
```

El `.gitignore` ya tenía `!.env.example` — preparado desde el inicio.

---

## SQLAlchemy 2

### Qué es un ORM

Object-Relational Mapper — traduce automáticamente entre clases Python y tablas SQL.
Sin ORM: SQL escrito como texto plano, resultados como tuplas sin nombre.
Con ORM: clases Python tipadas, acceso por nombre de atributo, SQL generado automáticamente.

### Paralelismo Pydantic vs SQLAlchemy

```python
# Pydantic — forma del dato para la API
class RecipeResponse(BaseModel):
    id: int
    name: str
    style: str

# SQLAlchemy — tabla real en PostgreSQL
class Recipe(Base):
    __tablename__ = "recipes"
    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    name: Mapped[str] = mapped_column(String(100), nullable=False)
    style: Mapped[str] = mapped_column(String(20), nullable=False)
```

Misma estructura conceptual, propósito distinto. No se sustituyen — se complementan.
Pydantic valida la entrada/salida de la API. SQLAlchemy define y gestiona la persistencia.

### Relationships — JOIN automático

```python
# En Recipe:
batches: Mapped[list["Batch"]] = relationship("Batch", back_populates="recipe")

# En Batch:
recipe: Mapped["Recipe"] = relationship("Recipe", back_populates="batches")
```

`batch.recipe` devuelve el objeto `Recipe` completo sin escribir ningún JOIN.
SQLAlchemy genera el SQL por debajo.

### session.py — conexión y ciclo de vida

```python
engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```

`get_db` es una dependencia FastAPI — cada petición abre una sesión, opera,
y la cierra automáticamente. `echo=True` imprime el SQL generado — útil en
desarrollo, desactivar en producción.

### Tres capas del modelo — patrón completo

```
RecipeCreate    ← lo que recibe la API (Pydantic)
RecipeResponse  ← lo que devuelve la API (Pydantic)
Recipe (DB)     ← lo que vive en PostgreSQL (SQLAlchemy)
```

`mock_data.py` desaparece — era el sustituto temporal de la capa DB.

---

## Alembic

### Qué es y por qué existe

Gestor de migraciones para SQLAlchemy. Una migración es un archivo Python que
describe un cambio en el esquema de la base de datos — `CREATE TABLE`,
`ALTER TABLE`, `DROP COLUMN`, etc.

Sin Alembic: cambiar un modelo SQLAlchemy requiere tirar la base de datos y
recrearla. Con Alembic: los cambios son incrementales, reversibles y versionados.

### Flujo completo

```bash
# 1. Generar migración automáticamente desde los modelos
alembic revision --autogenerate -m "descripción del cambio"

# 2. Revisar el archivo generado en alembic/versions/
# Alembic puede no detectar todos los cambios (ej: renombrado de columnas)

# 3. Aplicar
alembic upgrade head

# 4. Revertir (si hay un error)
alembic downgrade -1
```

### alembic_version — control de estado

Alembic crea automáticamente la tabla `alembic_version` en la base de datos.
Contiene el hash de la última migración aplicada. Así sabe qué migraciones
están pendientes en cada `upgrade head`.

### env.py — pieza crítica

Los modelos SQLAlchemy deben importarse explícitamente en `env.py`:

```python
from src.db.models import recipe, batch  # sin esto, Alembic no los detecta
target_metadata = Base.metadata
```

Sin estos imports, `--autogenerate` no detecta ninguna tabla — error silencioso
y muy común la primera vez.

### Archivos en el repo

```
backend/
├── alembic.ini                    # configuración: DATABASE_URL, rutas
├── alembic/
│   ├── env.py                     # configuración async + imports de modelos
│   ├── script.py.mako             # plantilla para nuevas migraciones
│   └── versions/
│       ├── .gitkeep               # mantiene la carpeta en Git aunque esté vacía
│       └── 653336fca96a_create_recipes_and_batches_tables.py
```

---

## Infraestructura resuelta esta semana

### Fix Netplan/cloud-init

jotasrv tenía dos archivos de configuración de red contradictorios:
- `00-installer-config.yaml` → IP estática `192.168.0.21` ✅
- `50-cloud-init.yaml` → `dhcp4: true` ← generado por cloud-init, podía pisar la estática

Solución:
```bash
# Deshabilitar cloud-init para red
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
# contenido: network: {config: disabled}

sudo rm /etc/netplan/50-cloud-init.yaml
sudo netplan apply
```

Verificación: `ip -4 addr show eno1` → `valid_lft forever` confirma IP estática real.

### IP fija Windows

El router Sercom (Movistar/O2) no tiene DHCP Reservation en su interfaz.
La IP del Windows cambió de `.10` a `.15` tras un reinicio.

Solución: IP estática configurada directamente en Windows:
```powershell
netsh interface ip set address "Ethernet" static 192.168.0.15 255.255.255.0 192.168.0.1
netsh interface ip set dns "Ethernet" static 8.8.8.8
```

`/etc/fstab` actualizado con la nueva IP para el mount SMB.

### Decisiones tomadas

| Decisión | Resultado |
| --- | --- |
| Dominio | `.com` — más reconocible para público general |
| GitHub Pages | Descartado — solo estático, no sirve la API |
| Correo | Zoho Mail Lite (~6 cuentas) + Thunderbird — pendiente de activar |
| KB Tres Tigris | Syncthing + jotasrv + Obsidian — pendiente de dominio |
| ELK | Anotado en sandbox `devops/` — no en producción |

---

## Estado del stack

```
jotasrv
└── Docker Compose
    ├── brewery-db (postgres:16) — tablas: recipes, batches, alembic_version
    ├── brewery-api (FastAPI + SQLAlchemy + Alembic)
    ├── brewery-nginx (reverse proxy + static files)
    └── brewery-cloudflared (Cloudflare Tunnel)
```

---

## Pendiente

| Item | Estado |
| --- | --- |
| Conectar endpoints FastAPI a PostgreSQL (sustituir mock_data) | ⏳ Siguiente |
| Modelo FermentationSample en SQLAlchemy + migración | ⏳ Siguiente |
| pytest con base de datos de test (no mock_data) | ⏳ Siguiente |
| python-dotenv en la app para leer DATABASE_URL | ⏳ Siguiente |
| Modelo Socio — RGPD, cuota, renovación | ⏳ Semana 5 cont. |
| MinIO | ⏳ Semana 6 |
| HashiCorp Vault | ⏳ Semana 7 |
