# Mejores Prácticas de Google Cloud para Backend en Python

Esta guía define las pautas para preparar la aplicación backend Python para su despliegue y operación en **Google Cloud Run**.

---

## 1. Contenedorización Segura (Dockerfile no-root)

Google Cloud Run ejecuta contenedores en un entorno gestionado. Se debe utilizar una imagen liviana basada en `python:3.11-slim`, crear un usuario sin privilegios de administrador (`appuser`) y escuchar en la variable de entorno `$PORT`.

```dockerfile
# Imagen base liviana oficial de Python
FROM python:3.11-slim

# Evitar la escritura de archivos .pyc en disco
ENV PYTHONDONTWRITEBYTECODE=1
# Forzar el envío directo de registros a stdout/stderr sin búfer
ENV PYTHONUNBUFFERED=1

# Establecer directorio de trabajo dentro del contenedor
WORKDIR /app

# Instalar dependencias del sistema necesarias para la compilación
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Copiar archivo de requerimientos
COPY requirements.txt .

# Instalar dependencias de Python
RUN pip install --no-cache-dir -r requirements.txt

# Copiar el código fuente de la aplicación
COPY . .

# Crear usuario no-root por razones de seguridad
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Exponer la variable de puerto por defecto de Cloud Run
ENV PORT=8080
EXPOSE 8080

# Comando de inicio del servidor ASGI Uvicorn
CMD exec uvicorn app.main:app --host 0.0.0.0 --port ${PORT}
```

---

## 2. Registros Estructurados en JSON para Cloud Logging (`app/core/logging.py`)

Para que Google Cloud Logging analice los mensajes de error e impresiones de la aplicación correctamente (severidades INFO, WARNING, ERROR), los registros deben formatearse en JSON.

```python
import logging
import json
import sys

class FormateadorCloudLogging(logging.Formatter):
    """Formateador de registros en JSON para integración nativa con Google Cloud Logging."""
    
    def format(self, record: logging.LogRecord) -> str:
        # Estructura del registro compatible con Google Cloud Logging
        log_payload = {
            "severity": record.levelname,
            "message": record.getMessage(),
            "logger": record.name,
            "component": "backend-python",
        }
        
        # Incluir información de excepciones si existen
        if record.exc_info:
            log_payload["exception"] = self.formatException(record.exc_info)
            
        return json.dumps(log_payload)

def configurar_logging():
    """Configura el registrador raíz para emitir mensajes JSON a stdout."""
    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(FormateadorCloudLogging())
    
    root_logger = logging.getLogger()
    root_logger.setLevel(logging.INFO)
    root_logger.handlers = [handler]
```

---

## 3. Integración con Google Cloud Secret Manager (`app/core/config.py`)

Para la gestión de contraseñas de base de datos o claves secretas JWT, se recomienda acceder directamente a Secret Manager o inyectar los secretos como variables de entorno desde la consola de Cloud Run.

```python
import os
from google.cloud import secretmanager

def obtener_secreto(secret_id: str, default: str = "") -> str:
    """
    Obtiene el valor de un secreto almacenado en Google Cloud Secret Manager.
    Si se ejecuta localmente, recurre a la variable de entorno o al valor por defecto.
    """
    project_id = os.getenv("GCP_PROJECT_ID")
    
    # Si estamos en entorno local sin ID de proyecto GCP, usar variable de entorno
    if not project_id:
        return os.getenv(secret_id, default)
        
    try:
        # Crear cliente de Secret Manager
        client = secretmanager.SecretManagerServiceClient()
        name = f"projects/{project_id}/secrets/{secret_id}/versions/latest"
        
        # Acceder a la versión del secreto
        response = client.access_secret_version(request={"name": name})
        return response.payload.data.decode("UTF-8")
    except Exception as err:
        print(f"[Advertencia] No se pudo obtener el secreto {secret_id} desde Secret Manager: {err}")
        return os.getenv(secret_id, default)
```
