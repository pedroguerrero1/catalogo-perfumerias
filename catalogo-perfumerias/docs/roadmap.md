# Roadmap

Estado: ⬜ pendiente · 🟡 en curso · ✅ terminado
Etapa actual: **2**

## Fase de diseño (en el chat, sin código)

### ✅ Etapa 1 — Análisis y MVP
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
Objetivo: algo funcionando, deployado y mostrable. Nada de la segunda versión hasta tener la demo online.

**Backend**
- Perfumerías creadas por script SQL (Flyway), sin registro ni panel de superadmin (D-010).
- Login de administrador con un único rol (ROLE_ADMIN). Cada usuario pertenece a una perfumería.
- CRUD de productos, marcas, categorías y tipos de producto, siempre filtrado por la perfumería del usuario autenticado.
- Configuración de la perfumería: nombre, logo, descripción, WhatsApp, Instagram, dirección, color principal, comportamiento del botón sin stock, título y descripción SEO.
- Endpoints públicos por slug (D-004): datos de la perfumería, listado paginado con búsqueda por nombre, filtros (marca, categoría, género, rango de precio) y orden (precio, nombre), detalle de producto.
- Subida de imágenes a un servicio externo (D-003); en MySQL solo la URL y su identificador.
- Errores uniformes, Bean Validation, Swagger, tests de services y controllers, tests de aislamiento entre perfumerías.

**Catálogo público**
- Inicio de la perfumería, listado con filtros y paginación, página por categoría, detalle de producto.
- "Sin stock" visible; botón de WhatsApp con mensaje armado y comportamiento configurable.
- Renderizado en el servidor (D-001). SEO básico: título y meta description únicos, canonical, URLs estables, `noindex` en búsquedas y filtros, sitemap.xml, robots.txt, Open Graph.
- Mobile-first, imágenes con proporción uniforme, texto alternativo y carga diferida.

**Panel administrativo**
- Login; listados y formularios de productos, marcas, categorías y tipos; cambio rápido de disponibilidad y precio; subida de imagen; configuración de la perfumería.

**Infraestructura**
- docker compose solo para MySQL local.
- Deploy de una demo con 2 perfumerías de ejemplo para mostrar el multi-tenant.

## Segunda versión
- Subdominios y dominios propios por perfumería, con redirecciones 301 desde la URL con slug.
- Panel de superadmin para crear perfumerías y usuarios; más roles.
- Variantes de un mismo perfume (varios ml en una misma ficha) (D-007).
- Notas olfativas normalizadas y filtro por nota.
- JSON-LD (Product, BreadcrumbList y LocalBusiness si hay datos); horarios.
- Search Console y analítica configurables por perfumería.
- Varias imágenes por producto, productos destacados, importación masiva por CSV.
- Cantidad de stock si los clientes la piden (D-006); recuperación de contraseña; refresh tokens.
- Caché de páginas públicas; respuesta 410 para productos eliminados.
