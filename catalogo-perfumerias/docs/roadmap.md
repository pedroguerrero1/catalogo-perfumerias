# Roadmap

Estado: ⬜ pendiente · 🟡 en curso · ✅ terminado
Etapa actual: **4**

## Fase de diseño (en el chat, sin código)

### ✅ Etapa 1 — Análisis y MVP
Resumen, MVP vs segunda versión, riesgos, decisiones pendientes.
Resultado: preguntas abiertas de spec.md resueltas y MVP definido abajo.

### ✅ Etapa 2 — Arquitectura y decisiones
Arquitectura general, estructura de carpetas, decisión de frontend por SEO (D-001), JWT vs sesión, servicio de imágenes.
Resultado: entradas en decisiones.md.

### ✅ Etapa 3 — Modelo de datos
Entidades, atributos, relaciones, cardinalidades, PK, FK, restricciones, esquema relacional y SQL inicial.
Resultado: docs/modelo-datos.md.

## Fase de construcción (Claude Code, en el repo)
Cada etapa se divide en tareas chicas. Cada tarea = una rama o un commit. Los tests se escriben junto con el código, no al final.

### ✅ Cambio de alcance — Carrito, pedidos y pagos (antes de la Etapa 4)
Primer cliente real: carrito y pago online con Mercado Pago Checkout Pro, y migración desde su sitio actual.
Resultado: módulo en spec.md, tablas y flujo en modelo-datos.md, D-016 a D-018 (D-006 reemplazada, D-009 y D-012 actualizadas), reglas de pagos en CLAUDE.md.

### ⬜ Etapa 4 — Base del backend
Proyecto Spring Boot, configuración MySQL con variables de entorno, Flyway, entidades y repositorios.
La V1 ya usa `productos.stock` (cantidad, D-017). Las tablas de pedidos y pagos se crean en la Etapa 12.

### ⬜ Etapa 5 — Lógica de negocio
Services + tests, DTOs, mappers, validaciones.

### ⬜ Etapa 6 — API REST
Controllers + tests, manejo global de errores, Swagger.

### ⬜ Etapa 7 — Seguridad y multi-tenant
Spring Security, login, roles, JWT si corresponde, aislamiento por perfumería + tests de aislamiento.

### ⬜ Etapa 8 — Base del frontend
Proyecto Next.js (D-001), arquitectura, routing, servicios de API.

### ⬜ Etapa 9 — Catálogo público
Listado, filtros, búsqueda, paginación, detalle, SEO básico por página.

### ⬜ Etapa 10 — Panel administrativo
CRUD de productos, marcas y categorías; stock; configuración de la perfumería.

### ⬜ Etapa 11 — Imágenes y WhatsApp
Servicio de imágenes, botón de WhatsApp configurable.

### ⬜ Etapa 12 — Carrito y pedidos
Migración de Flyway con pedidos, items, pagos y credenciales. Carrito en el navegador con "Enviar pedido por WhatsApp". Creación de pedidos con precios y stock de la base, vencimiento a los 30 minutos (`@Scheduled`), página pública del pedido. Panel: listado, detalle y cambios de estado. Tests de cálculo de totales, transiciones de estado y aislamiento.

### ⬜ Etapa 13 — Pagos con Mercado Pago
Credenciales cifradas por perfumería y su pantalla en el panel. Preferencia de Checkout Pro, webhook con validación de firma, consulta del pago a la API, idempotencia y descuento de stock. Todo probado en modo prueba (sandbox) con usuarios de prueba de Mercado Pago. Tests del webhook: duplicado, monto distinto, sin stock, pago tardío, firma inválida, perfumería equivocada.

### ⬜ Etapa 14 — Docker
Backend + MySQL con docker compose (si se confirma que aporta valor).

### ⬜ Etapa 15 — Deploy
El webhook necesita una URL pública con HTTPS: los pagos se terminan de probar acá, con el backend deployado.

### ⬜ Etapa 16 — Migración del primer cliente
Script único de importación desde Firebase (productos, marcas, categorías, imágenes; cómo leer Firebase se decide en esta etapa). Conteo de stock con el cliente (D-017). Credenciales de Mercado Pago en producción y una compra real de punta a punta. Recién después, el cliente deja el sitio actual. Antes de empezar: resolver la pregunta abierta del dominio propio (spec.md).

### ⬜ Etapa 17 — Portfolio
README profesional, capturas, descripción para GitHub y CV, preparación para entrevista.

**Por qué este orden**: carrito y pagos van después del catálogo y del panel porque se apoyan en ellos (productos, stock, configuración, login); hacerlos antes obligaría a construir sobre algo que todavía no existe. Van antes del deploy para salir con el flujo de compra completo. La migración va después del deploy porque necesita el sitio online, el webhook público y Mercado Pago en producción.

## MVP
Objetivo: algo funcionando, deployado y usado por el primer cliente. Nada de la segunda versión hasta que el cliente esté migrado.

**Backend**
- Perfumerías creadas por script SQL (Flyway), sin registro ni panel de superadmin (D-010).
- Login de administrador con un único rol (ROLE_ADMIN). Cada usuario pertenece a una perfumería.
- CRUD de productos, marcas, categorías y tipos de producto, siempre filtrado por la perfumería del usuario autenticado.
- Configuración de la perfumería: nombre, logo, descripción, WhatsApp, Instagram, dirección, color principal, comportamiento del botón sin stock, título y descripción SEO.
- Endpoints públicos por slug (D-004): datos de la perfumería, listado paginado con búsqueda por nombre, filtros (marca, categoría, género, rango de precio) y orden (precio, nombre), detalle de producto.
- Subida de imágenes a un servicio externo (D-003); en MySQL solo la URL y su identificador.
- Errores uniformes, Bean Validation, Swagger, tests de services y controllers, tests de aislamiento entre perfumerías.
- Stock por cantidad (D-017). Pedidos sin cuenta de cliente, pago con Mercado Pago Checkout Pro, webhook idempotente y descuento de stock al confirmar el pago (D-016). Credenciales de Mercado Pago cifradas por perfumería (D-018).

**Catálogo público**
- Inicio de la perfumería, listado con filtros y paginación, página por categoría, detalle de producto.
- "Sin stock" visible; botón de WhatsApp con mensaje armado y comportamiento configurable.
- Carrito con dos salidas: pagar con Mercado Pago o enviar el pedido por WhatsApp. Página del pedido por código.
- Renderizado en el servidor (D-001). SEO básico: título y meta description únicos, canonical, URLs estables, `noindex` en búsquedas y filtros, sitemap.xml, robots.txt, Open Graph.
- Mobile-first, imágenes con proporción uniforme, texto alternativo y carga diferida.

**Panel administrativo**
- Login; listados y formularios de productos, marcas, categorías y tipos; cambio rápido de stock y precio; subida de imagen; configuración de la perfumería.
- Pedidos: listado, detalle y cambios de estado. Configuración de credenciales de Mercado Pago.

**Infraestructura**
- docker compose solo para MySQL local.
- Deploy con el primer cliente en producción y una perfumería de ejemplo para mostrar el multi-tenant.

## Segunda versión
- Subdominios y dominios propios por perfumería, con redirecciones 301 desde la URL con slug.
- Panel de superadmin para crear perfumerías y usuarios; más roles.
- Variantes de un mismo perfume (varios ml en una misma ficha) (D-007).
- Notas olfativas normalizadas y filtro por nota.
- JSON-LD (Product, BreadcrumbList y LocalBusiness si hay datos); horarios.
- Search Console y analítica configurables por perfumería.
- Varias imágenes por producto, productos destacados, importación masiva por CSV.
- Recuperación de contraseña; refresh tokens.
- Caché de páginas públicas; respuesta 410 para productos eliminados.
- Conexión con Mercado Pago por OAuth (D-018); emails de confirmación de pedido; pedidos de WhatsApp guardados en el panel (D-016); costo de envío calculado; cuentas de cliente.
