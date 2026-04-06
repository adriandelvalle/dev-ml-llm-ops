# Plantilla: Actualizar README.md tras cada semana

> Usar al final de cada sesión de aprendizaje para mantener el portfolio actualizado.

---

## Checklist de actualización (Semana N)

### 1. Actualizar tabla "Learning Progress" en README.md

Ejemplo de cómo debe quedar en el archivo:

    ### Phase 1: Foundations 🔄 In Progress (Week N/4 completed)
    - [x] [Tema completado]
    - [ ] [Tema pendiente]

**Reglas**:
- Cambiar `Week X/4` por la semana actual.
- Marcar con `[x]` lo completado esta semana.
- Dejar con `[ ]` lo planeado para próximas semanas.

---

### 2. Añadir entrada en "Learning Notes"

Ejemplo:

    | 1 | N | [Tema de la semana] | [Ver](docs/learning/phase1-weekN-tema.md) |

**Ubicación**: Tabla bajo `### Learning Notes (Chronological)`.

---

### 3. Añadir cheatsheet nuevo (si creaste uno)

Ejemplo:

    | [Área] | [nombre-cheatsheet.md](docs/reference/nombre-cheatsheet.md) |

**Ubicación**: Tabla bajo `### Reference Cheatsheets`.

---

### 4. Actualizar "Featured Project" si hubo avance visible

Ejemplo:

    | 🟡 Fase 1 (scaffold) → 🟢 Fase 2 (API read-only) | FastAPI | Vehículo de aprendizaje |

**Actualizar también**: Lista de "Roadmap de features" (mover lo completado a arriba o tachar).

---

### 5. Actualizar fecha y filosofía al final

Ejemplo:

    > *Last updated: YYYY-MM-DD*
    > *Philosophy: Learning-first. 100% free stack. Depth > speed.*

---

## Commit message template

Copia y usa este mensaje:

    docs: update README with Phase X, Week N progress

    - Mark completed items in Learning Progress table
    - Add learning note link for week N
    - [Optional] Add new cheatsheet reference

---

## Recordatorio semanal

Al final de **cada sesión**, antes de cerrar:

- [ ] Código: commit + push en brewery-app/ (si hubo cambios)
- [ ] Aprendizaje: nota en docs/learning/phaseX-weekN.md
- [ ] Reference: cheatsheet nuevo en docs/reference/ (si aplica)
- [ ] Portfolio README: actualizar con esta plantilla
- [ ] Commit + push en portfolio/

> 💡 5 minutos al final de cada sesión = portfolio siempre actualizado para recruiters.
