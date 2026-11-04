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
- **Estado**: decidida (Etapa 2)
- **Contexto**: el SEO es parte del producto y los buscadores deben ver el contenido de productos y categorías sin depender de JavaScript. Una SPA con React + Vite no lo resuelve sola.
- **Alternativas**:
  1. Next.js para el catálogo público + Spring Boot como API. Mejor SEO; suma una tecnología nueva.
  2. React + Vite con prerenderizado de páginas públicas. Más simple; limitado si el catálogo cambia seguido.
  3. Spring Boot renderiza el catálogo público (Thymeleaf) y React solo para el panel admin. Aprovecha Java; menos común.
- **Decisión**: opción 1. Next.js (App Router) con JavaScript, una sola app en `frontend/` para catálogo público y panel admin.
  - Catálogo público: Server Components que llaman a la API de Spring (URL por variable de entorno) y devuelven HTML completo. Título, descripción, sitemap y robots con las funciones del framework. Render dinámico, sin ISR ni Server Actions en el MVP.
  - Panel admin: componentes cliente (`"use client"`) que llaman a la API desde el navegador con el token.
  - Next no tiene lógica de negocio: solo muestra datos y arma URLs.
- **Por qué**: el renderizado en el servidor resuelve el SEO sin trucos y es React, que iba a usar igual. Una sola tecnología de frontend. La opción 2 no sirve porque el catálogo cambia seguido y es multi-tenant (habría que reconstruir en cada cambio); la 3 deja dos frontends distintos.

## D-002 — Autenticación: JWT o sesión
- **Estado**: decidida (Etapa 2)
- **Contexto**: el panel admin necesita login, y frontend y API se deployan en dominios distintos.
- **Alternativas**:
  1. JWT en header `Authorization: Bearer`. Sin estado en el servidor y simple entre dominios; no se puede revocar antes de que venza.
  2. Sesión con cookie `JSESSIONID`. Revocación inmediata; entre dominios exige CORS con credenciales, `SameSite=None` y CSRF, y guarda estado en el servidor.
- **Decisión**: JWT firmado con HS256, con la librería jjwt y un filtro propio. Vence a las 8 h, sin refresh token en el MVP. El frontend lo guarda en `localStorage`.
- **Cómo funciona**:
  1. `POST /api/auth/login` con email y contraseña; Spring Security compara contra el hash BCrypt.
  2. Si es correcta, se genera un JWT con `sub` = id de usuario y vencimiento; la clave secreta sale de una variable de entorno.
  3. El frontend manda el token en cada request: `Authorization: Bearer <token>`.
  4. `JwtFiltro` verifica firma y vencimiento, carga el usuario desde la base (rechaza usuarios desactivados) y lo registra en el `SecurityContext`.
  5. `SecurityConfig`: `/api/auth/login` y `/api/publico/**` abiertos; `/api/admin/**` exige `ROLE_ADMIN`; sin token o con token inválido → 401.
  6. Los services obtienen la perfumería del usuario autenticado (`UsuarioActual.getPerfumeriaId()`), nunca de la URL ni del body.
  7. El layout de `/admin` redirige al login si no hay token. Es solo comodidad: la seguridad está en el backend.
- **Por qué**: con dominios distintos un header es mucho más simple que cookies entre sitios. Cargar el usuario en cada request mitiga la falta de revocación. jjwt con filtro propio se puede explicar línea por línea, a diferencia de `oauth2-resource-server`, que resuelve casi todo por configuración. El riesgo de `localStorage` es XSS: React escapa el contenido y no se usa `dangerouslySetInnerHTML`. Una cookie httpOnly gestionada por Next queda para la segunda versión.

## D-003 — Servicio de imágenes
- **Estado**: decidida (Etapa 2)
- **Contexto**: MySQL guarda solo la URL; las imágenes tienen que verse uniformes y optimizadas aunque las originales tengan distintas proporciones.
- **Alternativas**:
  1. Cloudinary. Transforma desde la URL (recorte, tamaño, WebP/AVIF, calidad) con CDN; plan gratuito suficiente. Es un servicio externo más.
  2. Firebase Storage. Gratis al principio; no transforma y trae el ecosistema Firebase para una sola función.
  3. AWS S3. Estándar; no transforma, e IAM y facturación son complejos para este proyecto.
  4. Disco del servidor. Sin servicios externos; en hostings gratuitos el disco se borra en cada deploy.
- **Decisión**: Cloudinary con el SDK oficial (`com.cloudinary:cloudinary-http5`) y subida a través del backend:
  1. El admin sube el archivo a `POST /api/admin/productos/{id}/imagen`.
  2. El backend valida tipo (jpg, png, webp) y tamaño (máximo 5 MB) y lo sube con la clave secreta (variable de entorno).
  3. Guarda `imagen_url` e `imagen_id`; si había una imagen anterior, la borra de Cloudinary.
  4. El frontend pide la imagen con transformación en la URL, por ejemplo `c_fill,ar_1:1,w_600,f_auto,q_auto`.
- **Por qué**: las transformaciones por URL resuelven la proporción uniforme y la optimización sin código propio. Subir a través del backend mantiene la clave secreta en el servidor. La subida directa desde el navegador con firma queda para más adelante.

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

## D-012 — Dependencias aprobadas
- **Estado**: decidida (Etapa 2)
- **Contexto**: no se agregan dependencias sin aprobación explícita. Este es el registro de las aprobadas fuera del stack base.
- **Decisión**:
  - `io.jsonwebtoken:jjwt` — generar y validar JWT (D-002).
  - `com.cloudinary:cloudinary-http5` — subir y borrar imágenes en Cloudinary (D-003).
- **Por qué**: cada una resuelve una decisión ya tomada y evita implementar a mano criptografía o un cliente HTTP.

## D-013 — Aislamiento de las relaciones entre tablas de una perfumería
- **Estado**: decidida (Etapa 3)
- **Contexto**: un producto referencia marca, categoría y tipo. Una FK simple (`marca_id → marcas.id`) no impide que apunte a una marca de otra perfumería.
- **Alternativas**:
  1. Validación en el service: buscar cada relación con `findByIdAndPerfumeriaId` y responder 404 si no pertenece a la perfumería del usuario. Simple de mapear en JPA; depende de que el service lo haga siempre.
  2. FK compuesta `(marca_id, perfumeria_id) → marcas(id, perfumeria_id)`. La base lo garantiza; en JPA obliga a mapear `perfumeria_id` dos veces con `insertable = false, updatable = false`, confuso de mantener.
- **Decisión**: opción 1, cubierta por los tests de aislamiento de cada service.
- **Por qué**: la regla multi-tenant ya exige que todo service filtre por perfumería; la validación es la misma consulta que se usa en el resto del CRUD. La FK compuesta sumaría una segunda barrera con un mapeo difícil de explicar.

## D-014 — Slugs de producto y categoría
- **Estado**: decidida (Etapa 3)
- **Contexto**: las URLs públicas (`/{slug}/productos/{producto}`, `/{slug}/categorias/{categoria}`) tienen que ser claras y estables para el SEO.
- **Alternativas**:
  1. Slug fijo: se genera al crear y no cambia al editar. URLs estables; puede quedar desactualizado si cambia mucho el nombre.
  2. Slug que se regenera al editar el nombre. Siempre coincide con el nombre; cambia la URL y se pierde la página indexada sin redirecciones 301.
- **Decisión**: opción 1. El backend lo genera desde el nombre (ej. `dior-sauvage-edp-100ml`), único por perfumería (`UNIQUE (perfumeria_id, slug)`), con sufijo `-2`, `-3`… si ya existe.
- **Por qué**: una URL que cambia rompe enlaces compartidos y resultados de Google; las redirecciones 301 quedan para la segunda versión.

## D-015 — Convenciones de la base de datos
- **Estado**: decidida (Etapa 3)
- **Decisión**:
  - Enums (género, rol, comportamiento sin stock) como `VARCHAR` + `CHECK`, mapeados con `@Enumerated(EnumType.STRING)`.
  - Precio en `DECIMAL(12,2)`, sin columna de moneda.
  - Collation `utf8mb4_0900_ai_ci`.
  - Todas las FK con `ON DELETE RESTRICT`.
  - Email de usuario único en todo el sistema.
  - Comportamiento sin stock configurable por perfumería: `DESHABILITAR_BOTON` (default) o `MOSTRAR_BOTON`.
- **Por qué**:
  - El `ENUM` nativo de MySQL exige `ALTER TABLE` para sumar valores, y `ORDINAL` se rompe si cambia el orden del enum en Java.
  - `DOUBLE` redondea mal el dinero.
  - `ai_ci` hace que la búsqueda y la unicidad no distingan tildes ni mayúsculas ("cafe" = "Café", "Dior" = "dior").
  - `RESTRICT` es la última barrera si el service no detecta el 409 de D-009.
  - El login recibe solo email y contraseña, así que el email tiene que identificar a un único usuario.
  - Por defecto el botón se deshabilita sin stock para concentrar las consultas en lo que se puede vender; queda configurable porque otra perfumería puede preferir recibirlas igual.
