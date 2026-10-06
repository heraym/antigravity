# Cuestionario Guiado de Diseño de Aplicaciones Web

Este cuestionario sirve como catálogo de referencia para que el Agente de Diseño realice preguntas al usuario durante la etapa de conceptualización. El agente **no debe lanzar todas las preguntas juntas**, sino seleccionar las relevantes según el grado de detalle proporcionado inicialmente.

---

## Categoría 0: Branding e Identidad Visual de Referencia
1. **¿Deseas basar el diseño en la marca o identidad visual de un sitio web existente?**
   - *Ejemplo*: "Sí, utiliza el branding de https://acme.com o de mi sitio web actual https://mi-empresa.com."
   - *Nota*: Si se especifica una URL, el agente utilizará la herramienta `read_url_content` para analizar los estilos, paleta de colores y tipografía del sitio indicado.
2. **Si no hay un sitio web de referencia, ¿prefieres utilizar la URL de branding por defecto configurada en el skill?**

---

## Categoría 1: Negocio, Propósito y Audiencia
1. **¿Cuál es el objetivo principal de la aplicación web?**
   - *Ejemplo*: "Es una plataforma SaaS de gestión de tareas colaborativa en tiempo real."
2. **¿Quiénes son los usuarios finales primarios y secundarios?**
   - *Ejemplo*: "Usuarios finales (empleados) y Administradores de equipo."
3. **¿Cuáles son los 3 casos de uso indispensables (MVP) para considerar exitosa la primera versión?**

---

## Categoría 2: Diseño de Interfaz y Experiencia de Usuario (UI/UX)
1. **¿Qué estilo estético prefieres para la interfaz si no proviene del sitio de referencia?**
   - *Opciones*: Dark mode premium con glassmorphism, Light mode limpio y minimalista, Neumorphism, o paleta corporativa específica.
2. **¿Existe alguna variación de color que quieras aplicar sobre el branding de referencia?**
3. **¿Cuáles son las vistas o pantallas clave que la app necesita?**
   - *Ejemplo*: Landing page, Pantalla de Login/Registro, Dashboard principal, Vista de detalles y Configuración de perfil.
4. **¿Se requieren animaciones o interacciones complejas?**
   - *Ejemplo*: Drag-and-drop de tarjetas, gráficos interactivos en vivo, transiciones suaves.

---

## Categoría 3: Arquitectura de Software y Stack Tecnológico
1. **¿Tienes alguna preferencia en el framework de Frontend?**
   - *Opciones*: HTML5/Vanilla JS, React (con Vite), Next.js, Vue.js, Svelte.
2. **¿Qué tecnología o lenguaje prefieres para el Backend?**
   - *Opciones*: Node.js (Express / NestJS / Fastify), Python (FastAPI / Django), Go, Java (Spring Boot), o Backend as a Service (Firebase / Supabase).
3. **¿Cómo se comunicará el Frontend con el Backend?**
   - *Opciones*: REST API tradicional, GraphQL, o WebSockets para tiempo real.

---

## Categoría 4: Persistencia y Modelo de Datos
1. **¿Qué tipo de datos procesará y almacenará la aplicación?**
   - *Ejemplo*: Datos estructurados relacionales (usuarios, pedidos, pagos) o documentos semi-estructurados / archivos multimedia.
2. **¿Tienes preferencia de motor de Base de Datos?**
   - *Opciones*: PostgreSQL, MySQL, MongoDB, SQLite, Firestore.
3. **¿Cuál es el volumen estimado de datos o tráfico esperado?**

---

## Categoría 5: Seguridad, Autenticación y Despliegue (SRE)
1. **¿Cómo deben autenticarse los usuarios?**
   - *Opciones*: Correo y contraseña con JWT, OAuth2 con Google/GitHub, Firebase Auth, Auth0.
2. **¿Existirán diferentes roles o niveles de permisos (RBAC)?**
   - *Ejemplo*: Rol 'Admin' (acceso total), Rol 'Editor' (crear y editar), Rol 'Lector' (solo lectura).
3. **¿En qué infraestructura se planea desplegar la aplicación?**
   - *Opciones*: Google Cloud Run, Vercel, Netlify, Kubernetes (GKE), Docker en servidor dedicado.
