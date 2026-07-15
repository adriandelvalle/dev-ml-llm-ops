# SQLAlchemy Cheatsheet

Quick reference for SQLAlchemy 2 async ORM with FastAPI.
Last updated: 2026-07-15

---

## Qué es un ORM

Object-Relational Mapper — traduce automáticamente entre clases Python y tablas SQL.

```
Sin ORM:  cursor.execute("INSERT INTO recipes ...") → tuplas sin nombre
Con ORM:  db.add(recipe) → acceso por atributo (recipe.name, recipe.style)
```

---

## Instalación

```
# requirements.txt
sqlalchemy==2.0.41
asyncpg==0.30.0    # driver async para PostgreSQL
```

---

## Base — raíz de todos los modelos

```python
# src/db/base.py
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass
```

Todos los modelos SQLAlchemy heredan de `Base`. Alembic usa `Base.metadata`
para detectar las tablas y generar migraciones automáticamente.

---

## Definir un modelo

```python
from sqlalchemy import String, Numeric, Integer, Text, Date, ForeignKey
from sqlalchemy.orm import Mapped, mapped_column, relationship
from sqlalchemy.sql import func
from datetime import datetime, date
from src.db.base import Base

class Recipe(Base):
    __tablename__ = "recipes"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(100), nullable=False)
    style: Mapped[str] = mapped_column(String(20), nullable=False)
    batch_size_liters: Mapped[float] = mapped_column(Numeric(5, 2), nullable=False)
    target_og: Mapped[float] = mapped_column(Numeric(5, 3), nullable=False)
    target_fg: Mapped[float] = mapped_column(Numeric(5, 3), nullable=False)
    target_ibu: Mapped[int | None] = mapped_column(Integer, nullable=True)
    target_abv: Mapped[float | None] = mapped_column(Numeric(4, 2), nullable=True)
    notes: Mapped[str | None] = mapped_column(Text, nullable=True)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())

    # Relationship — acceso directo a los batches sin escribir JOIN
    batches: Mapped[list["Batch"]] = relationship("Batch", back_populates="recipe")
```

---

## Paralelismo Pydantic vs SQLAlchemy

| Aspecto | Pydantic | SQLAlchemy |
| --- | --- | --- |
| Propósito | Validar datos de la API | Persistir datos en PostgreSQL |
| Herencia | `BaseModel` | `Base` (DeclarativeBase) |
| Campos | `field: type = Field(...)` | `field: Mapped[type] = mapped_column(...)` |
| Nullable | `Optional[str]` / `str \| None` | `nullable=True` |
| Auto-generado | no aplica | `server_default=func.now()` |

No se sustituyen — se complementan. Pydantic valida entrada/salida de la API,
SQLAlchemy define y gestiona la persistencia en disco.

---

## Tres capas del modelo

```
RecipeCreate    ← lo que recibe la API (Pydantic, sin id ni timestamps)
RecipeResponse  ← lo que devuelve la API (Pydantic, con id y timestamps)
Recipe (DB)     ← lo que vive en PostgreSQL (SQLAlchemy)
```

---

## Foreign key y relationship

```python
class Batch(Base):
    __tablename__ = "batches"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    recipe_id: Mapped[int] = mapped_column(Integer, ForeignKey("recipes.id"), nullable=False)
    # ...

    # Acceso directo al objeto Recipe sin JOIN manual
    recipe: Mapped["Recipe"] = relationship("Recipe", back_populates="batches")
```

`ForeignKey("recipes.id")` = `REFERENCES recipes(id)` en SQL puro.
`relationship` añade acceso Python — `batch.recipe` devuelve el objeto `Recipe` completo.

---

## session.py — conexión y ciclo de vida

```python
# src/db/session.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL")

engine = create_async_engine(DATABASE_URL, echo=True)

AsyncSessionLocal = sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```

**`echo=True`** — imprime el SQL generado en logs. Útil en desarrollo, desactivar en producción.

**`expire_on_commit=False`** — los objetos siguen accesibles después del commit.

**`get_db`** — dependencia FastAPI. Cada petición abre una sesión y la cierra automáticamente.

---

## Operaciones CRUD básicas (async)

```python
from sqlalchemy import select

# CREATE
new_recipe = Recipe(name="Asturian Pale Ale", style="APA", ...)
db.add(new_recipe)
await db.commit()
await db.refresh(new_recipe)  # recarga el objeto con id y created_at generados
return new_recipe

# READ — por id
result = await db.execute(select(Recipe).where(Recipe.id == recipe_id))
recipe = result.scalar_one_or_none()  # None si no existe

# READ — todos
result = await db.execute(select(Recipe))
recipes = result.scalars().all()

# UPDATE
recipe.status = "brewing"
await db.commit()

# DELETE
await db.delete(recipe)
await db.commit()
```

---

## Usar get_db en un endpoint FastAPI

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from src.db.session import get_db
from src.db.models.recipe import Recipe
from src.models.recipe import RecipeCreate, RecipeResponse

router = APIRouter()

@router.get("/{recipe_id}", response_model=RecipeResponse)
async def get_recipe(recipe_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Recipe).where(Recipe.id == recipe_id))
    recipe = result.scalar_one_or_none()
    if not recipe:
        raise HTTPException(status_code=404, detail=f"Recipe {recipe_id} not found")
    return recipe
```

---

## Troubleshooting

| Problema | Causa | Solución |
| --- | --- | --- |
| `NameError: relationship not defined` | Falta importar `relationship` | `from sqlalchemy.orm import Mapped, mapped_column, relationship` |
| `greenlet_spawn has not been called` | Usando sync en contexto async | Usar `create_async_engine` y `AsyncSession` |
| Objeto inaccesible tras commit | `expire_on_commit=True` (default) | Usar `expire_on_commit=False` en sessionmaker |
| `DATABASE_URL` es `None` | Variable de entorno no cargada | Verificar `.env` y que Docker Compose lo pasa al contenedor |
