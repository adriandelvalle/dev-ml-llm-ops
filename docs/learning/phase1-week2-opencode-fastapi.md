# Fase 1, Semana 2: OpenCode + FastAPI Scaffold

## Objetivos
- [x] Instalar OpenCode CLI vía script oficial
- [x] Configurar OpenCode con Ollama (qwen3:8b)
- [x] Generar scaffold de FastAPI con endpoint /health
- [x] Probar endpoint localmente con curl
- [x] Commit + push con mensajes convencionales

## Aprendizajes clave
- OpenCode detecta Ollama automáticamente si está corriendo en localhost:11434
- Usar venv es obligatorio en Ubuntu 24.04+ (PEP 668)
- Qwen3:8b soporta tool calling para edición de archivos; Qwen2.5 no
- FastAPI devuelve JSON automático con pydantic

## Comandos útiles
```bash
# Activar entorno
source backend/venv/bin/activate

# Ejecutar servidor
uvicorn backend.src.main:app --reload --host 0.0.0.0 --port 8000

# Probar endpoint
curl http://localhost:8000/health
