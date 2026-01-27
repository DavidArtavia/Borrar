# Clase de Bases de Datos (60 minutos): ER, Relaciones y Normalización

## Objetivos de aprendizaje
Al finalizar la sesión, el estudiante será capaz de:
- Diferenciar entidades, atributos y relaciones (1:1, 1:N, N:M) en un modelo ER.
- Diseñar un ERD sencillo con cardinalidades y opcionalidad.
- Mapear un ERD a esquema relacional (tablas, PK, FK).
- Aplicar 1FN, 2FN y 3FN y explicar cuándo desnormalizar.
- Escribir DDL y consultas JOIN básicas en SQL.

## Prerrequisitos
- Conocer tipos de datos básicos (texto, número, fecha).
- Familiaridad mínima con SQL (SELECT).

## Materiales
- Pizarra o slides.
- Motor SQL (SQLite, PostgreSQL o MySQL).
- Archivo de ejercicios SQL (incluido).
- Caso práctico (incluido).

## Agenda (60 min)
- 00:00–05: Introducción y objetivos
- 05:00–20: Modelo Entidad–Relación (ER) y cardinalidades
- 20:00–35: Normalización (1FN, 2FN, 3FN, BCNF en breve)
- 35:00–50: Del ER al esquema relacional + SQL DDL/JOINS
- 50:00–57: Actividad guiada (caso práctico)
- 57:00–60: Evaluación rápida y cierre

---

## 1) Conceptos base (para contextualizar)
- Base de datos vs. DBMS (PostgreSQL, MySQL, SQLite).
- Tabla, fila (tupla/registro), columna (atributo).
- Tipos de datos: INT, VARCHAR, DATE/TIMESTAMP, BOOLEAN, DECIMAL.
- Claves:
  - Clave primaria (PK): identifica unívocamente.
  - Clave candidata: posibles PK.
  - Clave foránea (FK): referencia a PK de otra tabla.
  - Clave compuesta: PK con varias columnas.
  - Natural vs. surrogate (autonumérica).
- Restricciones: NOT NULL, UNIQUE, CHECK, DEFAULT.
- Integridad referencial: ON DELETE/UPDATE (RESTRICT/NO ACTION/CASCADE/SET NULL).

---

## 2) Modelo Entidad–Relación (ER)
- Entidad: objeto del dominio (Estudiante, Curso).
- Atributos: simples (nombre), compuestos (dirección), multivaluados (teléfonos).
- Relaciones:
  - 1:1 (una persona ↔ un pasaporte)
  - 1:N (un cliente ↔ muchas órdenes)
  - N:M (muchos estudiantes ↔ muchos cursos)
- Cardinalidad y opcionalidad: 0..1, 1..1, 0..N, 1..N
- Notación (patas de cuervo) y lectura de reglas de negocio.

Ejemplo ER (texto):
- Entidades: Estudiante(id_estudiante, nombre, email), Curso(id_curso, titulo, creditos)
- Relación N:M “Matriculado_en” con atributos (fecha_matricula)
  - Se modela con entidad/tabla intermedia: Matricula(id_estudiante, id_curso, fecha_matricula)

Diagrama ASCII:
```
Estudiante (PK id_estudiante)
    1 ─< Matricula >─ N
                     ^
                     | N
Curso (PK id_curso)
```
- PK(Matricula) = (id_estudiante, id_curso)
- FKs: id_estudiante → Estudiante.id_estudiante, id_curso → Curso.id_curso

---

## 3) Normalización (por qué y cómo)
Problemas sin normalizar: redundancia, anomalías de inserción/actualización/borrado.

- 1FN: valores atómicos, sin grupos repetidos.
  - Anti-ejemplo: Estudiante(tel1, tel2, tel3). Solución: tabla EstudianteTelefono(estudiante_id, telefono).
- 2FN: sin dependencias parciales en PK compuesta.
  - Anti-ejemplo: DetallePedido(pedido_id, producto_id, nombre_producto, cantidad) donde nombre_producto depende solo de producto_id.
  - Solución: mover datos de producto a Producto y dejar en detalle solo lo que depende de toda la PK.
- 3FN: sin dependencias transitivas (A → B → C).
  - Anti-ejemplo: Empleado(id, depto_id, depto_nombre). depto_nombre depende de depto_id, no de empleado.id.
  - Solución: separar Departamento.
- BCNF (breve): todo determinante es clave candidata (útil mencionarlo, no profundizar por tiempo).

Cuándo desnormalizar: para rendimiento (lecturas frecuentes), con cuidado y midiendo impacto.

---

## 4) Del ER al esquema relacional + SQL
Reglas de mapeo:
- Entidad → Tabla (atributos simples → columnas).
- Relación 1:N → FK en el lado N.
- Relación N:M → tabla intermedia con FKs y PK compuesta.
- Atributos multivaluados → tabla aparte.
- Entidad débil → PK que incluye FK al identificador de la entidad fuerte.

Ejemplo SQL (DDL abreviado):
```sql
CREATE TABLE Estudiante (
  id_estudiante SERIAL PRIMARY KEY,
  nombre        VARCHAR(100) NOT NULL,
  email         VARCHAR(120) UNIQUE NOT NULL
);

CREATE TABLE Curso (
  id_curso  SERIAL PRIMARY KEY,
  titulo    VARCHAR(150) NOT NULL,
  creditos  INT CHECK (creditos BETWEEN 1 AND 12)
);

CREATE TABLE Matricula (
  id_estudiante INT NOT NULL,
  id_curso      INT NOT NULL,
  fecha_matricula DATE NOT NULL DEFAULT CURRENT_DATE,
  PRIMARY KEY (id_estudiante, id_curso),
  FOREIGN KEY (id_estudiante) REFERENCES Estudiante(id_estudiante) ON DELETE CASCADE,
  FOREIGN KEY (id_curso)      REFERENCES Curso(id_curso) ON DELETE RESTRICT
);
```

Consultas JOIN típicas:
```sql
-- Estudiantes y los cursos en los que están
SELECT e.nombre, c.titulo, m.fecha_matricula
FROM Matricula m
JOIN Estudiante e ON e.id_estudiante = m.id_estudiante
JOIN Curso c      ON c.id_curso      = m.id_curso;

-- Número de estudiantes por curso
SELECT c.titulo, COUNT(*) AS inscritos
FROM Curso c
LEFT JOIN Matricula m ON m.id_curso = c.id_curso
GROUP BY c.titulo
ORDER BY inscritos DESC;
```

Índices recomendados:
- FKs y columnas de JOINs frecuentes.
- Columnas de búsqueda (email).

---

## 5) Actividad guiada (7 minutos en clase)
Usa el “Caso_Practico.md”.
- Paso 1 (3 min): Identificar entidades, atributos y relaciones + cardinalidades.
- Paso 2 (2 min): Proponer tablas, PKs y FKs.
- Paso 3 (2 min): Detectar si hay que normalizar (1FN, 2FN, 3FN).

Si hay tiempo, implementar tablas base y una relación N:M.

---

## 6) Evaluación rápida (3 minutos)
1) Diferencia entre PK y FK con un ejemplo.
2) ¿Cómo modelas una relación N:M?
3) Da un ejemplo de 1FN violada y cómo corregir.
4) ¿Qué problema evita 3FN?
5) ¿Cuándo considerarías desnormalizar?

Rubrica breve:
- 5/5: Respuestas precisas con ejemplo.
- 3–4/5: Conceptos correctos, ejemplos mejorables.
- ≤2/5: Confusiones en PK/FK o en niveles de normalización.

---

## Tarea (opcional)
- Completar el caso práctico: ERD → esquema relacional → SQL DDL.
- Insertar datos de prueba (≥5 registros por tabla).
- Escribir 3 consultas JOIN y 1 agregación.
- Entregar ERD (imagen) y script SQL.

## Recursos
- “Database System Concepts” – Silberschatz, Korth, Sudarshan.
- “Fundamentals of Database Systems” – Elmasri & Navathe.
- PostgreSQL Docs: https://www.postgresql.org/docs/
- Draw.io / dbdiagram.io para ERDs.
