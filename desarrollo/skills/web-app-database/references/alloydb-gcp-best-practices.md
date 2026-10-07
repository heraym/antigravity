# Mejores Prácticas de Google Cloud para AlloyDB para PostgreSQL

**Google Cloud AlloyDB para PostgreSQL** es un servicio de base de datos relacional de nivel empresarial totalmente administrado, 100% compatible con PostgreSQL.

---

## 1. Conexión Segura mediante AlloyDB Auth Proxy

La forma recomendada de conectar aplicaciones alojadas en Google Cloud Run a AlloyDB es mediante el cliente **AlloyDB Auth Proxy** o el conector **AlloyDB Python Connector**.

### Ventajas del AlloyDB Auth Proxy
- **Cifrado mTLS Automático**: Cifra todo el tráfico de red sin necesidad de administrar certificados SSL/TLS manualmente.
- **Autenticación Basada en IAM**: Utiliza la identidad de la cuenta de servicio (Service Account) del contenedor para autenticarse, eliminando la necesidad de exponer contraseñas en código o variables.
- **Sin IP Pública**: Mantiene el clúster de AlloyDB en una red VPC privada sin acceso desde la Internet pública.

---

## 2. Uso del Conector Oficial de AlloyDB en Python (`app/db/alloydb_connector.py`)

Se puede integrar el paquete `google-cloud-alloydb-connector` directamente en la inicialización del motor de SQLAlchemy.

```python
import os
from google.cloud.alloydb.connector import Connector
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Configuración de variables de entorno para AlloyDB
PROJECT_ID = os.getenv("GCP_PROJECT_ID", "mi-proyecto-gcp")
REGION = os.getenv("GCP_REGION", "us-central1")
CLUSTER_NAME = os.getenv("ALLOYDB_CLUSTER", "mi-cluster-alloydb")
INSTANCE_NAME = os.getenv("ALLOYDB_INSTANCE", "instancia-principal")
DB_USER = os.getenv("ALLOYDB_USER", "usuario_app")
DB_PASS = os.getenv("ALLOYDB_PASSWORD", "password_secreta")
DB_NAME = os.getenv("ALLOYDB_NAME", "app_db")

# Construir URI del recurso AlloyDB
INSTANCE_URI = f"projects/{PROJECT_ID}/locations/{REGION}/clusters/{CLUSTER_NAME}/instances/{INSTANCE_NAME}"

def crear_engine_alloydb():
    """Crea un motor SQLAlchemy conectado de forma segura a AlloyDB usando el conector IAM."""
    # Inicializar el conector oficial de AlloyDB
    connector = Connector()

    def getconn():
        # Establecer la conexión con cifrado mTLS e identidad IAM
        conn = connector.connect(
            INSTANCE_URI,
            "pg8000",
            user=DB_USER,
            password=DB_PASS,
            db=DB_NAME
        )
        return conn

    # Crear motor SQLAlchemy utilizando la función de conexión conector
    engine = create_engine(
        "postgresql+pg8000://",
        creator=getconn,
        pool_size=10,
        max_overflow=20,
        pool_pre_ping=True
    )
    return engine
```

---

## 3. Separación de Instancia Principal y Réplicas de Lectura

AlloyDB permite crear réplicas de lectura de alto rendimiento para distribuir la carga de trabajo de consultas analíticas o de solo lectura.

```python
# Ejemplo de configuración para dirigir escrituras a la instancia principal
# y lecturas pesadas a la instancia réplica de AlloyDB

# URI de la instancia principal (Escrituras + Lecturas transaccionales)
PRIMARY_INSTANCE_URI = "projects/PROYECTO/locations/REGION/clusters/CLUSTER/instances/primary"

# URI de la instancia réplica (Solo Lecturas)
READ_REPLICA_INSTANCE_URI = "projects/PROYECTO/locations/REGION/clusters/CLUSTER/instances/read-replica"
```

---

## 4. Estrategia de Conexión Local vs Producción

El backend elegirá automáticamente el modo de conexión según el entorno de ejecución:

```python
import os
from app.db.session import engine as local_engine
from app.db.alloydb_connector import crear_engine_alloydb

def obtener_engine_activo():
    """Retorna el motor SQLAlchemy adecuado (Local Docker vs AlloyDB Cloud)."""
    entorno = os.getenv("ENTORNO", "desarrollo")
    
    if entorno == "produccion":
        # Conectar a AlloyDB en Google Cloud
        return crear_engine_alloydb()
    else:
        # Conectar a PostgreSQL local (Docker Compose)
        return local_engine
```
