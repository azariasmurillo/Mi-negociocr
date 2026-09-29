# Git Policy

## Objetivo

Este documento define la política oficial para cualquier operación relacionada con Git dentro del AI Engineering Framework.

Todos los Agent Skills deben respetar esta política.

---

# Principio

El agente asiste al desarrollador.

Nunca toma control del repositorio.

El desarrollador siempre tiene la última decisión.

---

# Operaciones protegidas

Las siguientes operaciones requieren autorización explícita del desarrollador.

- git add
- git commit
- git push
- git pull
- git merge
- git rebase
- git tag
- git stash
- git reset
- git revert
- Crear Pull Request
- Publicar Pull Request
- Aprobar Pull Request
- Merge Pull Request
- Eliminar ramas
- Crear Releases

---

# Regla principal

Nunca ejecutes una operación Git automáticamente.

Incluso si el trabajo parece terminado.

---

# Autorización válida

Solamente son válidas instrucciones explícitas como:

- Haz el commit.
- Crea el commit.
- Haz push.
- Crea el Pull Request.
- Publica el Pull Request.
- Haz merge.
- Publica el Release.

Si la autorización no es completamente explícita:

No ejecutar ninguna operación.

---

# Autorización NO válida

Las siguientes frases NO autorizan operaciones Git.

- Listo.
- Excelente.
- Perfecto.
- Continúa.
- Correcto.
- Ya quedó.
- Está bien.
- Terminamos.
- Muy bien.
- Me gusta.

---

# Antes de cualquier operación Git

Siempre verificar:

□ El código fue implementado.

□ El trabajo solicitado fue completado.

□ Se explicaron los cambios.

□ Se mostraron los archivos modificados.

□ El desarrollador revisó el trabajo.

□ Existe autorización explícita.

Si cualquiera de estas respuestas es NO:

Detenerse.

---

# Flujo oficial

Implementar

↓

Validar

↓

Explicar

↓

Esperar revisión humana

↓

Esperar autorización

↓

Git

---

# Pull Requests

Nunca crear un Pull Request automáticamente.

Antes debe existir:

- Implementación terminada.
- Validación.
- Revisión humana.
- Aprobación explícita.

---

# Commits

Nunca crear commits automáticamente.

Aunque:

- el código compile
- los tests pasen
- el usuario diga "listo"

Siempre esperar autorización.

---

# Operaciones vía API de plataforma

Esta sección extiende la política a operaciones realizadas mediante APIs de plataformas de código (por ejemplo GitHub) o mediante el agente.

El principio es el mismo:

El agente asiste al desarrollador.

Nunca toma acciones sobre repositorios o colaboradores sin autorización explícita.

## Operaciones protegidas vía API

Las siguientes operaciones requieren autorización explícita del desarrollador.

- Crear Pull Request
- Publicar Pull Request
- Crear Review
- Publicar Review
- Aprobar Pull Request
- Solicitar cambios en un Pull Request
- Crear comentarios en un Pull Request
- Editar comentarios existentes
- Publicar comentarios automáticos
- Crear Issues
- Crear Releases
- Publicar Releases

## Regla principal

Nunca ejecutes una operación de API de plataforma automáticamente.

Incluso si el trabajo parece terminado.

Incluso si el reporte o el artefacto ya están generados.

## Antes de cualquier operación vía API

Siempre verificar:

□ El artefacto a publicar fue revisado por el desarrollador.

□ El desarrollador conoce qué se va a publicar y dónde.

□ Existe autorización explícita para esta operación específica.

□ La operación es reversible o el desarrollador acepta que no lo es.

Si cualquiera de estas respuestas es NO:

Detenerse.

## Autorización válida

Solamente son válidas instrucciones explícitas como:

- Publica el review.
- Comenta el PR.
- Crea el Issue.
- Publica el Release.
- Aprueba el PR.

Si la autorización no es completamente explícita:

No ejecutar ninguna operación.

---

# Push

Nunca realizar push automáticamente.

Siempre esperar autorización.

---

# Merge

Nunca realizar merge automáticamente.

Siempre esperar autorización.

---

# Filosofía

La automatización nunca debe reemplazar el control del desarrollador.

El repositorio pertenece al equipo.

Las decisiones sobre Git pertenecen al desarrollador.
