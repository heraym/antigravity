# Patrones de ORM y Migraciones PostgreSQL

Esta guía proporciona los patrones recomendados para diseñar esquemas SQL en PostgreSQL, definir modelos ORM en Python y gestionar migraciones de base de datos con **Alembic**.

---

## 1. Esquema DDL SQL Estándar (`scripts/schema.sql`)

Esquema de creación de tablas en PostgreSQL/AlloyDB con tipos de datos óptimos (`UUID`, `TIMESTAMP WITH TIME ZONE`, `JSONB`).

```sql
-- Habilitar extensión para generación de identificadores UUID
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Tabla de Usuarios del sistema
CREATE TABLE IF NOT EXISTS usuarios (
    -- Identificador único universal (UUID v4)
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    -- Correo electrónico único para autenticación
    email VARCHAR(255) UNIQUE NOT NULL,
    -- Hash seguro de la contraseña
    password_hash VARCHAR(255) NOT NULL,
    -- Nombre completo del usuario
    nombre VARCHAR(100) NOT NULL,
    -- Estado activo o inactivo del usuario
    activo BOOLEAN DEFAULT TRUE,
    -- Marca de tiempo de registro en formato UTC
    fecha_creacion TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Crear índice para optimizar búsquedas por email
CREATE INDEX IF NOT EXISTS idx_usuarios_email ON usuarios(email);

-- Tabla de Publicaciones o Recursos
CREATE TABLE IF NOT EXISTS publicaciones (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    -- Clave foránea referenciando al usuario creador
    usuario_id UUID NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
    titulo VARCHAR(200) NOT NULL,
    contenido TEXT NOT NULL,
    fecha_publicacion TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Crear índice en la clave foránea usuario_id
CREATE INDEX IF NOT EXISTS idx_publicaciones_usuario_id ON publicaciones(usuario_id);
```

---

## 2. Configuración de Sesiones ORM SQLAlchemy (`app/db/session.py`)

Proporciona un motor de conexión reutilizable y seguro, compatible tanto con PostgreSQL local como con AlloyDB.

```python
import os
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

# Construir URL de conexión desde variables de entorno
# Formato Local: postgresql://usuario:password@localhost:5432/app_db
# Formato AlloyDB (Proxy): postgresql://usuario:password@127.0.0.1:5432/app_db
DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql://usuario_app:password_local@localhost:5432/app_db"
)

# Configurar motor SQLAlchemy con pool de conexiones optimizado
engine = create_engine(
    DATABASE_URL,
    pool_size=10,            # Número de conexiones permanentes en el pool
    max_overflow=20,         # Conexiones adicionales temporales bajo alta carga
    pool_pre_ping=True,       # Verificar la salud de la conexión antes de usarla
    pool_recycle=1800        # Reciclar conexiones cada 30 minutos
)

# Crear fábrica de sesiones de base de datos
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Clase base declarativa para los modelos ORM
Base = declarative_base()

def obtener_db():
    """Generador de sesiones de base de datos para inyección de dependencias en FastAPI."""
    db = SessionLocal()
    try:
        yield db
    finally:
        # Garantizar el cierre de la sesión al finalizar la solicitud
        db.close()
```

---

## 3. Definición de Modelo ORM Python (`app/models/usuario.py`)

```python
import uuid
from datetime import datetime
from sqlalchemy import Column, String, Boolean, DateTime
from sqlalchemy.dialects.postgresql import UUID
from app.db.session import Base

class UsuarioModel(Base):
    """Modelo ORM representando la tabla de usuarios en PostgreSQL / AlloyDB."""
    __tablename__ = "usuarios"

    # Identificador único de tipo UUID v4
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    # Correo electrónico único registrado
    email = Column(String(255), unique=True, nullable=False, index=True)
    # Hash seguro bcrypt de la contraseña
    password_hash = Column(String(255), nullable=False)
    # Nombre del usuario
    nombre = Column(String(100), nullable=False)
    # Estado del usuario
    activo = Column(Boolean, default=True)
    # Fecha de creación registrada en UTC
    fecha_creacion = Column(DateTime(timezone=True), default=datetime.utcnow)
```
