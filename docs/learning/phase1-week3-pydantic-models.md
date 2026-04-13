# Fase 1, Semana 3: Pydantic Models & API Structure

## Fecha
2026-04-13

## Objetivo
Añadir validación de datos real a la API con Pydantic, definir los modelos de dominio
de la cervecería, y estructurar el proyecto de forma mantenible y profesional.

## Estado al inicio de la sesión
- FastAPI corriendo con un único endpoint `/health`
- Todo el código en `backend/src/main.py`
- Sin modelos, sin validación, sin estructura

## Estado al final de la sesión
- API con 7 endpoints funcionales bajo `/api/v1/`
- Modelos Pydantic para Recipe, Batch y FermentationSample
- Estructura de carpetas profesional
- Mock data en memoria como capa de datos temporal
- Bug de `.gitignore` detectado y corregido

---

## Conceptos aprendidos

### ¿Qué es Pydantic?
Librería Python que permite definir la estructura y reglas de los datos como clases.
Valida, convierte y documenta automáticamente sin necesidad de escribir código defensivo manual.

**Sin Pydantic** — esto para cada campo:
```python
if "litros" not in data:
    raise ValueError("falta litros")
if not isinstance(data["litros"], (int, float)):
    try:
        data["litros"] = float(data["litros"])
    except:
        raise ValueError("litros debe ser número")
```

**Con Pydantic** — esto una vez para todos los campos:
```python
class Lote(BaseModel):
    litros: float
```

Las tres cosas que hace automáticamente:
1. **Valida** — rechaza datos que no cumplen las reglas
2. **Convierte** — `"500"` → `500.0` si es seguro hacerlo
3. **Documenta** — FastAPI genera el Swagger UI automáticamente a partir de los modelos

### Patrón Create / Response
El modelo que recibe la API no es el mismo que devuelve. Razón: campos como `id`,
`status` o `created_at` los genera el sistema — si los aceptaras en la entrada,
cualquiera podría falsificarlos.

```
RecipeCreate  →  lo que entra  (sin id, sin created_at)
RecipeResponse →  lo que sale  (con id, con created_at)
```

En el futuro añadiremos un tercero: `RecipeDB` para lo que vive en PostgreSQL.

### Composición de modelos
Modelos que contienen otros modelos para reflejar que los datos tienen ciclos de vida
diferentes.

`BatchMeasurements` es un modelo separado dentro de `BatchResponse` porque:
- Las mediciones no existen cuando se crea el lote — se acumulan durante el proceso
- Permite un endpoint `PATCH /batches/{id}/measurements` independiente
- Prepara la estructura para cuando las mediciones vivan en su propia tabla en PostgreSQL

### Enums con dominio real
Vocabulario controlado para valores que solo pueden ser uno de un conjunto definido.

```python
class BeerStyle(str, Enum):
    IPA = "IPA"
    LAGER = "LAGER"
    NEIPA = "NEIPA"
    APA = "APA"
    STOUT = "STOUT"
    PORTER = "PORTER"
    WHEAT = "WHEAT"
```

Heredar de `str` además de `Enum` hace el valor directamente serializable a JSON.
Si alguien manda `"style": "INVENTADA"` la API lo rechaza automáticamente.

### Field() — validación declarativa
```python
name: str = Field(..., min_length=2, max_length=100)
batch_size_liters: float = Field(..., gt=0, le=100)
target_ibu: Optional[int] = Field(None, ge=0, le=120)
```

- `...` significa campo obligatorio
- `None` significa campo opcional con valor por defecto null
- `gt`, `lt`, `ge`, `le` son greater than, less than, greater or equal, less or equal
- `min_length`, `max_length` para strings

### Versionado de API
Los endpoints viven bajo `/api/v1/` deliberadamente. Permite introducir
`/api/v2/` con cambios breaking sin afectar clientes que ya usan v1.

---

## Estructura creada

```
backend/src/
├── main.py                    # Solo arranca la app y registra routers
├── api/
│   └── v1/
│       ├── __init__.py
│       ├── recipes.py         # GET / POST /api/v1/recipes/
│       └── batches.py         # GET / POST /api/v1/batches/
├── models/
│   ├── __init__.py
│   ├── recipe.py              # RecipeBase, RecipeCreate, RecipeResponse, BeerStyle
│   ├── batch.py               # BatchBase, BatchCreate, BatchResponse, BatchStatus, BatchMeasurements
│   └── fermentation.py        # FermentationSampleBase, Create, Response
└── core/
    ├── __init__.py
    └── mock_data.py           # Datos en memoria hasta PostgreSQL (Semana 5)
```

---

## Endpoints implementados

| Método | Ruta | Descripción |
| --- | --- | --- |
| GET | `/health` | Estado del servicio |
| GET | `/api/v1/recipes/` | Listar todas las recetas |
| GET | `/api/v1/recipes/{id}` | Obtener receta por ID |
| POST | `/api/v1/recipes/` | Crear nueva receta |
| GET | `/api/v1/batches/` | Listar todos los lotes |
| GET | `/api/v1/batches/{id}` | Obtener lote por ID |
| POST | `/api/v1/batches/` | Crear nuevo lote |

---

## Comando de arranque correcto

```bash
# Siempre desde backend/ — no desde la raíz del proyecto
cd ~/projects/brewery-app/backend
source venv/bin/activate
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

**Por qué desde `backend/`**: Python resuelve los imports relativos al directorio
desde donde se lanza uvicorn. Desde `backend/`, `src` es un módulo de primer nivel.
Desde la raíz, `src` no existe — existe `backend.src`, lo que rompe todos los imports
internos. Esto desaparecerá cuando tengamos Docker (Semana 4) porque el `WORKDIR`
del contenedor lo gestiona.

---

## Bug detectado y corregido: .gitignore demasiado agresivo

**Problema**: la regla `models/` en `.gitignore` ignoraba silenciosamente
`backend/src/models/` — los modelos Pydantic nunca llegaron al repo en el primer commit.

**Causa**: la regla fue diseñada para ignorar carpetas de modelos ML (pesos de redes
neuronales, LLMs descargados) pero al ser genérica afectaba a cualquier carpeta
llamada `models/` en cualquier nivel del proyecto.

**Corrección**: reglas específicas con ruta absoluta desde la raíz del repo:
```
# ML model artifacts (weights, checkpoints) - NOT Pydantic models
/mlops/models/
/llmops/models/
*.bin
```

**Lección**: un `.gitignore` agresivo puede excluir código crítico sin ningún aviso.
Siempre revisar `git status` antes de un commit y verificar que los archivos esperados
aparecen en staging.

---

## Deuda técnica pendiente (Semana 3)

| Item | Estado | Semana |
| --- | --- | --- |
| pytest + httpx: primeros tests | ⏳ Pendiente | 3 |
| pre-commit + commitizen | ⏳ Pendiente | 3 |
| Endpoints FermentationSample | ⏳ Pendiente | 3 |
| PATCH /batches/{id}/measurements | ⏳ Pendiente | 3 |

---

## Validación en vivo — lo que vimos funcionar

```bash
# Estilo inventado → rechazado con mensaje exacto
curl -X POST /api/v1/recipes/ -d '{"style": "INVENTADA", ...}'
# → "Input should be 'IPA', 'LAGER', 'NEIPA', 'APA', 'STOUT', 'PORTER' or 'WHEAT'"

# Campos obligatorios ausentes → lista completa de errores
curl -X POST /api/v1/recipes/ -d '{"name": "Test"}'
# → 4 errores: style, batch_size_liters, target_og, target_fg — todos a la vez

# Datos correctos → RecipeResponse con id y created_at generados por el sistema
curl -X POST /api/v1/recipes/ -d '{"name": "Gijon Stout", "style": "STOUT", ...}'
# → {"id": 3, "created_at": "2026-04-13T11:03:36", ...}
```