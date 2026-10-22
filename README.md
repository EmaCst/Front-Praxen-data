# Praxen · Panel administrativo

Frontend estático del sistema de prevención de lesiones mediante entrenamiento neuromuscular. Permite a administradores autenticarse, consultar los resultados reales y comparar el rendimiento por ejercicio.

## Arquitectura

- Frontend: HTML, CSS y JavaScript nativo, sin dependencias ni compilación.
- API: `https://api-praxen.onrender.com` ([API-Praxen](https://github.com/EmaCst/API-Praxen)).
- Autenticación: `POST /api/auth/login` + validación del rol con `GET /api/auth/me`.
- Gestión de cuentas (solo `superadmin`): listado, creación de administradores, activación/desactivación y auditoría mediante `/api/admin/administradores`.
- Datos: rutas protegidas `/api/admin/atletas`, `/api/admin/federaciones`, `/api/admin/resultados/*`.

Las tres modalidades se identifican como `aimlab`, `feet_targets` y `running_direction_change`. Las estadísticas se obtienen directamente de la API, que aplica una agregación equitativa entre atletas. No se mezclan métricas con unidades o sentidos de mejora distintos.

## Despliegue en Cloudflare Pages

1. Subir este repositorio a GitHub.
2. En Cloudflare, **Workers & Pages → Create application → Pages → Connect to Git**.
3. Elegir `EmaCst/Front-Praxen-data` y la rama principal.
4. Framework preset: `None`; comando de compilación: vacío; directorio de salida: `.` (raíz del repositorio).
5. Desplegar. Los cambios posteriores a la rama principal se publicarán automáticamente.

## Configuración obligatoria en Render

Agregar el origen HTTPS exacto del frontend a `CORS_ORIGINS` del servicio API, conservando los orígenes que ya se utilicen. Ejemplo:

```env
CORS_ORIGINS=https://praxen.pages.dev
```

Si se usa un dominio personalizado, incluir también su origen. Tras cambiar la variable, comprobar que las solicitudes `OPTIONS` y `POST` desde el navegador funcionan. No habilitar CORS universal para datos administrativos.

La cuenta `admin` se aprovisiona desde una terminal confiable del backend, como se explica en `API-Praxen/docs/admin-results-api.md`. El registro público no crea administradores.

El bootstrap crea ahora un `superadmin`, único rol que puede acceder a **Administradores**. Desde el panel solo se crean cuentas `admin`; no se pueden promover superadministradores. El estado de las cuentas se verifica en cada solicitud a la API.

## Seguridad y límites

- Token JWT en memoria únicamente: al actualizar la pestaña es necesario volver a iniciar sesión.
- No hay credenciales ni secretos incluidos en el repositorio.
- El control de permisos real ocurre en la API, no en el frontend.
- La exportación CSV incluye solo las 20 sesiones recientes cargadas en la vista, no todo el historial.
- La búsqueda de atletas requiere al menos dos caracteres; las listas son paginadas.
- Los selectores de comparación muestran los atletas cargados en la página actual del directorio. Se puede navegar el directorio para cambiar ese conjunto.
- La federación mostrada es un dato declarado por el atleta, todavía no verificado.
- Los análisis no prueban eficacia clínica ni prevención de lesiones sin una evaluación validada.

## Verificación

```bash
node --check app.js
```

Se recomienda verificar con un usuario administrador en Render que las migraciones estén aplicadas, que la cuenta tenga permisos y que CORS permita el origen publicado. La API puede tardar en responder después de un período de inactividad en planes gratuitos.
