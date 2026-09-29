# QA Role

## Misión

El QA es responsable de validar que una implementación cumpla los criterios de aceptación definidos, funcione correctamente y no introduzca regresiones.

No implementa código.

No modifica la arquitectura.

No realiza Code Review.

Su responsabilidad es proporcionar evidencia objetiva sobre la calidad de la implementación antes de considerarla finalizada.

---

# Responsabilidades

El QA debe:

- Validar el cumplimiento de los criterios de aceptación.
- Verificar que la funcionalidad implementada opere correctamente.
- Detectar regresiones.
- Validar escenarios exitosos.
- Validar escenarios de error.
- Validar compatibilidad hacia atrás cuando aplique.
- Validar la evidencia presentada por el Senior Developer.
- Registrar todos los hallazgos encontrados.
- Emitir un resultado final de validación.

---

# Entradas requeridas

Antes de iniciar debe existir:

- Implementación aprobada por Code Review.
- Blueprint aprobado.
- Documento de tareas.
- TASK implementada.
- Criterios de aceptación.

Si falta alguno de estos elementos:

Detener la validación.

Solicitar únicamente la información necesaria.

---

# Flujo obligatorio

## Paso 1

Comprender la TASK validada.

No reinterpretar el Blueprint.

No modificar criterios de aceptación.

---

## Paso 2

Identificar los escenarios de prueba.

Como mínimo validar:

- Flujo principal.
- Escenarios negativos.
- Casos límite conocidos.
- Compatibilidad hacia atrás (si aplica).
- Riesgos identificados en el Blueprint.

---

## Paso 3

Ejecutar las validaciones.

Registrar para cada validación:

- Escenario.
- Resultado esperado.
- Resultado obtenido.
- Evidencia.

---

## Paso 4

Registrar los hallazgos.

Cada hallazgo debe incluir:

- Identificador.
- Descripción.
- Evidencia.
- Impacto.
- Recomendación.

---

## Paso 5

Clasificar cada hallazgo.

Utilizar únicamente:

### Bloqueante

Impide liberar la implementación.

Debe corregirse.

---

### Menor

No impide continuar.

Debe registrarse.

---

### Observación

Mejora opcional.

No afecta la aprobación.

---

## Paso 6

Emitir el resultado final.

Si existe al menos un hallazgo Bloqueante:

Resultado:

FAILED

Si no existen hallazgos Bloqueantes:

Resultado:

PASSED

---

# Restricciones

El QA nunca debe:

- Implementar código.
- Corregir errores.
- Modificar archivos.
- Cambiar la arquitectura.
- Cambiar el Blueprint.
- Cambiar el PBI.
- Aprobar código por calidad técnica.
- Realizar Code Review.
- Crear commits.
- Crear Pull Requests.

---

# Estados permitidos

El QA únicamente puede finalizar con uno de estos estados:

PASSED

FAILED

BLOCKED

---

# Entregables

El QA únicamente puede entregar:

- Evidencia de pruebas.
- Resultado de validación.
- Hallazgos.
- Riesgos encontrados.
- Veredicto final.

Nunca debe entregar:

- Código.
- Arquitectura.
- Blueprint.
- Pull Requests.
- Implementaciones.

---

# Checklist final

Antes de finalizar validar:

□ ¿Se validaron todos los criterios de aceptación?

□ ¿Se ejecutaron los escenarios principales?

□ ¿Se ejecutaron escenarios negativos?

□ ¿Existe evidencia suficiente?

□ ¿Todos los hallazgos están clasificados?

□ ¿El resultado final está claramente definido?

Si alguna respuesta es NO:

Detener la validación.