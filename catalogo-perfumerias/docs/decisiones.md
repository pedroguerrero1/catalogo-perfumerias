# Decisiones técnicas

Registro de cada decisión importante: qué se eligió, qué alternativas había y por qué.
Sirve para que Claude no vuelva a discutir lo ya decidido y como guion para entrevistas.

Formato:

## D-XXX — Título
- **Estado**: pendiente / decidida / reemplazada por D-YYY
- **Contexto**: qué problema hay que resolver
- **Alternativas**: opciones consideradas, con ventajas y desventajas
- **Decisión**: qué se eligió
- **Por qué**: razones (en mis palabras, como lo explicaría en una entrevista)

---

## D-001 — Tecnología del frontend público y SEO
- **Estado**: pendiente (Etapa 2)
- **Contexto**: el SEO es parte del producto y los buscadores deben ver el contenido de productos y categorías sin depender de JavaScript. Una SPA con React + Vite no lo resuelve sola.
- **Alternativas**:
  1. Next.js para el catálogo público + Spring Boot como API. Mejor SEO; suma una tecnología nueva.
  2. React + Vite con prerenderizado de páginas públicas. Más simple; limitado si el catálogo cambia seguido.
  3. Spring Boot renderiza el catálogo público (Thymeleaf) y React solo para el panel admin. Aprovecha Java; menos común.
- **Decisión**: —
- **Por qué**: —

## D-002 — Autenticación: JWT o sesión
- **Estado**: pendiente (Etapa 2)

## D-003 — Servicio de imágenes
- **Estado**: pendiente (Etapa 2)

## D-004 — Cómo se identifica la perfumería en el catálogo público (slug, subdominio o dominio propio)
- **Estado**: pendiente (Etapa 1)
