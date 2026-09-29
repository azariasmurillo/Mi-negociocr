---
name: android-device-deploy
description: "Compila e instala la app MarchamoDigital.AndroidRFID en un dispositivo físico Android conectado por USB-C y la lanza automáticamente. USA CUANDO: probar la app en el móvil, instalar APK en dispositivo, correr app Android en físico, testear RFID en Zebra real, desplegar a móvil, install device, adb install, run on device, probar pantalla en el teléfono."
argument-hint: "Indica el flavor a instalar: 'local' (K8s local), 'dev' (Azure ALB dev) o 'prod'. Por defecto: local."
user-invocable: true
---

# Android Device Deploy — MarchamoDigital.AndroidRFID

---

## Framework Integration

Este Skill pertenece al Project Profile Marchamo del AI Engineering Framework.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Compilar e instalar la app `MarchamoDigital.AndroidRFID` en un dispositivo Android físico y lanzarla automáticamente.

**Problema que resuelve:**
Probar la app en el dispositivo real requiere conocer comandos específicos de Gradle, ADB y los flavors del proyecto.

**Responsabilidades que NO tiene este Skill:**
- No modifica código fuente de la app.
- No realiza operaciones Git.
- No interactúa con el backend .NET.

---

## Propósito

Compilar e instalar la app **MarchamoDigital.AndroidRFID** directamente en un dispositivo Android físico conectado por USB-C, y lanzarla automáticamente sin intervención manual en el teléfono.

---

## Prerrequisitos

| Requisito | Cómo verificar |
|-----------|---------------|
| Dispositivo Android conectado por USB-C | `adb devices` → debe aparecer `device` (no `unauthorized`) |
| **Depuración USB activa** en el teléfono | Ajustes → Opciones de desarrollador → Depuración USB |
| **adb** disponible | Path: `/home/ftorres/Android/Sdk/platform-tools/adb` |
| Android SDK instalado | `/home/ftorres/Android/Sdk/` |
| Gradle wrapper funcional | `./gradlew --version` desde el repo |

### Dispositivo de referencia
- **Modelo:** REA-NX9 (Honeywell — terminal de captura con pistola RFID)
- **Android:** 14
- **ID USB:** `AB5X024119001840`

---

## Flavors disponibles

| Flavor | Package ID | Backend URL | Uso |
|--------|-----------|-------------|-----|
| `local` | `cr.fi.ins.marchamorfid.local` | `http://192.168.100.18:8080/` | K8s local (kind) |
| `dev` | `cr.fi.ins.marchamorfid.dev` | `https://f2bag7c7d6g5bvbn.fz31.alb.azure.com/` | Azure ALB dev |
| `prod` | `cr.fi.ins.marchamorfid` | `https://f2bag7c7d6g5bvbn.fz31.alb.azure.com/` | Producción |

---

## Procedimiento

### PASO 1 — Configurar adb en el PATH

```bash
export PATH="$PATH:/home/ftorres/Android/Sdk/platform-tools"
```

> Agregar esta línea al `~/.bashrc` o `~/.bash_profile` para que sea permanente.

### PASO 2 — Verificar dispositivo conectado

```bash
adb devices
```

Salida esperada:
```
List of devices attached
AB5X024119001840        device
```

Si aparece `unauthorized`: desbloquear el teléfono y aceptar el diálogo de confianza de depuración USB.

### PASO 3 — Compilar e instalar (en un solo comando)

```bash
cd /home/ftorres/GitHub_INS/MarchamoDigital.AndroidRFID

# Flavor local (K8s local)
./gradlew installLocalDebug

# Flavor dev (Azure ALB dev)  
./gradlew installDevDebug

# Flavor prod
./gradlew installProdDebug
```

Salida esperada al finalizar:
```
Installing APK 'app-local-debug.apk' on 'REA-NX9 - 14' for :app:local-debug
Installed on 1 device.
BUILD SUCCESSFUL in 2m 9s
```

### PASO 4 — Lanzar la app automáticamente

```bash
# Flavor local
adb shell am start -n "cr.fi.ins.marchamorfid.local/cr.fi.ins.marchamorfid.MainActivity"

# Flavor dev
adb shell am start -n "cr.fi.ins.marchamorfid.dev/cr.fi.ins.marchamorfid.MainActivity"

# Flavor prod
adb shell am start -n "cr.fi.ins.marchamorfid/cr.fi.ins.marchamorfid.MainActivity"
```

### Comandos combinados (instalar + lanzar)

```bash
export PATH="$PATH:/home/ftorres/Android/Sdk/platform-tools"
cd /home/ftorres/GitHub_INS/MarchamoDigital.AndroidRFID

# Local (más común para desarrollo)
./gradlew installLocalDebug && \
  adb shell am start -n "cr.fi.ins.marchamorfid.local/cr.fi.ins.marchamorfid.MainActivity"
```

---

## Logs en tiempo real

Ver logs de la app durante la prueba:

```bash
# Solo logs relevantes de la app
adb logcat --pid=$(adb shell pidof -s cr.fi.ins.marchamorfid.local) -v time

# Logs de conversión EPC → CodTagId
adb logcat -s "EpcTagIdConverter:D" "RfidConversion:D" -v time

# Logs de red (Retrofit)
adb logcat -s "OkHttp:D" -v time

# Limpiar y ver todo desde ahora
adb logcat -c && adb logcat -v time | grep -i "marchamorfid\|RFID\|EPC\|TagId"
```

---

## Troubleshooting

| Problema | Causa | Solución |
|----------|-------|---------|
| `adb: instrucción no encontrada` | adb no está en PATH | `export PATH="$PATH:/home/ftorres/Android/Sdk/platform-tools"` |
| `unauthorized` en `adb devices` | Teléfono no confía en la PC | Aceptar diálogo en el teléfono |
| `offline` en `adb devices` | Cable USB o modo de conexión incorrecto | Desconectar y reconectar; cambiar modo USB a "Transferencia de archivos" |
| `INSTALL_FAILED_VERSION_DOWNGRADE` | Versión instalada es más nueva | `adb uninstall cr.fi.ins.marchamorfid.local && ./gradlew installLocalDebug` |
| `BUILD FAILED` en Gradle | Error de compilación | Ver el mensaje completo y corregir antes de reinstalar |
| App instala pero no conecta al backend | VPN o IP incorrecta en flavor local | Verificar IP del host con `ip route get 1 \| awk '{print $7}'` y actualizar `app/build.gradle.kts` |

---

## Verificar IP del host (para flavor local)

El flavor `local` apunta a `192.168.100.18`. Si la IP de la máquina cambia:

```bash
# Obtener la IP actual del host
ip route get 1 | awk '{print $7}'
```

Actualizar en `app/build.gradle.kts`:
```kotlin
create("local") {
    buildConfigField("String", "BASE_URL", "\"http://<nueva-ip>:8080/\"")
}
```

Luego reinstalar: `./gradlew installLocalDebug`

---

## Notas de arquitectura

- La selección de `BASE_URL` por flavor es gracias al patrón `@Named("baseUrl")` implementado en `AppModule.kt` — el módulo `core/network` no accede directamente a `BuildConfig`.
- Las credenciales de autenticación (`authClientId`, `authClientSecret`, `authSubscriptionKey`) vienen de `gradle.properties` local (no commiteado) o de CI secrets.
- Para ver todos los endpoints disponibles: levantar el backend con `localhost-api-testing` skill y abrir `/swagger`.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Revisar UI/UX de la app instalada | `android-ui-ux-expert` / `android-ui-ux-reviewer` |
| Levantar el backend local para pruebas | `localhost-api-testing` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
