# Fase 1, Semana 2: OpenCode + FastAPI Scaffold

## Objetivos
- [x] Instalar y configurar OpenCode CLI (v1.3.17)
- [x] Conectar OpenCode con Ollama local (modelo Qwen3:8b)
- [x] Generar scaffold inicial de FastAPI con IA
- [x] Implementar endpoint `/health` funcional
- [x] Configurar entorno virtual (`venv`) para aislamiento de dependencias
- [x] Probar endpoint localmente con `curl`
- [x] Documentar y hacer commit con Conventional Commits

## Tech Stack Utilizado
| Componente | Tecnología | Notas |
| :--- | :--- | :--- |
| **AI Coding** | OpenCode CLI + Ollama | Modelo: `qwen3:8b` (soporta tool calling) |
| **Backend** | FastAPI + Uvicorn | ASGI server, auto-reload activado |
| **Python** | 3.12.3 | Entorno virtual (`backend/venv`) obligatorio por PEP 668 |
| **Control** | Git + GitHub | Commits convencionales, branch `main` |

## Aprendizajes Clave
1. **PEP 668 en Ubuntu 24.04+**: `pip install` directo está bloqueado. Solución profesional: `python3 -m venv backend/venv && source backend/venv/bin/activate`.
2. **OpenCode + Ollama**: OpenCode detecta Ollama automáticamente si corre en `localhost:11434`. Para edición de archivos (tool calling), se recomienda `qwen3:8b` o superior; `qwen2.5-coder` funciona para análisis pero no para escritura automática.
3. **FastAPI Estructura**: Separación lógica `backend/src/main.py`. Uvicorn se ejecuta desde el directorio raíz del módulo o usando `uvicorn src.main:app`.
4. **Flujo de trabajo con IA**: Exploración automática del proyecto → Detección de estado vacío → Generación contextual → Validación manual con `curl`.

## Comandos Útiles
```bash
# Activar entorno y correr servidor
cd ~/projects/brewery-app
source backend/venv/bin/activate
uvicorn backend.src.main:app --reload --host 0.0.0.0 --port 8000

# Probar endpoint
curl http://localhost:8000/health
# Output: {"status":"healthy","timestamp":"..."}

# Salir del venv
deactivate
```