# Instrucciones neutrales para mi-negociocr-app

## Hechos verificados

- El source Flutter está bajo `lib/`, con `core` y `features` por dominio; hay repositorios de datos por feature y pantallas/widgets.
- `main.dart` inicializa Flutter y Riverpod; Sentry se inicializa solo cuando la configuración lo habilita.
- `sessionProvider` administra la sesión; `SessionStorage` persiste token, URL de API, identidad y `business_id` con almacenamiento seguro; el flujo de recordar acceso también puede persistir la contraseña en ese almacenamiento.
- `ApiClient` usa Dio, añade Bearer, maneja timeouts, refresh de token y cierre ante 401 no recuperable.
- Los repositorios reciben `ApiClient` y normalmente el `BusinessId`; revisa cada endpoint porque el tenant puede ir en ruta, query o body.
- La navegación verificada usa `MaterialApp` y `Navigator`. No asumas otra librería de routing sin inspección.

## Reglas de trabajo

- Mantén la sesión, refresh, logout y expiración en los puntos existentes; no implementes un segundo cliente HTTP o almacenamiento paralelo.
- No expongas tokens, claves ni contraseñas (incluida la contraseña guardada para recordar acceso) en logs, errores, capturas o documentación.
- Valida que exista `BusinessId` antes de llamar a un repositorio que lo requiere y evita mezclar sesiones.
- Reutiliza modelos y repositorios de la feature. No pongas llamadas Dio directamente en widgets si ya existe un repositorio/servicio apropiado.
- Para cambios de sincronización, ventas, créditos, pagos o inventario, verifica primero el contrato API y marca cualquier comportamiento móvil no confirmado como pendiente.

## Pendiente de verificación

Esta fase no establece una convención de router, una arquitectura de estado adicional ni un contrato offline: solo se documenta lo observado en el source actual.
