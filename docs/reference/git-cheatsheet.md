# Git Cheatsheet & Convenciones

> Documentación de referencia para comandos Git y convenciones de equipo.
> Actualizado: 2026-04-06 | Proyecto: brewery-app + portfolio

---

## Flujo básico de trabajo

```bash
# Ver estado actual
git status

# Añadir archivos al staging
git add <archivo>           # Uno específico
git add .                   # Todos los cambios
git add -p                  # Interactivo: revisar cambio por cambio

# Ver diferencias antes de commitear
git diff                    # Cambios no staged
git diff --staged          # Cambios ya en staging

# Hacer commit
git commit -m "tipo: descripción breve"

# Ver historial
git log --oneline          # Resumen compacto
git log -p -1              # Detalles del último commit

```

## Convencional Commits: Tipos y cuándo usarlos

| Prefijo | Cuándo usarlo | Ejemplo |
|---------|--------------|---------|
| `feat:` | Nueva funcionalidad visible para el usuario | `feat: add /health endpoint` |
| `fix:` | Corrección de bug | `fix: correct permission check in audit script` |
| `docs:` | Solo documentación (README, comments, docs/) | `docs: add Git cheatsheet reference` |
| `style:` | Formato, sin cambios de lógica (espacios, lint) | `style: format Python files with black` |
| `refactor:` | Cambios de código que no añaden ni arreglan features | `refactor: extract permission check to function` |
| `perf:` | Mejoras de rendimiento | `perf: cache model loading in inference endpoint` |
| `test:` | Añadir o corregir tests | `test: add unit test for audit-permissions.sh` |
| `chore:` | Mantenimiento, configs, herramientas (sin impacto en usuario) | `chore: add .gitignore for Python artifacts` |
| `ci:` | Cambios en CI/CD (GitHub Actions, etc.) | `ci: add lint workflow on push` |
| `build:` | Cambios en build system o dependencias | `build: upgrade FastAPI to 0.110.0` |

### ¿Por qué usar Convencional Commits?

1.  **Legibilidad**: El historial se lee como un changelog automático.
2.  **Automatización**: Herramientas como `semantic-release` generan versiones y changelogs solos.
3.  **Búsqueda**: `git log --grep="^feat:"` para ver solo nuevas features.
4.  **Profesionalismo**: Estándar adoptado por equipos senior y empresas.

>  **Regla**: El mensaje después del prefijo debe ser **imperativo** ("add", no "added") y en **inglés** para consistencia internacional.

---


## Ramas (Branches)

```bash
# Crear y cambiar a nueva rama
git checkout -b feature/nombre-feature

# Listar ramas
git branch -a              # Todas (locales + remotas)
git branch -vv             # Con información de tracking

# Fusionar ramas
git merge feature/nombre-feature   # Desde la rama destino

# Eliminar rama
git branch -d feature/nombre-feature   # Local (solo si está mergeada)
git branch -D feature/nombre-feature   # Forzar eliminación
git push origin --delete feature/nombre-feature  # Remota

```


### Convención de nombres de ramas
| Tipo | Formato | Ejemplo |
|------|---------|---------|
| Feature | `feature/descripcion-breve` | `feature/add-inventory-endpoint` |
| Fix | `fix/descripcion-breve` | `fix/correct-permission-audit` |
| Hotfix | `hotfix/issue-critico` | `hotfix/fix-health-check-timeout` |
| Experimento | `experiment/idea` | `experiment/try-ollama-14b-model` |

---

## Sincronización con remoto (GitHub)

```bash
# Añadir remote (primera vez)
git remote add origin git@github.com:usuario/repo.git

# Ver remotes configurados
git remote -v

# Subir cambios (push)
git push                    # Si ya hay upstream configurado
git push -u origin main    # Primera vez: establece tracking

# Bajar cambios (pull)
git pull                   # Fetch + merge automático

# Ver estado de sincronización
git status                 # Muestra si estás ahead/behind del remote

```

## Troubleshooting común

| Problema | Solución |
|----------|----------|
| "Changes not staged for commit" | Usa `git add <archivo>` antes de commitear |
| "Nothing to commit, working tree clean" | No hay cambios nuevos, todo está commiteado |
| "Permission denied (publickey)" al push | Revisa claves SSH: `ssh -T git@github.com` |
| Quieres cambiar el último commit | `git commit --amend -m "nuevo mensaje"` (solo si no hiciste push) |
| Quieres deshacer cambios no commiteados | `git restore <archivo>` o `git checkout -- <archivo>` |
| Quieres ver qué cambió en un commit | `git show <hash>` o `git log -p -1` |

---


## Criterios de "commit listo"

Antes de hacer `git commit`, preguntate:

- [ ] ¿Los cambios tienen un **proposito unico y claro**? (no mezclar features)
- [ ] ¿El mensaje sigue **Convencional Commits** (`tipo: descripcion`)?
- [ ] ¿La descripcion esta en **imperativo + ingles**? ("add", no "added")
- [ ] ¿Probe que el codigo **funciona** antes de commitear?
- [ ] ¿Anadi/actualice **tests** si corresponde?
- [ ] ¿Actualice la **documentacion** si cambio el comportamiento?

> **Regla de oro**: *"Un commit debe ser como un capitulo de libro: autocontenido, con proposito claro, y que pueda leerse independientemente."*

---

## Enlaces utiles

- [Conventional Commits Spec](https://www.conventionalcommits.org/)
- [Git Cheat Sheet (GitHub)](https://education.github.com/git-cheat-sheet-education.pdf)
- [Oh Shit, Git!?!](https://ohshitgit.com/) - Para cuando algo sale mal

---



