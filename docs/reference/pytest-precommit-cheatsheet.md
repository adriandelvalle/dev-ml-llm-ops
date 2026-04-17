# pytest + pre-commit Cheatsheet

Quick reference for testing and commit quality automation.
Last updated: 2026-04-17

---

## pytest — Ejecutar tests

```bash
# Siempre desde backend/
cd ~/projects/brewery-app/backend
source venv/bin/activate

# Todos los tests
pytest -v

# Solo un archivo
pytest -v tests/test_recipes.py

# Solo tests cuyo nombre contiene una palabra
pytest -v -k "test_get"

# Parar al primer fallo
pytest -v -x

# Ver output de print() dentro de los tests
pytest -v -s
```

---

## pytest — Estructura obligatoria

```
backend/
├── tests/
│   ├── __init__.py          # hace la carpeta un módulo Python
│   ├── conftest.py          # fixtures compartidos entre archivos
│   ├── test_recipes.py      # archivo de tests — debe empezar por test_
│   └── test_batches.py
└── pytest.ini               # configuración de pytest
```

> pytest descubre tests automáticamente buscando archivos `test_*.py`
> y funciones `def test_*()`. Si no sigues la convención, no los encuentra.

---

## pytest.ini — Configuración

```ini
[pytest]
asyncio_mode = auto
testpaths = tests
```

---

## Anatomía de un test

```python
def test_nombre_descriptivo_del_comportamiento(client):
    # 1. Arrange — prepara los datos
    payload = {"name": "Gijon Stout", "style": "STOUT", ...}

    # 2. Act — ejecuta la acción
    response = client.post("/api/v1/recipes/", json=payload)

    # 3. Assert — verifica el resultado
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Gijon Stout"
```

El patrón **Arrange / Act / Assert** es el estándar universal de testing.

---

## assert — referencia rápida

```python
assert response.status_code == 200      # igualdad
assert response.status_code != 500      # desigualdad
assert len(response.json()) == 2        # longitud
assert "id" in data                     # clave existe en dict
assert data["final_fg"] is None         # valor es null
assert data["status"] == "planned"      # string exacto
```

---

## Códigos HTTP más usados en tests

| Código | Significado | Cuándo esperarlo |
| --- | --- | --- |
| 200 | OK | GET exitoso |
| 201 | Created | POST exitoso |
| 404 | Not Found | ID que no existe |
| 422 | Unprocessable Entity | Datos inválidos — Pydantic los rechaza |
| 500 | Internal Server Error | Bug en el servidor — nunca debe aparecer |

---

## conftest.py — Fixtures

```python
import pytest
from fastapi.testclient import TestClient
from src.main import app
from src.core import mock_data

@pytest.fixture(autouse=True)
def reset_mock_data():
    """Se ejecuta automáticamente antes de cada test."""
    mock_data.RECIPES.clear()
    mock_data.RECIPES.update({
        1: mock_data.RECIPES_DEFAULT[1],
        2: mock_data.RECIPES_DEFAULT[2],
    })
    mock_data.next_recipe_id = 3
    mock_data.next_batch_id = 2

@pytest.fixture
def client():
    """Cliente HTTP en memoria — no necesita el servidor corriendo."""
    return TestClient(app)
```

**`autouse=True`** — se ejecuta antes de cada test automáticamente.
**`TestClient(app)`** — habla directamente con FastAPI sin uvicorn.

---

## Reglas de testing profesional

| Regla | Explicación |
| --- | --- |
| Un test por comportamiento | No testear el mismo caso con 100 datos distintos |
| Tests independientes | Cada test parte de estado limpio — usa reset en conftest |
| Nombres descriptivos | `test_create_recipe_invalid_style` no `test_3` |
| Acceso por módulo | `mock_data.next_id` no `from mock_data import next_id` |
| Tests son contratos | Verifican comportamiento futuro, no solo estado actual |

---

## pre-commit — Referencia rápida

### Instalación (una vez por repo)

```bash
cd ~/projects/brewery-app          # raíz del repo, no backend/
source backend/venv/bin/activate
pre-commit install                 # hook pre-commit (archivos)
pre-commit install --hook-type commit-msg  # hook commit-msg (mensaje)
```

### Ejecutar manualmente sobre todos los archivos

```bash
pre-commit run --all-files
```

### Actualizar versiones de los hooks

```bash
pre-commit autoupdate
```

---

## .pre-commit-config.yaml

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace    # espacios al final de línea
      - id: end-of-file-fixer      # salto de línea al final del archivo
      - id: check-yaml             # sintaxis YAML válida
      - id: check-json             # sintaxis JSON válida

  - repo: https://github.com/commitizen-tools/commitizen
    rev: v4.13.10
    hooks:
      - id: commitizen
        stages: [commit-msg]       # valida el mensaje del commit
```

---

## .cz.toml — Configuración de commitizen

```toml
[tool.commitizen]
name = "cz_conventional_commits"
version = "0.1.0"
tag_format = "v$version"
update_changelog_on_bump = true
```

---

## Conventional Commits — tipos válidos

| Tipo | Cuándo |
| --- | --- |
| `feat:` | Nueva funcionalidad |
| `fix:` | Corrección de bug |
| `docs:` | Solo documentación |
| `test:` | Añadir o corregir tests |
| `chore:` | Mantenimiento, configs |
| `refactor:` | Cambios sin nueva funcionalidad ni bug |
| `ci:` | Cambios en CI/CD |
| `style:` | Formato, sin cambios de lógica |
| `perf:` | Mejoras de rendimiento |
| `build:` | Cambios en dependencias |

---

## Hooks instalados y dónde viven

```
.git/hooks/
├── pre-commit      # verifica archivos antes del commit
└── commit-msg      # verifica el mensaje del commit
```

> Estos archivos los genera `pre-commit install` automáticamente.
> No los edites a mano. No los commitees — están en .gitignore por defecto.

---

## Flujo completo con pre-commit activo

```bash
git add .
git commit -m "feat: add fermentation sample endpoint"

# Pre-commit ejecuta automáticamente:
# trailing-whitespace......Passed
# end-of-file-fixer........Passed
# check-yaml...............Passed
# commitizen check.........Passed

# Solo entonces:
# [main abc1234] feat: add fermentation sample endpoint
```

---

## Troubleshooting

| Problema | Solución |
| --- | --- |
| `ModuleNotFoundError` en pytest | Ejecutar desde `backend/`, no desde raíz |
| Test falla por orden de ejecución | Añadir reset en `conftest.py` con `autouse=True` |
| pre-commit no se ejecuta | Verificar con `pre-commit install` que está instalado |
| Commit rechazado por commitizen | Revisar formato: `tipo: descripción en imperativo` |
| pre-commit lento la primera vez | Normal — descarga y cachea los hooks, luego es instantáneo |
