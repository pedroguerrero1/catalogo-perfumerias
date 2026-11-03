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
- **Recomendación (Etapa 1)**: opción 1, Next.js (App Router) con JavaScript, una sola app para catálogo público y panel admin. Uso simple: páginas públicas renderizadas en el servidor que llaman a la API, render dinámico (sin ISR ni Server Actions en el MVP), admin como componentes cliente. Se descarta la opción 2 porque el catálogo cambia seguido y es multi-tenant (habría que reconstruir en cada cambio), y la 3 porque deja dos tecnologías de frontend.
- **Decisión**: —
- **Por qué**: —

## D-002 — Autenticación: JWT o sesión
- **Estado**: pendiente (Etapa 2)
- **Recomendación (Etapa 1)**: JWT (Bearer), porque frontend y API van en dominios distintos y las cookies de sesión cross-site complican CORS y SameSite.

## D-003 — Servicio de imágenes
- **Estado**: pendiente (Etapa 2)
- **Recomendación (Etapa 1)**: Cloudinary, porque recorta, redimensiona y convierte a WebP desde la URL (proporción uniforme y optimización) y su plan gratuito alcanza.

## D-004 — Cómo se identifica la perfumería en el catálogo público
- **Estado**: decidida (Etapa 1)
- **Contexto**: el catálogo público atiende a varias perfumerías y cada request tiene que saber de cuál se trata.
- **Alternativas**:
  1. Slug en la ruta (`/{slug}/...`). Simple, un solo dominio y un solo certificado; el valor SEO queda en el dominio de la plataforma.
  2. Subdominio (`{slug}.dominio.com`). Más prolijo; requiere DNS comodín y resolver la perfumería por el host.
  3. Dominio propio por cliente. Lo mejor para el cliente; requiere configurar DNS y SSL por cliente.
- **Decisión**: slug en la ruta en el MVP: `/{slug}/productos/{producto}`, `/{slug}/categorias/{categoria}`. Subdominio y dominio propio en la segunda versión, con redirecciones 301.
- **Por qué**: es lo más simple de construir, deployar y explicar, y alcanza para la demo. La ruta después del slug se mantiene igual en todos los esquemas para que migrar a dominio propio sea solo una redirección. Se reservan slugs como `admin` y `api`.

## D-005 — Idioma del código y la documentación
- **Estado**: decidida (Etapa 1)
- **Contexto**: hay que elegir un idioma único para código, base de datos, endpoints, docs y commits.
- **Alternativas**:
  1. Todo en inglés. Estándar en la industria; mezcla idiomas con el dominio del negocio.
  2. Todo en español, salvo términos del framework. Coincide con el dominio y con los clientes.
- **Decisión**: español en explicaciones, docs, commits, clases, métodos, variables, tablas, columnas y endpoints. En inglés solo lo propio del framework y las convenciones de Java/Spring: sufijos de capa (ProductoController, ProductoService, ProductoRepository, ProductoDTO), prefijos get/set/is y métodos heredados o derivados (findById, save, findByPerfumeriaId...).
- **Por qué**: el dominio (perfumería, notas, decants) se expresa mejor en español y el código queda alineado con la spec. Las consultas derivadas de Spring Data exigen el prefijo `findBy`, así que un método como `buscarPorCategoria()` va con `@Query` o en el service.
- **Ejemplos**: entidad Producto, tabla productos, columna perfumeria_id, endpoint /api/productos, commit "feat: agrega entidad Producto".

## D-006 — Modelo de stock
- **Estado**: decidida (Etapa 1)
- **Contexto**: el admin tiene que poder indicar si un producto está disponible, y nunca debe mostrarse como disponible algo que no lo está.
- **Alternativas**:
  1. Cantidad entera. Más información; si no se actualiza en cada venta, muestra como disponible algo agotado.
  2. Booleano `disponible`. Simple y fácil de mantener al día; no indica cuántas unidades quedan.
- **Decisión**: booleano `disponible` más `activo` (visible u oculto). La cantidad queda para la segunda versión si un cliente la pide.
- **Por qué**: las perfumerías venden por WhatsApp y no suelen llevar un conteo; un sí/no es más honesto y no se desactualiza.

## D-007 — Mismo perfume en varios tamaños
- **Estado**: decidida (Etapa 1)
- **Contexto**: un perfume puede venderse en varios ml (por ejemplo, frasco y decants de 5 y 10 ml).
- **Alternativas**:
  1. Cada tamaño es un producto distinto. Simple; duplica descripción y notas.
  2. Producto con variantes. Ficha única; suma una entidad y complejidad en filtros, precios y stock.
- **Decisión**: en el MVP cada tamaño es un producto con su propio slug. Variantes en la segunda versión.
- **Por qué**: evita una entidad extra y su lógica mientras se valida el producto; la duplicación es aceptable al principio.

## D-008 — Categoría vs. tipo de producto
- **Estado**: decidida (Etapa 1)
- **Contexto**: "Decants" y "Body Splash" aparecen tanto en categorías como en tipos, lo que genera confusión al cargar datos.
- **Decisión**: la categoría es cómo navega el cliente (Perfumes, Decants, Body Splash). El tipo es la presentación o concentración (EDP, EDT, Extrait de parfum, Body Splash, Decant). Ambas son tablas por perfumería, con valores iniciales cargados al crear la perfumería. El género es un enum (Hombre, Mujer, Unisex).
- **Por qué**: separa navegación de característica técnica; categorías y tipos se agregan sin tocar código, como pide la spec.

## D-009 — Eliminación de productos
- **Estado**: decidida (Etapa 1)
- **Contexto**: la spec pide no borrar productos por quedarse sin stock y un comportamiento coherente para usuarios y buscadores.
- **Alternativas**:
  1. Borrado lógico (marca de eliminado). Recuperable; obliga a filtrar en todas las consultas.
  2. Borrado físico cuando el admin elimina, más `activo` para ocultar. Simple.
- **Decisión**: borrado físico solo cuando el admin elimina; `activo = false` para ocultar; sin stock sigue visible con "Sin stock". Público: inactivo o eliminado → 404; agotado → 200. Eliminar una marca o categoría con productos → 409.
- **Por qué**: `activo` ya cubre el caso de ocultar sin perder datos, así que el borrado lógico sería una segunda forma de hacer lo mismo.

## D-010 — Alta de perfumerías
- **Estado**: decidida (Etapa 1)
- **Decisión**: en el MVP las perfumerías y sus usuarios admin se crean con scripts SQL (Flyway). Panel de superadmin en la segunda versión.
- **Por qué**: para la demo alcanza con 2 perfumerías; un panel de superadmin agrega rol, pantallas y permisos sin aportar al MVP.

## D-011 — Estructura del repositorio
- **Estado**: decidida (Etapa 1)
- **Alternativas**: monorepo o un repositorio para backend y otro para frontend.
- **Decisión**: monorepo con `backend/`, `frontend/` y `docs/`.
- **Por qué**: un solo historial muestra la evolución completa del proyecto, y la documentación queda junto a ambos.
