# Patrones de Código Python (FastAPI, Pydantic y JWT)

Esta guía documenta los patrones de implementación en Python recomendados para construir las APIs backend del proyecto.

---

## 1. Esquemas de Validación Pydantic (`app/schemas/usuario.py`)

Los esquemas de Pydantic garantizan la validación automática del cuerpo de las peticiones HTTP y la serialización de respuestas.

```python
from pydantic import BaseModel, EmailStr, Field
from typing import Optional
from datetime import datetime

# Esquema base para los datos comunes del usuario
class UsuarioBase(BaseModel):
    # Correo electrónico validado
    email: EmailStr = Field(..., description="Correo electrónico del usuario")
    # Nombre completo del usuario
    nombre: str = Field(..., min_length=2, max_length=100, description="Nombre completo")

# Esquema para la solicitud de registro o creación de usuario
class UsuarioCrear(UsuarioBase):
    # Contraseña en texto plano suministrada durante el registro
    password: str = Field(..., min_length=8, description="Contraseña segura")

# Esquema para la respuesta pública del perfil de usuario (sin hash de contraseña)
class UsuarioRespuesta(UsuarioBase):
    # Identificador único en base de datos
    id: str = Field(..., description="Identificador único del usuario")
    # Fecha de registro en formato ISO
    fecha_creacion: datetime = Field(..., description="Fecha de creación del registro")

    class Config:
        # Habilitar modo ORM para compatibilidad con SQLAlchemy/SQLModel
        from_attributes = True
```

---

## 2. Gestión de Seguridad y JWT (`app/core/security.py`)

Proporciona funciones para el hashing de contraseñas y la generación de tokens de acceso firmados.

```python
import os
from datetime import datetime, timedelta
from typing import Optional
from jose import jwt, JWTError
from passlib.context import CryptContext

# Contexto de hashing para contraseñas usando bcrypt
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Llave secreta extraída de las variables de entorno o Secret Manager
SECRET_KEY = os.getenv("JWT_SECRET_KEY", "clave_secreta_por_defecto_cambiar_en_produccion")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 60

def verificar_password(password_plano: str, password_hash: str) -> bool:
    """Verifica si la contraseña ingresada coincide con el hash almacenado."""
    return pwd_context.verify(password_plano, password_hash)

def obtener_password_hash(password: str) -> str:
    """Genera un hash seguro bcrypt a partir de una contraseña en texto plano."""
    return pwd_context.hash(password)

def crear_token_acceso(data: dict, expiracion_delta: Optional[timedelta] = None) -> str:
    """Genera un token JWT firmado con un tiempo de expiración determinado."""
    to_encode = data.copy()
    if expiracion_delta:
        expire = datetime.utcnow() + expiracion_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    
    # Agregar timestamp de expiración al payload del token
    to_encode.update({"exp": expire})
    
    # Firmar y codificar el token JWT
    token_codificado = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return token_codificado
```

---

## 3. Configuración de App Principal y CORS (`app/main.py`)

Punto de entrada de FastAPI con habilitación de CORS, manejo de middleware y rutas.

```python
import os
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.v1.auth import router as auth_router

# Inicializar la aplicación FastAPI con metadatos
app = FastAPI(
    title="API Backend Web App",
    description="Servicio backend REST desarrollado en Python con FastAPI para Google Cloud Run",
    version="1.0.0"
)

# Configurar orígenes permitidos para CORS (Frontend)
ORIGENES_PERMITIDOS = [
    os.getenv("FRONTEND_URL", "http://localhost:5173"),
    "http://localhost:3000",
]

# Agregar middleware CORS para permitir peticiones desde la aplicación web
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # En producción restringir a ORIGENES_PERMITIDOS
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Endpoint de comprobación de salud para monitoreo de Google Cloud Run
@app.get("/health", tags=["Monitoreo"])
def verificar_salud():
    """Endpoint de salud (Health Check) utilizado por Cloud Run."""
    return {"estado": "ok", "servicio": "backend-python"}

# Registrar los enrutadores de la versión 1 de la API
app.include_router(auth_router, prefix="/api/v1/auth", tags=["Autenticación"])
```
