# Roadmap

Estado: ⬜ pendiente · 🟡 en curso · ✅ terminado
Etapa actual: **1**

## Fase de diseño (en el chat, sin código)

### ⬜ Etapa 1 — Análisis y MVP
Resumen, MVP vs segunda versión, riesgos, decisiones pendientes.
Resultado: preguntas abiertas de spec.md resueltas y MVP definido abajo.

### ⬜ Etapa 2 — Arquitectura y decisiones
Arquitectura general, estructura de carpetas, decisión de frontend por SEO (D-001), JWT vs sesión, servicio de imágenes.
Resultado: entradas en decisiones.md.

### ⬜ Etapa 3 — Modelo de datos
Entidades, atributos, relaciones, cardinalidades, PK, FK, restricciones, esquema relacional y SQL inicial.
Resultado: docs/modelo-datos.md.

## Fase de construcción (Claude Code, en el repo)
Cada etapa se divide en tareas chicas. Cada tarea = una rama o un commit. Los tests se escriben junto con el código, no al final.

### ⬜ Etapa 4 — Base del backend
Proyecto Spring Boot, configuración MySQL con variables de entorno, Flyway, entidades y repositorios.

### ⬜ Etapa 5 — Lógica de negocio
Services + tests, DTOs, mappers, validaciones.

### ⬜ Etapa 6 — API REST
Controllers + tests, manejo global de errores, Swagger.

### ⬜ Etapa 7 — Seguridad y multi-tenant
Spring Security, login, roles, JWT si corresponde, aislamiento por perfumería + tests de aislamiento.

### ⬜ Etapa 8 — Base del frontend
Proyecto frontend (según D-001), arquitectura, routing, servicios de API.

### ⬜ Etapa 9 — Catálogo público
Listado, filtros, búsqueda, paginación, detalle, SEO básico por página.

### ⬜ Etapa 10 — Panel administrativo
CRUD de productos, marcas y categorías; stock; configuración de la perfumería.

### ⬜ Etapa 11 — Imágenes y WhatsApp
Servicio de imágenes, botón de WhatsApp configurable.

### ⬜ Etapa 12 — Docker
Backend + MySQL con docker compose (si se confirma que aporta valor).

### ⬜ Etapa 13 — Deploy

### ⬜ Etapa 14 — Portfolio
README profesional, capturas, descripción para GitHub y CV, preparación para entrevista.

## MVP
(se completa al terminar la Etapa 1)

## Segunda versión
(se completa al terminar la Etapa 1)
