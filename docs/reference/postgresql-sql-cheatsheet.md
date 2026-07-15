# PostgreSQL & SQL Cheatsheet

Quick reference for SQL fundamentals and PostgreSQL-specific features.
Last updated: 2026-07-15

---

## Acceso con psql

```bash
# Alias configurado en ~/.bashrc
psql-brew

# Comando completo (si el alias no está disponible)
docker exec -it brewery-db psql -U brewery -d brewery_db
```

### Comandos internos de psql (no son SQL)

```sql
\dt                    -- listar todas las tablas
\d nombre_tabla        -- describir estructura de una tabla
\dn                    -- listar schemas
\du                    -- listar usuarios/roles
\q                     -- salir
\?                     -- ayuda de comandos psql
SHOW timezone;         -- ver timezone activo (siempre UTC en este proyecto)
```

---

## Tipos de datos más usados

| Tipo SQL | Equivalente Python | Cuándo usarlo |
| --- | --- | --- |
| `SERIAL` | auto int | IDs autoincrementados |
| `INTEGER` | int | Números enteros |
| `NUMERIC(p,s)` | Decimal | Medidas de precisión (OG, ABV, volumen) |
| `VARCHAR(n)` | str con límite | Textos cortos con longitud conocida |
| `TEXT` | str | Textos largos sin límite |
| `BOOLEAN` | bool | Valores true/false |
| `DATE` | date | Solo fecha, sin hora |
| `TIMESTAMP` | datetime | Fecha + hora (siempre en UTC) |

> **Nunca usar `FLOAT`** para medidas de precisión — tiene errores de redondeo
> binario. Usar `NUMERIC(precision, escala)`.

---

## CREATE TABLE

```sql
CREATE TABLE recipes (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    style VARCHAR(20) NOT NULL,
    batch_size_liters NUMERIC(5,2) NOT NULL,
    target_og NUMERIC(5,3) NOT NULL,
    target_fg NUMERIC(5,3) NOT NULL,
    target_ibu INTEGER,              -- nullable por defecto
    target_abv NUMERIC(4,2),         -- nullable por defecto
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Con foreign key

```sql
CREATE TABLE batches (
    id SERIAL PRIMARY KEY,
    recipe_id INTEGER NOT NULL REFERENCES recipes(id),  -- foreign key
    brew_date DATE NOT NULL,
    brewer VARCHAR(50) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'planned',
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## INSERT

```sql
-- Sin id ni created_at — los genera PostgreSQL automáticamente
INSERT INTO recipes (name, style, batch_size_liters, target_og, target_fg)
VALUES ('Asturian Pale Ale', 'APA', 50.00, 1.052, 1.010);

-- Recuperar el id generado inmediatamente
INSERT INTO recipes (name, style, batch_size_liters, target_og, target_fg)
VALUES ('Gijon Stout', 'STOUT', 50.00, 1.060, 1.014)
RETURNING id;
```

---

## SELECT

```sql
-- Todas las columnas
SELECT * FROM recipes;

-- Columnas específicas
SELECT id, name, style FROM recipes;

-- Con condición
SELECT * FROM recipes WHERE style = 'APA';

-- Múltiples condiciones
SELECT * FROM batches WHERE status = 'fermenting' AND brewer = 'jota';

-- Ordenado
SELECT * FROM recipes ORDER BY created_at DESC;

-- Limitado
SELECT * FROM recipes LIMIT 10;
```

---

## UPDATE

```sql
-- Siempre con WHERE — sin él actualiza TODAS las filas
UPDATE batches SET status = 'brewing' WHERE id = 2;

-- Múltiples columnas
UPDATE batches SET status = 'fermenting', notes = 'fermentación activa' WHERE id = 2;
```

> `UPDATE N` en el output indica cuántas filas se modificaron.
> Si ves un número inesperadamente alto, hay un problema con el WHERE.

---

## DELETE

```sql
-- Siempre con WHERE — sin él borra TODAS las filas
DELETE FROM batches WHERE id = 2;

-- Borrar múltiples filas con condición
DELETE FROM batches WHERE recipe_id = 1;
DELETE FROM batches WHERE brew_date < '2026-01-01';
DELETE FROM batches WHERE status = 'planned';
```

---

## Foreign keys — integridad referencial

### Comportamiento por defecto: RESTRICT

```sql
-- Intento de insertar batch con recipe_id inexistente → ERROR
INSERT INTO batches (recipe_id, ...) VALUES (999, ...);
-- ERROR: insert or update on table "batches" violates foreign key constraint
-- DETAIL: Key (recipe_id)=(999) is not present in table "recipes".

-- Intento de borrar receta con batches apuntando a ella → ERROR
DELETE FROM recipes WHERE id = 1;
-- ERROR: update or delete on table "recipes" violates foreign key constraint
-- DETAIL: Key (id)=(1) is still referenced from table "batches".
```

### Opciones de comportamiento al borrar

| Opción | Comportamiento | Cuándo usar |
| --- | --- | --- |
| `RESTRICT` (default) | Bloquea el borrado | Datos de negocio con historial importante |
| `CASCADE` | Borra en cascada | Peligroso — usar con mucho cuidado |
| `SET NULL` | Pone NULL en la FK | Cuando el historial puede quedar huérfano |

---

## Secuencias — comportamiento importante

Las secuencias (`SERIAL`) **nunca retroceden**, incluso si el INSERT falla.
Un INSERT fallido consume el número de la secuencia — la siguiente inserción
exitosa recibe el número siguiente, no el fallido. Resultado: los IDs pueden
tener huecos (1, 3, 4... sin el 2).

**Nunca asumir que los IDs son consecutivos sin huecos.**
**Nunca calcular el próximo ID manualmente — usar `RETURNING id` o el ORM.**

---

## UTC vs hora local

PostgreSQL opera en UTC por defecto:

```sql
SHOW timezone;  -- Etc/UTC
SELECT NOW();   -- 2026-07-15 12:00:00+00
```

Si el servidor está en CEST (UTC+2), `created_at` mostrará 2 horas menos
que la hora local — comportamiento correcto y deliberado.

Regla: **guardar siempre en UTC, convertir a hora local solo en la presentación**.

---

## Soft delete — patrón empresarial

En lugar de `DELETE` real, marcar como inactivo:

```sql
ALTER TABLE recipes ADD COLUMN deleted_at TIMESTAMP;
ALTER TABLE recipes ADD COLUMN is_active BOOLEAN DEFAULT TRUE;

-- "Borrar" una receta
UPDATE recipes SET deleted_at = NOW(), is_active = FALSE WHERE id = 1;

-- Consultar solo activos
SELECT * FROM recipes WHERE is_active = TRUE;
```

Ventaja: historial preservado, recuperación posible, sin romper foreign keys.

---

## Alembic version

```sql
-- Ver qué migración está aplicada actualmente
SELECT * FROM alembic_version;
-- version_num: hash de la última migración aplicada
```

---

## Troubleshooting

| Error | Causa | Solución |
| --- | --- | --- |
| `violates foreign key constraint` al INSERT | FK apunta a fila inexistente | Verificar que el id referenciado existe |
| `violates foreign key constraint` al DELETE | Hay filas apuntando a la que quieres borrar | Borrar primero las filas dependientes |
| ID inesperado (huecos en la secuencia) | INSERTs fallidos consumieron números | Normal, nunca asumir consecutividad |
| `created_at` 2 horas menos que la hora local | PostgreSQL en UTC, servidor en CEST | Comportamiento correcto |
