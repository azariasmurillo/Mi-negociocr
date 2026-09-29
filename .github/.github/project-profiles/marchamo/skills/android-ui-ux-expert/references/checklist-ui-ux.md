# Checklist de revisión UI/UX — antes de aprobar cualquier PR con cambios de pantalla

Marcar todos. Un "no" en las secciones 1–3 bloquea el merge.

## 1. Estados de pantalla (obligatorio)

- [ ] La pantalla modela **todos** los estados: `Loading`, `Empty`, `Content`, `Error` (sealed interface, nunca booleanos sueltos).
- [ ] Ningún error se muestra como texto genérico ("Error") — siempre mensaje humano + acción de recuperación.
- [ ] `Empty` usa el componente `EmptyState` compartido (ícono + título + siguiente paso).
- [ ] Botones deshabilitados cuando su acción no aplica (ej. `Limpiar` con lista vacía) y habilitados solo con precondiciones cumplidas.
- [ ] Acciones destructivas (`Limpiar`, `Cancelar sesión`) piden confirmación con `ConfirmDialog`.

## 2. Accesibilidad (obligatorio)

- [ ] Contraste texto/fondo ≥ 4.5:1 (texto normal) o ≥ 3:1 (texto grande ≥ 18sp). Verificar con el Accessibility Scanner o a ojo contra los tokens del theme.
- [ ] Todos los íconos de acción tienen `contentDescription`; los decorativos usan `null`.
- [ ] Touch targets ≥ 48×48dp (usar `minimumInteractiveComponentSize()` si el visual es menor).
- [ ] Texto en `sp`, nunca `dp`. La pantalla no se rompe con escala de fuente 1.5×.
- [ ] Contadores/estados que cambian solos usan `semantics { liveRegion = LiveRegionMode.Polite }`.

## 3. Consistencia (obligatorio)

- [ ] Labels de navegación provienen de `AppDestination` (una sola fuente de verdad drawer/home/pantalla).
- [ ] Título de pantalla SOLO en la `TopAppBar` — nunca duplicado como heading debajo.
- [ ] Destinos top-level → hamburguesa; pantallas hijas → back arrow. Nunca ambos criterios mezclados.
- [ ] Todos los strings en `strings.xml`, revisados ortográficamente (tildes, ortografía del español).
- [ ] Cero colores hardcodeados — solo `MaterialTheme.colorScheme.*`.
- [ ] Cero identificadores técnicos crudos (GUIDs, EPCs completos, URLs) visibles al usuario final: truncar + copiar.

## 4. Patrón de pantalla de escaneo RFID

- [ ] Usa `InsRfidControlBar` del design-system (no implementación ad-hoc de conexión/escaneo).
- [ ] El botón de escaneo es un toggle: "Iniciar escaneo" ↔ "Detener escaneo" (tonal `error` al estar activo).
- [ ] Contador de etiquetas visible en vivo, con `fontFeatureSettings = "tnum"` o `Modifier.animateContentSize` controlado para que no salte.
- [ ] Haptic feedback al detectar tags nuevos (el operador no mira la pantalla).
- [ ] La lista de tags usa `LazyColumn` con `key = { it.epc }` (evita recomposiciones y saltos al escanear en ráfaga).

## 5. Jerarquía visual

- [ ] Título de tarjeta/ítem con más peso visual que su descripción (`titleMedium onSurface` > `bodyMedium onSurfaceVariant`).
- [ ] Banners informativos en superficie tonal (`surfaceContainerHigh` / `secondaryContainer`) — nunca bloques blancos sobre fondo oscuro.
- [ ] Máximo UN botón `InsPrimaryButton` (acción principal) visible por pantalla; secundarias en `InsSecondaryButton`/`TextButton`.
- [ ] Spacing en grilla de 4dp (8/12/16/24) usando constantes `InsSpacing.*`, no valores mágicos.

## 6. Antes de cerrar

- [ ] Probado en el dispositivo de referencia (REA-NX9, Android 14) en condiciones reales de luz.
- [ ] Probado el flujo con el lector desconectándose a mitad de escaneo.
- [ ] Screenshots del antes/después adjuntos al PR.
