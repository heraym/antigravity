# Mejores Prácticas de Google Cloud para aplicaciones Frontend React/Node.js

Esta guía proporciona la configuración óptima para empaquetar, asegurar y desplegar aplicaciones frontend construidas con React y Node.js en servicios como **Google Cloud Run** o **Firebase Hosting**.

---

## 1. Contenedorización Multi-Stage (Dockerfile de Producción)

Para despliegues eficientes en **Google Cloud Run**, es imprescindible utilizar construcciones en varias etapas (multi-stage builds) para obtener imágenes livianas (< 30 MB) y de alta velocidad de arranque.

```dockerfile
# =========================================================
# Etapa 1: Construcción de la aplicación (Node.js)
# =========================================================
FROM node:20-alpine AS construccion

# Establecer directorio de trabajo en el contenedor
WORKDIR /app

# Copiar archivos de definición de dependencias
COPY package*.json ./

# Instalar dependencias de desarrollo y producción
RUN npm ci

# Copiar todo el código fuente del proyecto
COPY . .

# Argumentos de compilación para variables de entorno
ARG VITE_API_BASE_URL
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL

# Compilar los archivos estáticos optimizados
RUN npm run build

# =========================================================
# Etapa 2: Servidor web de producción ligero (Nginx Alpine)
# =========================================================
FROM nginx:alpine-slim

# Copiar configuración personalizada de Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copiar artefactos compilados desde la etapa de construcción
COPY --from=construccion /app/dist /usr/share/nginx/html

# Exponer el puerto configurado para Google Cloud Run (puerto 8080 por defecto)
EXPOSE 8080

# Iniciar servidor Nginx en primer plano
CMD ["nginx", "-g", "daemon off;"]
```

---

## 2. Configuración de Nginx para Single Page Applications (`nginx.conf`)

Garantiza la gestión correcta de rutas client-side (React Router) y la compresión gzip.

```nginx
# Configuración del servidor HTTP para Google Cloud Run
server {
    # Escuchar en el puerto exigido por Cloud Run ($PORT o 8080)
    listen 8080;
    server_name localhost;

    # Raíz donde se ubican los archivos estáticos de React
    root /usr/share/nginx/html;
    index index.html;

    # Habilitar compresión Gzip para optimizar transferencia
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # Redirección de todas las rutas de SPA hacia index.html
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Caché agresivo para activos inmutables (JS, CSS, Imágenes)
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, no-transform";
    }

    # Cabeceras de seguridad HTTP recomendadas por Google Cloud
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";
}
```

---

## 3. Manejo Seguro de Variables de Entorno

- **Desarrollo Local**: Utilizar `.env.local` con variables prefijadas con `VITE_` (ejemplo: `VITE_API_BASE_URL=http://localhost:3000/api/v1`).
- **Despliegue en Cloud Run**: Pasar las variables durante el proceso de Build en Cloud Build o Artifact Registry, evitando incluir tokens o secretos en las imágenes de contenedor.
- **Inyección de Secretos**: Si el frontend requiere llaves públicas (ejemplo: Firebase API Key, Mapbox Token), configurarlas desde **Google Cloud Secret Manager**.
