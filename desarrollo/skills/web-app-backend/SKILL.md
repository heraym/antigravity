---
name: web-app-backend
description: Skill especializado para el desarrollo Backend en Python (FastAPI/Uvicorn/Pydantic) de aplicaciones web. Implementa los contratos de API y modelos descritos en el Documento de Diseño (SDD) con las mejores prácticas de Google Cloud Run (contenedor sin privilegios de root, Cloud Logging estructurado, Secret Manager, CORS y comprobación de salud /health).
---

# Skill de Desarrollo Backend (`web-app-backend`)

Este skill define el protocolo de desarrollo para que el subagente de Backend construya una API REST/GraphQL robusta, segura y escalable en **Python** basada en las especificaciones del Documento de Diseño de Software (`docs/design/system-design.md`).

---

## Misión y Responsabilidades del Agente Backend

1. **Consumir el Documento de Diseño**: Inspeccionar `docs/design/system-design.md` para extraer los contratos de API (endpoints, métodos HTTP, esquemas JSON), el modelo Entidad-Relación y los requerimientos de seguridad.
2. **Construir la Arquitectura Backend en Python**: Implementar la API usando **FastAPI** y **Uvicorn**, con validación mediante **Pydantic** y persistencia de datos.
3. **Asegurar Autenticación y Autorización**: Implementar firmado y verificación de tokens **JWT** (con hashing seguro `passlib[bcrypt]`).
4. **Habilitar CORS**: Configurar middleware CORS para autorizar las solicitudes provenientes del cliente Frontend.
5. **Cumplir Estándares de Google Cloud Run**:
   - Escuchar dinámicamente en el puerto configurado por la variable de entorno `$PORT` (8080 por defecto).
   - Proporcionar endpoint de salud (`GET /health`).
   - Emitir registros en formato JSON estructurado compatible con **Google Cloud Logging**.
   - Configurar `Dockerfile` no-root para ejecución segura.
6. **Comentarios de Código en Español**: Todos los archivos Python y scripts deben incluir comentarios descriptivos en español.

---

## Flujo de Trabajo Paso a Paso

### Paso 1: Inspección del Diseño
1. Abrir y analizar `docs/design/system-design.md`.
2. Identificar:
   - Especificaciones de endpoints (rutas, parámetros, cuerpos de solicitud y respuesta).
   - Esquemas de tablas de base de datos e identificadores.
   - Algoritmo de autenticación y caducidad de tokens.

### Paso 2: Estructuración del Proyecto Python
Organizar el código en una arquitectura modular limpia:

```text
backend/
├── app/
│   ├── main.py              # Punto de entrada FastAPI y configuración de middlewares
│   ├── core/
│   │   ├── config.py        # Lectura de variables de entorno y Secret Manager
│   │   ├── security.py      # Funciones JWT y hashing de contraseñas
│   │   └── logging.py       # Registros estructurados en JSON para Cloud Logging
│   ├── api/
│   │   ├── v1/
│   │   │   ├── router.py    # Enrutador principal de la API v1
│   │   │   ├── auth.py      # Endpoints de login y registro
│   │   │   └── recursos.py  # Endpoints de recursos del sistema
│   ├── schemas/             # Modelos de validación Pydantic
│   ├── models/              # Modelos de entidad ORM / DB
│   └── db/                  # Gestión de conexiones a la base de datos
├── requirements.txt         # Dependencias Python
├── Dockerfile               # Contenedor optimizado para Google Cloud Run
└── README.md
```

### Paso 3: Implementación de la Capa de API y Seguridad
1. Crear esquemas Pydantic para validar solicitudes y respuestas.
2. Implementar endpoints de autenticación (`/api/v1/auth/login`) que retornen tokens JWT válidos.
3. Proteger endpoints privados utilizando dependencias de seguridad de FastAPI (`Depends(obtener_usuario_actual)`).

Ver [Patrones de API Python](references/python-api-patterns.md) para ver ejemplos de código.

### Paso 4: Preparación para Google Cloud Run
1. Configurar `main.py` para incluir la ruta de comprobación de estado:
   ```python
   # Endpoint de salud exigido para monitoreo en Google Cloud Run / Load Balancers
   @app.get("/health", status_code=200)
   def verificar_salud():
       # Retornar estado activo del servicio backend
       return {"estado": "ok", "servicio": "backend-python"}
   ```
2. Crear un `Dockerfile` no-root optimizado para ejecuciones en Cloud Run.

Ver [Buenas Prácticas de Google Cloud para Backend](references/gcp-backend-best-practices.md).

### Paso 5: Verificación y Pruebas
1. Iniciar servidor de desarrollo local:
   ```bash
   ## Ejecutar servidor Uvicorn en modo desarrollo
   uvicorn app.main:app --reload --port 8080
   ```
2. Verificar que los endpoints cumplan con los contratos JSON del documento de diseño.

---

## Recursos del Skill

- [Patrones de API Python](references/python-api-patterns.md)
- [Buenas Prácticas de Google Cloud para Backend](references/gcp-backend-best-practices.md)
