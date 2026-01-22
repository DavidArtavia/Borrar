# Estructura de la Base de Datos

## Introducción

La estructura de la base de datos está diseñada con el propósito de maximizar la separación lógica de componentes y promover la modularidad. Esto se logra utilizando esquemas (`schemas`) y acomodando las diversas entidades y procesos en ubicaciones específicas según su funcionalidad. La separación descrita no solo facilita la mantenibilidad y escalabilidad del sistema, sino que también asegura una buena organización general.

## Características Clave

1. **Separación por Esquemas**: 
   - Los esquemas (`schemas`) son fundamentales para organizar las tablas, procedimientos almacenados (SP), funciones (`functions`) y otros objetos de la base de datos según su propósito funcional. Por ejemplo:
     - Las **tablas** relacionadas con usuarios y negocios están en un esquema `public` o `core`.
     - Los **procedimientos almacenados (SP)** para procesos de autenticación se encuentran en un esquema `auth_proc`.
     - Otros componentes específicos (como auditorías) son asignados a sus propios esquemas.

2. **Aislamiento de Objetos**: 
   - Cada componente de la base de datos (tablas, SP, índices, logs) se encapsula en su propio esquema según su funcionalidad. Esto:
     - Previene conflictos de nombres.
     - Aumenta la claridad y facilidad del mantenimiento.

3. **Buenas Prácticas**: 
   - El uso de convenciones de nombres claras asegura que cualquier desarrollador pueda interpretar fácilmente la responsabilidad de un objeto.
   - Los índices son creados cuidadosamente para mejorar la performance en búsquedas frecuentes.

4. **Ejemplo de Usos de Schemas**: 
   - Ejemplo del esquema `auth_proc`:
     ```sql
     CREATE OR REPLACE FUNCTION auth_proc.SP_AUTH_LOGIN(
       P_EMAIL       VARCHAR,
       P_PASSWORD    VARCHAR,
       P_IP          VARCHAR DEFAULT NULL,
       P_USER_AGENT  TEXT DEFAULT NULL
     )
     RETURNS TABLE (
       USER_ID UUID,
       EMAIL VARCHAR,
       ROLE VARCHAR,
       BUSINESS_ID UUID
     )
     LANGUAGE plpgsql
     AS $$
     -- Lógica de autenticación aquí
     $$;
     ```

     Nota: Al separar funciones relacionadas con autenticación en el esquema `auth_proc`, aseguramos un punto de acceso centralizado para tareas de autenticación.

## Ejemplo Práctico de "CREATE FUNCTION"

Un ejemplo más detallado de un SP organizando su lógica empresarial en esquemas bien definidos:

```sql
CREATE OR REPLACE FUNCTION auth_proc.SP_AUTH_REGISTER(
  P_BUSINESS_NAME   VARCHAR,
  P_BUSINESS_PHONE  VARCHAR,
  P_EMAIL           VARCHAR,
  P_PASSWORD        VARCHAR,
  P_IP              VARCHAR DEFAULT NULL,
  P_USER_AGENT      TEXT DEFAULT NULL
)
RETURNS TABLE (
  USER_ID UUID,
  EMAIL VARCHAR,
  ROLE VARCHAR,
  BUSINESS_ID UUID
)
LANGUAGE plpgsql
AS $$
DECLARE
  V_BUSINESS_ID UUID := gen_random_uuid();
  V_USER_ID UUID := gen_random_uuid();
  V_PASSWORD_HASH TEXT;
  V_ROLE VARCHAR := 'PYME';
BEGIN
  -- Verifica si ya existe el correo
  IF EXISTS (SELECT 1 FROM USERS U WHERE U.EMAIL = P_EMAIL) THEN
    RAISE EXCEPTION 'USER_ALREADY_EXISTS';
  END IF;

  -- Hasea la contraseña
  V_PASSWORD_HASH := crypt(P_PASSWORD, gen_salt('bf'));

  -- Inserta negocio
  INSERT INTO BUSINESSES (ID, NAME, PHONE)
  VALUES (V_BUSINESS_ID, P_BUSINESS_NAME, P_BUSINESS_PHONE);

  -- Inserta usuario
  INSERT INTO USERS (ID, EMAIL, PASSWORD_HASH, ROLE, BUSINESS_ID)
  VALUES (V_USER_ID, P_EMAIL, V_PASSWORD_HASH, V_ROLE, V_BUSINESS_ID);

  -- Audita
  INSERT INTO AUTH_AUDIT_LOGS (USER_ID, BUSINESS_ID, ACTION, IP_ADDRESS, USER_AGENT)
  VALUES (V_USER_ID, V_BUSINESS_ID, 'REGISTER_SUCCESS', P_IP, P_USER_AGENT);

  -- Retorna datos alterados
  RETURN QUERY
  SELECT
    V_USER_ID::UUID,
    P_EMAIL::VARCHAR,
    V_ROLE::VARCHAR,
    V_BUSINESS_ID::UUID;

END;
$$;
```

## Importancia de los Esquemas

1. **Modularidad**: Permite que múltiples equipos puedan desarrollar objetos de la base de datos sin conflictos, siempre que trabajen en esquemas separados.
2. **Seguridad**: Mediante la asignación de permisos por esquema, se puede gestionar con mayor granularidad qué usuarios tienen acceso a qué objetos.
3. **Documentación y Mantenibilidad**: Una estructura más organizada permite documentar y mantener el sistema de manera más eficiente.
4. **Estandarización**: Fomenta la uniformidad en grandes proyectos.

## Cómo Implementar la Separación en Proyectos Existentes

1. **Crear esquemas**:
   ```sql
   CREATE SCHEMA auth_proc;
   CREATE SCHEMA core;
   ```

2. **Asignar permisos por esquema**:
   ```sql
   GRANT USAGE ON SCHEMA auth_proc TO dev_team;
   REVOKE USAGE ON SCHEMA core FROM test_team;
   ```

3. **Migrar objetos existentes**:
   - Por ejemplo, mover todas las funciones de autenticación al esquema `auth_proc`:
     ```sql
     ALTER FUNCTION SP_AUTH_LOGIN SET SCHEMA auth_proc;
     ALTER FUNCTION SP_AUTH_REGISTER SET SCHEMA auth_proc;
     ```

4. **Configurar convenciones claras** para equipo de desarrollo:
   - Tablas principales en `core`.
   - Lógica de negocio en esquema específico.

## Conclusión

La separación a nivel de estructura de la base de datos, utilizando esquemas, mejora la calidad del desarrollo al ofrecer una clara delineación de responsabilidades y un sistema más seguro, escalable y mantenible.

Con este enfoque, se crea un flujo óptimo que aborda desde la lógica empresarial hasta la implementación técnica efectiva.

# Estructura de la Base de Datos

## Introducción

La base de datos sigue un diseño bien estructurado que prioriza la separación lógica y organizativa mediante el uso de esquemas (`schemas`) y una jerarquía de archivos clara. La estructura propuesta no solo mejora la mantenibilidad y la escalabilidad, sino que también estandariza el desarrollo entre equipos.

A continuación, se detalla la organización de carpetas y la explicación de cada archivo junto con la importancia de los esquemas.

---

## Organización de Carpetas y Archivos

La estructura de carpetas sigue un orden lógico para favorecer la mantenibilidad, como se ilustra a continuación:

```
DTB/
└── Producto Base/
    ├── 01-Create.sql
    ├── 02-Schemas.sql
    ├── 03-Tables/
    │   ├── USERS.sql
    │   ├── BUSINESSES.sql
    │   └── AUTH_AUDIT_LOGS.sql
    ├── 04-Procedures/
    │   ├── SP_AUTH_LOGIN.sql
    │   └── SP_AUTH_REGISTER.sql
    └── 05-Indexes/
        ├── IDX_USERS_EMAIL.sql
        ├── IDX_USERS_BUSINESS_ID.sql
        └── IDX_AUDIT_LOGS.sql
```

### Descripción de Archivos

1. **`01-Create.sql`**:
   - Archivo principal que ejecuta la creación de la base de datos.
   - Incluye instrucciones como:
     ```sql
     CREATE DATABASE NAME_DB;
     ```

2. **`02-Schemas.sql`**:
   - Define los esquemas de la base de datos para organizar componentes.
   - Por ejemplo:
     ```sql
     CREATE SCHEMA public;
     CREATE SCHEMA auth_proc;
     CREATE SCHEMA audit;
     ```

3. **`03-Tables/`**:
   - Carpeta que contiene las definiciones de las tablas principales.
   - Archivos destacados:
     - **`USERS.sql`**:
       ```sql
       CREATE TABLE USERS (
         ID UUID PRIMARY KEY,
         EMAIL VARCHAR(150) NOT NULL UNIQUE,
         PASSWORD_HASH TEXT NOT NULL,
         ROLE VARCHAR(20) NOT NULL CHECK (ROLE IN ('ADMIN', 'PYME')),
         BUSINESS_ID UUID NOT NULL,
         IS_ACTIVE BOOLEAN NOT NULL DEFAULT TRUE,
         CREATED_AT TIMESTAMP NOT NULL DEFAULT NOW(),
         CONSTRAINT fk_users_business FOREIGN KEY (BUSINESS_ID)
           REFERENCES BUSINESSES(ID)
           ON DELETE RESTRICT
       );
       ```
     - **`BUSINESSES.sql`**:
       ```sql
       CREATE TABLE BUSINESSES (
         ID UUID PRIMARY KEY,
         NAME VARCHAR(150) NOT NULL,
         PHONE VARCHAR(30) NOT NULL,
         IS_ACTIVE BOOLEAN NOT NULL DEFAULT TRUE,
         CREATED_AT TIMESTAMP NOT NULL DEFAULT NOW()
       );
       ```
     - **`AUTH_AUDIT_LOGS.sql`**:
       ```sql
       CREATE TABLE AUTH_AUDIT_LOGS (
         ID BIGSERIAL PRIMARY KEY,
         USER_ID UUID,
         BUSINESS_ID UUID,
         ACTION VARCHAR(50) NOT NULL,
         IP_ADDRESS VARCHAR(45),
         USER_AGENT TEXT,
         CREATED_AT TIMESTAMP NOT NULL DEFAULT NOW()
       );
       ```

4. **`04-Procedures/`**:
   - Carpeta que organiza los procedimientos almacenados (SP).
   - Archivos destacados:
     - **`SP_AUTH_LOGIN.sql`**:
       Implementa el proceso de login.
       ```sql
       CREATE OR REPLACE FUNCTION auth_proc.SP_AUTH_LOGIN(
         P_EMAIL       VARCHAR,
         P_PASSWORD    VARCHAR,
         P_IP          VARCHAR DEFAULT NULL,
         P_USER_AGENT  TEXT DEFAULT NULL
       )
       RETURNS TABLE (
         USER_ID UUID,
         EMAIL VARCHAR,
         ROLE VARCHAR,
         BUSINESS_ID UUID
       )
       LANGUAGE plpgsql
       AS $$
       BEGIN
         -- Lógica del login
       END;
       $$;
       ```
     - **`SP_AUTH_REGISTER.sql`**:
       Implementa el registro de nuevos usuarios y negocios.
       ```sql
       CREATE OR REPLACE FUNCTION auth_proc.SP_AUTH_REGISTER(
         P_BUSINESS_NAME   VARCHAR,
         P_BUSINESS_PHONE  VARCHAR,
         P_EMAIL           VARCHAR,
         P_PASSWORD        VARCHAR
       )
       RETURNS TABLE (
         USER_ID UUID,
         EMAIL VARCHAR,
         ROLE VARCHAR,
         BUSINESS_ID UUID
       )
       LANGUAGE plpgsql
       AS $$
       BEGIN
         -- Lógica del registro
       END;
       $$;
       ```

5. **`05-Indexes/`**:
   - Contiene los índices definidos para optimizar el acceso a datos.
   - Archivos destacados:
     - **`IDX_USERS_EMAIL.sql`**:
       ```sql
       CREATE INDEX IDX_USERS_EMAIL ON USERS(EMAIL);
       ```
     - **`IDX_USERS_BUSINESS_ID.sql`**:
       ```sql
       CREATE INDEX IDX_USERS_BUSINESS_ID ON USERS(BUSINESS_ID);
       ```
     - **`IDX_AUDIT_LOGS.sql`**:
       ```sql
       CREATE INDEX IDX_AUDIT_USER_ID ON AUTH_AUDIT_LOGS(USER_ID);
       ```

---

## Uso de los Schemas y su Importancia

### ¿Qué son los Schemas?

Los esquemas permiten dividir y categorizar los elementos de una base de datos como tablas, índices y funciones, mejorando la modularidad y la seguridad del sistema. Cada esquema puede entenderse como un "espacio de nombres" para evitar la colisión de objetos y establecer límites claros en cuanto a permisos.

### Beneficios de Usar Schemas

1. **Organización**: Facilitan la separación lógica de los objetos según su funcionalidad.
2. **Mantenibilidad**: Permiten a distintos equipos trabajar en secciones específicas sin conflictos.
3. **Seguridad**: Posibilitan la asignación de permisos granular, como restringir el acceso a funciones sensibles.
4. **Escalabilidad**: Aseguran una estructura que se adapta al crecimiento del sistema.

### Ejemplo de Separación por Schemas

```sql
CREATE SCHEMA auth_proc; -- Esquema para lógica de autenticación
CREATE SCHEMA audit;     -- Esquema para registros de auditoría
```

Luego, cada objeto se asigna explícitamente al esquema correspondiente:

```sql
-- Crear tabla en esquema audit
CREATE TABLE audit.AUTH_AUDIT_LOGS (
  ID BIGSERIAL PRIMARY KEY,
  USER_ID UUID,
  ACTION VARCHAR(50) NOT NULL,
  TIMESTAMP TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Crear función en esquema auth_proc
CREATE OR REPLACE FUNCTION auth_proc.SP_AUTH_LOGIN() RETURNS VOID AS $$
BEGIN
  -- Lógica
END;
$$;
```

---

## Conclusión

Esta estructura modular basada en esquemas y carpetas mejora significativamente la organización y la capacidad de mantenimiento del proyecto. Al seguir estándares claros, garantizamos un sistema eficiente, seguro y escalable.
