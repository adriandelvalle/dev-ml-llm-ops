# Fase 1, Semana 3 (Parte 2): pytest, Testing & pre-commit

## Fecha
2026-04-17

## Objetivo
Añadir una batería de tests automatizados a la API y configurar pre-commit
para garantizar la calidad del código y los commits de forma automática.

## Estado al inicio de la sesión
- API con 7 endpoints funcionando y validación Pydantic
- Sin tests — cambios podían romper cosas sin saberlo
- Commits dependían de la memoria del desarrollador

## Estado al final de la sesión
- 14 tests cubriendo GET, POST y validación en recipes y batches
- pre-commit activo en brewery-app y portfolio
- commitizen rechaza automáticamente commits mal formateados

---

## Conceptos aprendidos

### ¿Qué es un test?

Un test es una pregunta con respuesta esperada. El mismo concepto que medir
la densidad final de un lote — tienes un valor esperado y compruebas el real.

```
Pregunta:  si llamo a GET /api/v1/recipes/1, ¿qué devuelve?
Esperado:  status 200 y name="Asturian Pale Ale"
Real:      lo que devuelve la API al ejecutar el test
Resultado: coinciden → PASSED / no coinciden → FAILED
```

### Por qué existen los tests

Sin tests, un cambio en el código puede romper cosas sin que nadie lo note
hasta que llega a producción. Con tests, `pytest` lo detecta en segundos.

En empresas con CI/CD, los tests se ejecutan automáticamente en cada commit.
Si alguno falla, el código no puede llegar a producción — es una red de
seguridad automática.

### Anatomía de un test — línea a línea

```python
def test_get_recipe_by_id(client):          # función que empieza por test_
    response = client.get("/api/v1/recipes/1")  # petición HTTP en memoria
    assert response.status_code == 200      # afirmo que respondió con éxito
    data = response.json()                  # convierto JSON a diccionario
    assert data["id"] == 1                  # afirmo que el id es 1
    assert data["name"] == "Asturian Pale Ale"  # afirmo el nombre
    assert data["style"] == "APA"           # afirmo el estilo
```

**`assert`** es la pieza central. Significa "afirmo que esto es verdad".
Si es verdad, el test continúa. Si no, falla en esa línea exacta y muestra
qué valor esperabas y qué recibiste.

### Un test por comportamiento, no por dato

No se testea que el id 999 devuelva 404, el 998 devuelva 404, el 997...
Se testea el comportamiento: *"cuando pido un ID que no existe, la API devuelve 404"*.
Un solo test cubre todos los IDs inexistentes.

Los tres comportamientos de un GET por ID:
1. El ID existe → 200 con datos correctos
2. El ID no existe → 404
3. El ID tiene formato inválido → 422

Tres comportamientos, tres tests. Independientemente de cuántos datos haya.

### Los tests son contratos, no snapshots

Un test no verifica el estado actual — verifica que el comportamiento
se mantiene en el futuro. Cuando llegue PostgreSQL y se reescriban los
endpoints, los mismos tests verifican que el contrato sigue cumpliéndose.
La implementación cambia, el contrato no.

---

## TestClient — el cliente HTTP en memoria

```python
from fastapi.testclient import TestClient
from src.main import app

client = TestClient(app)
```

`TestClient` habla directamente con la app FastAPI en memoria, sin necesitar
el servidor corriendo. Mismo resultado que curl, sin infraestructura.

Permite ejecutar `pytest` sin arrancar uvicorn — esencial para CI/CD donde
no hay nadie para arrancar nada manualmente.

---

## conftest.py — el escenario limpio

`conftest.py` es un archivo especial que pytest lee automáticamente.
Contiene fixtures — ingredientes reutilizables que los tests pueden pedir.

```python
@pytest.fixture(autouse=True)
def reset_mock_data():
    mock_data.RECIPES.clear()
    mock_data.RECIPES.update({
        1: mock_data.RECIPES_DEFAULT[1],
        2: mock_data.RECIPES_DEFAULT[2],
    })
    mock_data.next_recipe_id = 3
    mock_data.next_batch_id = 2
```

**`@pytest.fixture`** — convierte la función en un ingrediente inyectable.

**`autouse=True`** — se ejecuta automáticamente antes de cada test sin
que nadie lo pida. Sin esto, el estado se acumula entre tests y los
resultados dependen del orden de ejecución — tests frágiles e impredecibles.

**Por qué resetear**: si un test crea una receta con id=3, el siguiente
test que espera que solo haya 2 recetas va a fallar. El reset garantiza
que cada test parte del mismo estado limpio.

---

## Bug real encontrado y resuelto: estado compartido

### El problema
`test_create_recipe_increments_id` esperaba `id=3` pero recibió `id=4`.

### La causa
El módulo `recipes.py` importaba `next_recipe_id` directamente:
```python
from src.core.mock_data import next_recipe_id  # copia local del valor
```
Cuando el conftest reseteaba `mock_data.next_recipe_id`, el endpoint
seguía usando su copia local — ignoraba el reset.

### La solución
Acceder siempre a través del módulo:
```python
from src.core import mock_data
# ...
id=mock_data.next_recipe_id  # accede al valor actual, no a una copia
```

### La lección
Los imports directos de valores mutables crean copias locales que no
se actualizan cuando el original cambia. Para estado compartido que
necesita resetearse, siempre acceder a través del módulo.

---

## pre-commit — automatizar la calidad

`pre-commit` ejecuta verificaciones automáticamente antes de que el commit
se complete. Si algo falla, el commit se rechaza.

### Por qué existe
En un equipo nadie confía en la memoria de nadie. Se automatiza.
Antes: dependías de recordar Conventional Commits.
Ahora: Git lo verifica y rechaza cualquier mensaje incorrecto.

### Configuración (.pre-commit-config.yaml)

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace    # elimina espacios al final de línea
      - id: end-of-file-fixer      # garantiza salto de línea al final
      - id: check-yaml             # valida sintaxis YAML
      - id: check-json             # valida sintaxis JSON

  - repo: https://github.com/commitizen-tools/commitizen
    rev: v4.13.10
    hooks:
      - id: commitizen
        stages: [commit-msg]       # se ejecuta al escribir el mensaje
```

### Instalación (una vez por repo)

```bash
pre-commit install                          # hook pre-commit
pre-commit install --hook-type commit-msg  # hook commit-msg
```

Se instala en `.git/hooks/` — Git lo ejecuta automáticamente.

### Lo que ocurre en cada commit

```
git commit -m "feat: add new endpoint"
    │
    ├── trailing-whitespace......Passed
    ├── end-of-file-fixer........Passed
    ├── check-yaml...............Passed
    └── commitizen check.........Passed  ← formato correcto

[main xxxxx] feat: add new endpoint    ← commit registrado
```

### Commit rechazado — ejemplo real

```
git commit -m "arregle cosas"

commitizen check.........Failed
commit validation: failed!
pattern: (feat|fix|docs|style|refactor|test|chore|ci|build)...: descripción

← el commit NO se registra
```

### Dónde vive la configuración

- `.pre-commit-config.yaml` — raíz del repo (qué verificaciones ejecutar)
- `.cz.toml` — raíz del repo (formato de commits para commitizen)
- `.git/hooks/` — generado automáticamente por `pre-commit install`

### Importante: un pre-commit por repo

pre-commit actúa sobre un repositorio Git. Si tienes dos repos
(`brewery-app` y `portfolio`), necesitas instalarlo en cada uno por separado.
La configuración puede ser idéntica — la instalación es independiente.

---

## venv — por qué siempre

Python del sistema en Ubuntu 24.04 tiene PEP 668 activo — bloquea `pip install`
directo para proteger las herramientas del sistema operativo.

Cada proyecto necesita su propio entorno aislado para evitar conflictos de
versiones entre proyectos y entre el proyecto y el sistema.

Alternativa moderna a conocer: **`uv`** — gestor de entornos escrito en Rust,
extremadamente rápido, ganando terreno en 2025-2026. Lo exploraremos en Fase 4-5.

---

## Estructura final de tests

```
backend/
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # TestClient fixture + reset automático
│   ├── test_recipes.py      # 7 tests: list, get, get 404, create, validación
│   └── test_batches.py      # 7 tests: list, get, get 404, measurements, create
├── pytest.ini               # asyncio_mode=auto, testpaths=tests
└── requirements.txt         # pytest, httpx, pytest-asyncio añadidos
```

## Comando para ejecutar tests

```bash
cd ~/projects/brewery-app/backend
source venv/bin/activate
pytest -v                    # verbose — muestra cada test individualmente
pytest -v tests/test_recipes.py  # solo un archivo
pytest -v -k "test_get"     # solo tests cuyo nombre contiene "test_get"
```

---

## Deuda técnica cerrada esta sesión

| Item | Estado |
| --- | --- |
| pytest + httpx: primeros tests | ✅ Completado — 14 tests |
| pre-commit + commitizen | ✅ Completado — brewery-app y portfolio |

## Semana 3 completada ✅

| Item | Estado |
| --- | --- |
| Pydantic models: Recipe, Batch, FermentationSample | ✅ |
| API v1: 7 endpoints GET/POST | ✅ |
| Mock data con reset correcto | ✅ |
| 14 tests con conftest y fixtures | ✅ |
| pre-commit + commitizen en ambos repos | ✅ |
