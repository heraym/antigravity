---
name: web-app-frontend
description: Skill especializado para el desarrollo de la interfaz de usuario (Frontend) de aplicaciones web con React, Node.js y TailwindCSS/Vanilla CSS, implementando las especificaciones del Documento de Diseño (SDD) y las mejores prácticas de Google Cloud (Cloud Run, contenedores multi-stage, variables de entorno).
---

# Skill de Desarrollo Frontend (`web-app-frontend`)

Este skill define las directrices y el flujo de trabajo para que el subagente de Frontend construya una aplicación web moderna, interactiva y de alto rendimiento basada en el Documento de Diseño de Software (`docs/design/system-design.md`).

---

## Misión y Responsabilidades del Agente Frontend

1. **Consumir el Documento de Diseño**: Inspeccionar `docs/design/system-design.md` para extraer paletas de colores, fuentes, layouts, vistas principales y especificaciones de endpoints de API.
2. **Realizar cuestionario con las preguntas de Interfaz al usuario**: Inspeccionar `skills/web-app-frontend/references/questionnaire.md` y haz las preguntas al usuario en base a las directivas de ese documento.
3. **Inicializar Proyecto React / Node.js**: Configurar el entorno frontend usando Vite con React (o Next.js) asegurando una estructura limpia y escalable.
4. **Aplicar el Sistema de Diseño**: Traducir los tokens de diseño (colores, tipografía, bordes, sombras) a CSS/TailwindCSS.
5. **Construir Componentes y Vistas**: Crear componentes modulares, manejadores de estado y flujos de navegación responsivos.
6. **Conectar con la Capa de API**: Construir un cliente API limpio con gestión de autenticación por tokens JWT y manejo de errores.
7. **Cumplir Buenas Prácticas de Google Cloud**: Preparar la aplicación para empaquetado optimizado en contenedor Docker multi-stage listo para despliegue en Google Cloud Run o Firebase Hosting.
8. **Documentar Código en Español**: Todos los comentarios en el código fuente deben estar redactados en español.

---

## Flujo de Trabajo Paso a Paso

### Paso 1: Lectura e Inspección del Diseño
1. Abrir y leer el archivo `docs/design/system-design.md`.
2. Identificar:
   - Paleta de colores primaria, secundaria, fondos y acentos.
   - Fuentes tipográficas de títulos y cuerpo.
   - Lista de vistas/pantallas del mapa del sitio.
   - Contratos de API REST/GraphQL (rutas, métodos HTTP, tokens de autenticación).

### Paso 2: Realizar cuestionario con las preguntas de Interfaz al usuario
1. Abrir y leer el archivo `skills/web-app-frontend/references/questionnaire.md` 
2. Realiza las preguntas usando la tool `ask_question` en base a las directivas de ese documento

### Paso 3: Inicialización del Proyecto
1. Crear el proyecto frontend en Node.js si no existe:
   ```bash
   ## Inicializar proyecto React con Vite en la carpeta actual o subcarpeta frontend
   npx -y create-vite@latest ./ --template react
   npm install
   ```
2. Instalar dependencias necesarias (ej. `react-router-dom`, `lucide-react` para iconos, etc.).

### Paso 4: Configuración de Estilos y Tokens
1. Actualizar `index.css` definiendo las variables CSS del sistema de diseño extraído:
   ```css
   /* Variables globales de diseño extraídas del Documento de Diseño */
   :root {
     /* Color principal de la marca */
     --color-primario: #6366f1;
     /* Color secundario de acento */
     --color-secundario: #0ea5e9;
     /* Fondo principal oscuro/claro */
     --color-fondo: #0f172a;
     /* Color de contenedores y tarjetas */
     --color-superficie: #1e293b;
     /* Color de texto principal */
     --color-texto: #f8fafc;
   }
   ```

### Paso 5: Construcción de la Arquitectura de Componentes
Seguir la estructura de carpetas modular:
- `src/components/`: Componentes UI reutilizables (Botones, Tarjetas, Inputs, Skeletons).
- `src/pages/`: Vistas o pantallas completas correspondientes al mapa del sitio.
- `src/services/`: Clientes de API y funciones de integración.
- `src/context/`: Contextos globales (ej. Autenticación, Tema).

Ver [Patrones de Componentes React](references/component-patterns.md) para más detalles.

### Paso 6: Preparación para Google Cloud (Cloud Run / Firebase)
1. Configurar variables de entorno prefijadas con `VITE_` (ej. `VITE_API_BASE_URL`).
2. Incluir el archivo `Dockerfile` de producción usando construcciones multi-stage (Node.js para build, Nginx Alpine para servir estáticos de alto rendimiento).

Ver [Buenas Prácticas de Google Cloud para Frontend](references/gcp-frontend-best-practices.md).

### Paso 7: Verificación y Compilación
1. Validar compilación ejecutable: `npm run build`.
2. Verificar que no haya errores ni advertencias de consola.

---

## Recursos del Skill

- [Patrones de Componentes React](references/component-patterns.md)
- [Buenas Prácticas de Google Cloud para Frontend](references/gcp-frontend-best-practices.md)
