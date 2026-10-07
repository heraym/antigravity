---
name: web-app-database
description: Skill especializado para la gestión de bases de datos PostgreSQL y AlloyDB para Google Cloud. Genera esquemas DDL SQL, scripts de migración Alembic, entonos de desarrollo local con Docker Compose y modelos ORM Python para el Backend.
---

# Skill de Gestión de Base de Datos (`web-app-database`)

Este skill define el protocolo de actuación para que el subagente de Base de Datos modele, migre y optimice la capa de persistencia relacional en **PostgreSQL**, garantizando compatibilidad 100% entre el entorno local (Docker) y producción (**Google Cloud AlloyDB para PostgreSQL**).

---

## Misión y Responsabilidades del Agente de Base de Datos

1. **Consumir el Modelo del Documento de Diseño**: Inspeccionar el diagrama Entidad-Relación (ER) y especificaciones en `docs/design/system-design.md`.
2. **Generar Esquema DDL SQL**: Crear scripts de creación de tablas, índices, restricciones y claves foráneas en formato ANSI SQL compatible con PostgreSQL / AlloyDB.
3. **Proporcionar Entorno Local (Docker Compose)**: Entregar archivo `docker-compose.yml` para ejecutar PostgreSQL 15+ localmente durante el desarrollo.
4. **Gestionar Migraciones (Alembic)**: Configurar **Alembic** para el control de versiones del esquema de base de datos.
5. **Asistir al Subagente Backend**: Proveer los modelos ORM de **SQLAlchemy / SQLModel** y la gestión de sesiones de conexión (`db/session.py`).
6. **Cumplir Buenas Prácticas de AlloyDB para Google Cloud**:
   - Conexión segura mediante **AlloyDB Auth Proxy** sin exponer direcciones IP públicas.
   - Configuración de autenticación mediante IAM.
   - Manejo eficiente de pools de conexión (SQLAlchemy `QueuePool` o PgBouncer).
   - Separación de lecturas/escrituras para aprovechar las réplicas de lectura de AlloyDB.
7. **Comentarios en Español**: Todo el código SQL, Python, YAML y documentación debe tener comentarios descriptivos en español.

---

## Flujo de Trabajo Paso a Paso

### Paso 1: Lectura del Documento de Diseño
1. Abrir `docs/design/system-design.md`.
2. Analizar el diagrama ER de Mermaid y la sección de entidades.
3. Identificar campos, tipos de datos PostgreSQL (UUID, VARCHAR, TIMESTAMP, JSONB), claves primarias e índices.

### Paso 2: Creación del Entorno Local (`docker-compose.yml`)
Configurar el entorno de desarrollo local con contenedor PostgreSQL:

```yaml
# Configuración del entorno de desarrollo local para PostgreSQL (compatible con AlloyDB)
version: '3.8'

services:
  # Servicio de Base de Datos PostgreSQL local
  postgres_local:
    image: postgres:15-alpine
    container_name: app_postgres_local
    environment:
      POSTGRES_USER: usuario_app       # Usuario de desarrollo
      POSTGRES_PASSWORD: password_local # Contraseña de desarrollo
      POSTGRES_DB: app_db               # Nombre de la base de datos
    ports:
      - "5432:5432"                    # Mapeo del puerto PostgreSQL
    volumes:
      - datos_pg:/var/lib/postgresql/data # Persistencia de datos local

volumes:
  datos_pg: # Volumen Docker para guardar los datos
```

### Paso 3: Generación del Esquema DDL SQL (`scripts/schema.sql`)
Crear las tablas garantizando estándares de diseño relacional (UUIDs como PK, timestamps UTC, índices en claves foráneas).

### Paso 4: Configuración de Migraciones con Alembic
Inicializar y configurar Alembic para automatizar la aplicación de cambios de esquema en entornos locales y en AlloyDB.

Ver [Patrones de ORM y Migraciones PostgreSQL](references/postgres-orm-patterns.md).

### Paso 5: Integración con Google Cloud AlloyDB
Configurar las cadenas de conexión para conectar con AlloyDB a través de **AlloyDB Auth Proxy**.

Ver [Buenas Prácticas de AlloyDB para Google Cloud](references/alloydb-gcp-best-practices.md).

---

## Recursos del Skill

- [Patrones de ORM y Migraciones PostgreSQL](references/postgres-orm-patterns.md)
- [Buenas Prácticas de AlloyDB para Google Cloud](references/alloydb-gcp-best-practices.md)
