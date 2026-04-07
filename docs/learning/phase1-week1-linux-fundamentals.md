 Fase 1, Semana 1: Linux Fundamentos Aplicados

## Fecha
2026-04-06

## Objetivo
Linux (FHS, permisos, procesos) aplicado a un proyecto real, no en abstracto.

## Estructura creada
```
brewery-app/
├── backend/src/{api,core,models}/    # Código FastAPI futuro
├── docs/decisions/                   # ADRs del proyecto
└── scripts/                          # Herramientas de automatización
```

## Conceptos aprendidos

### FHS (Filesystem Hierarchy Standard)
| Ruta | Propósito | Decisión para brewery-app |
|------|-----------|--------------------------|
| `/home/jota/` | Home del usuario | ✅ Proyecto vive aquí (desarrollo) |
| `/home/jota/projects/` | Espacio de proyectos | ✅ Organización personal |
| `/opt/` o `/srv/` | Producción | ⚠️ Futuro: si desplegamos en producción |

### Permisos Linux
| Permiso | Octal | Uso en brewery-app |
|---------|-------|-------------------|
| `rwx------` | 700 | `.git/` (solo owner) |
| `rwxr-xr-x` | 755 | Carpetas del proyecto (públicas, no writable) |
| `rw-r--r--` | 644 | Archivos de código (.py, .md) |
| `rw-------` | 600 | Archivos sensibles (.env, secrets) |

### Hallazgo de la auditoría
El sistema crea carpetas con `775` por defecto (umask 002). Para seguridad, ajustamos a:
- `.git/` → `700` (más restrictivo, solo tú)
- Carpetas proyecto → `755` (otros pueden leer, no escribir)

## Script creado: `audit-permissions.sh`

**Propósito**: Verificar automáticamente que los permisos del proyecto cumplen el estándar definido.

**Uso**:
```bash
./scripts/audit-permissions.sh /ruta/al/proyecto
