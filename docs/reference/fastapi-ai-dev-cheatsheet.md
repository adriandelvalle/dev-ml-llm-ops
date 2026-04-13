# FastAPI + Pydantic + OpenCode + Ollama Cheatsheet

Quick reference for local AI-assisted Python backend development.
Last updated: 2026-04-13

---

## Python Environment

| Action | Command |
| --- | --- |
| Create venv | `python3 -m venv backend/venv` |
| Activate | `source backend/venv/bin/activate` |
| Deactivate | `deactivate` |
| Install deps | `pip install -r backend/requirements.txt` |
| Freeze deps | `pip freeze > backend/requirements.txt` |

---

## FastAPI & Uvicorn

### Arranque correcto — siempre desde `backend/`

```bash
cd ~/projects/brewery-app/backend
source venv/bin/activate
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

> **Por qué desde `backend/`**: Python resuelve imports relativos al directorio
> desde donde se lanza uvicorn. Desde `backend/`, `src` es un módulo de primer nivel.
> Desde la raíz del proyecto, `src` no existe — existe `backend.src`, lo que rompe
> todos los imports internos con `ModuleNotFoundError: No module named 'src'`.
> Esto desaparecerá con Docker (Semana 4) porque el `WORKDIR` del contenedor lo gestiona.

| Action | Command |
| --- | --- |
| Health check | `curl http://localhost:8000/health` |
| Swagger UI | `http://localhost:8000/docs` |
| ReDoc | `http://localhost:8000/redoc` |
| Acceso red local | `http://192.168.0.21:8000/docs` |

---

## Pydantic — Referencia rápida

### Modelo base
```python
from pydantic import BaseModel, Field
from typing import Optional

class RecipeBase(BaseModel):
    name: str = Field(..., min_length=2, max_length=100)   # obligatorio
    batch_size_liters: float = Field(..., gt=0, le=100)    # obligatorio, rango
    notes: Optional[str] = Field(None, max_length=1000)   # opcional
```

### Operadores de Field()
| Operador | Significado | Ejemplo |
| --- | --- | --- |
| `...` | Campo obligatorio | `Field(...)` |
| `None` | Opcional, default null | `Field(None)` |
| `gt` | greater than (>) | `Field(..., gt=0)` |
| `ge` | greater or equal (>=) | `Field(..., ge=0)` |
| `lt` | less than (<) | `Field(..., lt=100)` |
| `le` | less or equal (<=) | `Field(..., le=100)` |
| `min_length` | longitud mínima string | `Field(..., min_length=2)` |
| `max_length` | longitud máxima string | `Field(..., max_length=100)` |
| `description` | texto para Swagger | `Field(..., description="OG objetivo")` |

### Patrón Create / Response (estándar profesional)
```python
class RecipeBase(BaseModel):
    """Campos comunes."""
    name: str
    style: BeerStyle

class RecipeCreate(RecipeBase):
    """Lo que recibe la API. Nunca incluir id ni timestamps."""
    pass

class RecipeResponse(RecipeBase):
    """Lo que devuelve la API. Añade campos generados por el sistema."""
    id: int
    created_at: str
```

### Composición de modelos
```python
class BatchMeasurements(BaseModel):
    """Modelo separado porque los datos llegan en fases distintas del proceso."""
    pre_boil_og: Optional[float] = Field(None)
    final_fg: Optional[float] = Field(None)

class BatchResponse(BatchBase):
    id: int
    measurements: BatchMeasurements   # modelo anidado
```

### Enums con dominio real
```python
from enum import Enum

class BeerStyle(str, Enum):   # hereda str para serialización JSON directa
    IPA = "IPA"
    NEIPA = "NEIPA"
    APA = "APA"
```

---

## Estructura de proyecto

```
backend/
├── src/
│   ├── main.py            # solo arranca la app y registra routers
│   ├── api/
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── recipes.py     # endpoints de recetas
│   │       └── batches.py     # endpoints de lotes
│   ├── models/
│   │   ├── __init__.py
│   │   ├── recipe.py          # RecipeBase, RecipeCreate, RecipeResponse, BeerStyle
│   │   ├── batch.py           # BatchBase, BatchCreate, BatchResponse, BatchMeasurements
│   │   └── fermentation.py    # FermentationSample models
│   └── core/
│       ├── __init__.py
│       └── mock_data.py       # datos en memoria hasta PostgreSQL (Semana 5)
├── tests/                     # pytest (pendiente Semana 3)
├── venv/
└── requirements.txt
```

> **Por qué `api/v1/`**: versionado explícito. Permite introducir `/api/v2/`
> con cambios breaking sin romper clientes que ya consumen v1.

---

## Registrar routers en main.py

```python
from fastapi import FastAPI
from src.api.v1 import recipes, batches

app = FastAPI(title="Brewery App API", version="0.1.0")

app.include_router(recipes.router, prefix="/api/v1/recipes", tags=["recipes"])
app.include_router(batches.router, prefix="/api/v1/batches", tags=["batches"])
```

---

## Endpoints actuales

| Método | Ruta | Descripción |
| --- | --- | --- |
| GET | `/health` | Estado del servicio |
| GET | `/api/v1/recipes/` | Listar recetas |
| GET | `/api/v1/recipes/{id}` | Obtener receta por ID |
| POST | `/api/v1/recipes/` | Crear receta |
| GET | `/api/v1/batches/` | Listar lotes |
| GET | `/api/v1/batches/{id}` | Obtener lote por ID |
| POST | `/api/v1/batches/` | Crear lote |

---

## Testing con curl

```bash
# Listar recetas
curl -s http://localhost:8000/api/v1/recipes/ | python3 -m json.tool

# Crear receta
curl -s -X POST http://localhost:8000/api/v1/recipes/ \
  -H "Content-Type: application/json" \
  -d '{"name": "Gijon Stout", "style": "STOUT", "batch_size_liters": 50,
       "target_og": 1.060, "target_fg": 1.014, "target_ibu": 40, "target_abv": 6.0}' \
  | python3 -m json.tool

# Obtener lote por ID
curl -s http://localhost:8000/api/v1/batches/1 | python3 -m json.tool
```

---

## Port Forwarding & Acceso Remoto

| Escenario | Solución |
| --- | --- |
| VS Code cerrado | Reabrir VS Code → pestaña PORTS → restaurar 8000 |
| Sin VS Code | `ssh -L 8000:localhost:8000 jota@192.168.0.21` |
| Acceso directo red local | `http://192.168.0.21:8000/docs` |
| Persistencia del servicio | Docker `restart: always` — Semana 4 |

---

## .gitignore — regla crítica para modelos

```gitignore
# CORRECTO — específico por ruta
/mlops/models/
/llmops/models/
*.gguf
*.bin

# INCORRECTO — demasiado agresivo, ignora también src/models/ de Pydantic
models/
```

> Lección: siempre hacer `git status` antes de commit y verificar
> que los archivos esperados aparecen en staging.

---

## OpenCode + Ollama

| Task | Prompt / Command |
| --- | --- |
| Start AI | `opencode` |
| Create endpoint | `"Create a GET /items endpoint with Pydantic validation"` |
| Refactor | `"Refactor this function to use async/await and add error handling"` |
| Add tests | `"Generate pytest unit tests for this endpoint"` |