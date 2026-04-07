# FastAPI + OpenCode + Ollama Cheatsheet

Quick reference for local AI-assisted Python backend development.

## Python Environment
| Action | Command |
| :--- | :--- |
| Create venv | `python3 -m venv backend/venv` |
| Activate | `source backend/venv/bin/activate` |
| Deactivate | `deactivate` |
| Install deps | `pip install -r backend/requirements.txt` |
| Freeze deps | `pip freeze > backend/requirements.txt` |

## FastAPI & Uvicorn
| Action | Command |
| :--- | :--- |
| Run dev server | `uvicorn backend.src.main:app --reload --host 0.0.0.0 --port 8000` |
| Health check | `curl http://localhost:8000/health` |
| Swagger UI | `http://localhost:8000/docs` |
| ReDoc | `http://localhost:8000/redoc` |

## Port Forwarding & Acceso Remoto
| Escenario | Solución |
| :--- | :--- |
| VS Code cerrado | Reabrir VS Code → pestaña PORTS → restaurar 8000 |
| Sin VS Code | `ssh -L 8000:localhost:8000 jota@<server-ip>` |
| Acceso directo (red segura) | `http://<server-ip>:8000/docs` |
| API caída tras reinicio | Usar `systemd` o Docker `restart: always` (Semana 4) |

## OpenCode + Ollama
| Task | Prompt / Command |
| :--- | :--- |
| Start AI | `opencode` |
| Create endpoint | `"Create a GET /items endpoint with Pydantic validation"` |
| Refactor | `"Refactor this function to use async/await and add error handling"` |
| Add tests | `"Generate pytest unit tests for this endpoint"` |

