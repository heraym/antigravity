# Patrones de Componentes React y Consumo de APIs

Esta guía detalla los patrones recomendados para estructurar componentes React, manejar estados globales/locales y conectar con servicios backend.

---

## 1. Cliente de API Centralizado (`src/services/apiClient.js`)

Se debe utilizar un cliente centralizado que gestione la URL base, cabeceras de autenticación JWT e intercepción de errores.

```javascript
// URL base de la API backend proveniente de las variables de entorno de Vite
const BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000/api/v1';

/**
 * Función auxiliar para realizar peticiones HTTP autenticadas
 * @param {string} endpoint - Ruta relativa del endpoint (ej. '/auth/login')
 * @param {object} options - Opciones de la petición fetch (method, headers, body)
 */
export async function solicitarAPI(endpoint, options = {}) {
  // Obtener el token de autenticación del almacenamiento local
  const token = localStorage.getItem('token_sesion');

  // Cabeceras por defecto para la comunicación JSON
  const headers = {
    'Content-Type': 'application/json',
    ...(token && { Authorization: `Bearer ${token}` }),
    ...options.headers,
  };

  try {
    // Ejecutar petición HTTP
    const respuesta = await fetch(`${BASE_URL}${endpoint}`, {
      ...options,
      headers,
    });

    // Validar respuesta no exitosa
    if (!respuesta.ok) {
      const errorDatos = await respuesta.json().catch(() => ({}));
      throw new Error(errorDatos.mensaje || `Error en la petición HTTP: ${respuesta.status}`);
    }

    // Retornar respuesta parseada
    return await respuesta.json();
  } catch (error) {
    // Registrar error para trazabilidad
    console.error(`[Error de API en ${endpoint}]:`, error.message);
    throw error;
  }
}
```

---

## 2. Componente de UI Reutilizable con Estados (`src/components/BotonAccion.jsx`)

Los componentes de interfaz deben ser receptivos, accesibles y contar con estados visuales de carga (loading) y deshabilitado.

```jsx
import React from 'react';

/**
 * Componente de Botón de Acción Principal
 * @param {object} props - Propiedades del botón (texto, onClick, cargando, deshabilitado)
 */
export function BotonAccion({ texto, onClick, cargando = false, deshabilitado = false, tipo = 'button' }) {
  return (
    <button
      type={tipo}
      onClick={onClick}
      disabled={deshabilitado || cargando}
      className="boton-primario"
    >
      {cargando ? (
        // Indicador visual de carga
        <span className="spinner-carga">Procesando...</span>
      ) : (
        // Texto normal del botón
        <span>{texto}</span>
      )}
    </button>
  );
}
```

---

## 3. Contexto Global de Autenticación (`src/context/AuthContext.jsx`)

Maneja el estado de sesión del usuario en toda la aplicación.

```jsx
import React, { createContext, useState, useEffect, useContext } from 'react';
import { solicitarAPI } from '../services/apiClient';

// Crear el contexto de autenticación
const ContextoAutenticacion = createContext(null);

/**
 * Proveedor de Estado de Autenticación
 */
export function ProveedorAutenticacion({ children }) {
  const [usuario, setUsuario] = useState(null);
  const [cargando, setCargando] = useState(true);

  // Validar sesión activa al cargar la aplicación
  useEffect(() => {
    const token = localStorage.getItem('token_sesion');
    if (token) {
      solicitarAPI('/auth/perfil')
        .then((datos) => setUsuario(datos.usuario))
        .catch(() => localStorage.removeItem('token_sesion'))
        .finally(() => setCargando(false));
    } else {
      setCargando(false);
    }
  }, []);

  /**
   * Función para iniciar sesión
   */
  const iniciarSesion = async (credenciales) => {
    const respuesta = await solicitarAPI('/auth/login', {
      method: 'POST',
      body: JSON.stringify(credenciales),
    });

    // Guardar token y actualizar estado de usuario
    localStorage.setItem('token_sesion', respuesta.token);
    setUsuario(respuesta.usuario);
  };

  /**
   * Función para cerrar sesión
   */
  const cerrarSesion = () => {
    localStorage.removeItem('token_sesion');
    setUsuario(null);
  };

  return (
    <ContextoAutenticacion.Provider value={{ usuario, cargando, iniciarSesion, cerrarSesion }}>
      {children}
    </ContextoAutenticacion.Provider>
  );
}

/**
 * Hook personalizado para acceder a la autenticación
 */
export const useAuth = () => useContext(ContextoAutenticacion);
```
