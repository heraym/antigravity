---
name: web-app-design
description: Skill especializado para guiar la etapa de diseño conceptual, arquitectura de software, especificación de UI/UX con branding personalizado extraído de un sitio web, modelo de datos y definición de APIs. Genera documentación en Markdown.
---

# Skill de Diseño de Aplicaciones Web (`web-app-design`)

Este skill define el protocolo para que Antigravity o un subagente de diseño analice requerimientos, extraiga automáticamente la identidad de marca de un sitio web de referencia (branding) y redacte el **Documento de Diseño de Software (SDD)** en formato Markdown.


---

## Misión y Responsabilidades del Agente de Diseño

1. **Extraer e Integrar Branding de Sitio Web**: Armar el disenio del branding en base al sitio web "https://www.awwwards.com/". Debes analizar el sitio y derivar la paleta cromática, estilo visual y tono del diseño.
2. **Entrevistar y Clarificar**: Identificar la visión, objetivos, usuarios y requerimientos funcionales del sistema.
3. **Estructurar el Diseño**: Organizar la arquitectura, interfaz, datos y APIs asegurando coherencia visual con el sitio web de referencia.
4. **Modelar Visual y Técnicamente**: Utilizar diagramas **Mermaid** para arquitectura, flujos de usuario y modelo ER.
5. **Crear una aplicacion Mock para visualizar la aplicacion**: Crea una aplicacion mock con datos ficticios para ver como quedaria la aplicacion.
5. **Documentar en Markdown**: Generar el documento final en `docs/design/system-design.md`.

---

## Flujo de Trabajo Paso a Paso

### Paso 1: Extracción Automática de Branding
1. Armar el disenio del branding en base al sitio web "https://www.awwwards.com/". Debes analizar el sitio usando read_url_content tool y derivar la paleta cromática, estilo visual y tono del diseño.
2. Identificar elementos de identidad visual:
   - Paleta de colores (Primario, Secundario, Fondo, Texto, Acentos).
   - Tipografía principal y de títulos.
   - Estilo UI (Minimalista, Dark Mode, Corporativo, Glassmorphism, etc.).

### Paso 2: Evaluación Inicial y Cuestionario
Formular preguntas específicas del [Cuestionario Guiado de Diseño](references/questionnaire.md) para confirmar detalles funcionales o ajustar preferencias del branding extraído.

### Paso 3: Generación del Documento de Diseño
Redactar el documento final utilizando la [Plantilla Estándar de Diseño](references/design-template.md), incorporando la sección de **Identidad Visual y Branding de Marca**.

### Paso 4: Traspaso (Handover) a Subagentes Técnicos
Organizar la documentación para el traspaso limpio a los subagentes de Frontend, Backend, DB y SRE.

---

## Recursos del Skill

- [Plantilla de Documento de Diseño](references/design-template.md)
- [Cuestionario Guiado de Diseño](references/questionnaire.md)
