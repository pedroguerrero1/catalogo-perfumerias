# Especificación del producto

## Objetivos
1. Base de producto reutilizable y adaptable para distintas perfumerías (clientes reales).
2. Proyecto de portfolio técnicamente sólido: frontend, backend, API REST, base de datos, autenticación y panel administrativo, que pueda explicar en entrevistas.

No es una página visualmente bonita: es una aplicación real.

**Primer cliente real**: una perfumería con un catálogo en Firebase y venta por WhatsApp. Pasa a esta plataforma con carrito y pago online con Mercado Pago. Su sitio actual sigue funcionando hasta la migración (ver docs/roadmap.md).

## Usuarios
- **Cliente final**: navega el catálogo público y compra sin iniciar sesión.
- **Administrador de perfumería**: gestiona los datos de SU perfumería desde un panel privado.
- (Futuro) otros roles. La arquitectura debe permitir agregarlos.

## Multi-tenant
- Una misma aplicación maneja varias perfumerías.
- Cada perfumería tiene sus propios productos, categorías, marcas, precios y configuración.
- Un usuario pertenece a una perfumería y solo puede ver y modificar los datos de esa perfumería.
- El aislamiento se contempla en modelo de datos, entidades, repositorios, servicios, seguridad y endpoints. Nunca solo en el frontend.

## Perfumería (configuración)
Nombre, logo, colores principales, descripción, WhatsApp, Instagram, dirección, datos de contacto y configuración general del catálogo (ej: comportamiento del botón de WhatsApp sin stock).

## Catálogo público
- Página principal con información de la perfumería y redes sociales
- Listado de productos con búsqueda, filtros, ordenamiento y paginación
- Navegación por categorías
- Detalle de producto
- Botón "Consultar por WhatsApp"
- Botón "Agregar al carrito" y carrito (ver "Carrito, pedidos y pagos")
- Diseño responsive, pensado primero para celulares

## Productos
Campos mínimos: nombre, marca, categoría, género, mililitros, tipo de producto, precio, imagen, descripción, notas de salida, notas de corazón, notas de fondo, stock, activo/inactivo.

- Categorías iniciales: Perfumes, Decants, Body Splash. Deben poder agregarse nuevas sin tocar código.
- Géneros: Hombre, Mujer, Unisex.
- Tipos: EDP, EDT, Extrait de parfum, Body Splash, Decant. Deben poder agregarse nuevos valores.

## Stock
- El stock es una cantidad de unidades por producto (D-017). El admin la carga y la corrige.
- Sin stock (`0`): se muestra claramente "Sin stock", nunca como disponible, y no se puede agregar al carrito.
- El stock se descuenta solo cuando Mercado Pago confirma el pago. Los pedidos por WhatsApp no lo tocan.
- El comportamiento del botón de WhatsApp sin stock es configurable.
- No se eliminan físicamente productos por quedarse sin stock (estado activo/inactivo + stock).

## Carrito, pedidos y pagos
Decisiones en D-016, D-017 y D-018. Modelo y flujo en docs/modelo-datos.md.

**Carrito**
- Vive en el navegador: productos y cantidades. Se conserva al recargar la página.
- No se puede pedir más cantidad que el stock disponible.
- Dos formas de terminar la compra:
  - **Pagar con Mercado Pago**: solo si la perfumería tiene Mercado Pago configurado y activo.
  - **Enviar pedido por WhatsApp**: arma un mensaje con productos, cantidades y total. No crea un pedido ni descuenta stock.

**Checkout con Mercado Pago (Checkout Pro)**
- El comprador completa nombre, email, teléfono y forma de entrega (retiro o envío; si es envío, dirección). No necesita cuenta.
- El backend crea el pedido con precios y stock de la base y lo redirige a Mercado Pago. El sitio nunca maneja datos de tarjeta.
- No se ofrecen medios de pago en efectivo.
- El pedido vence a los 30 minutos sin pago.
- El envío no se cobra en el sitio: se coordina con el cliente por WhatsApp. Después del pago se muestra un botón "Coordinar entrega por WhatsApp" con el código del pedido.
- El pedido queda pagado solo cuando el webhook de Mercado Pago lo confirma. La página de retorno solo muestra el estado.
- Página pública del pedido por código: estado, productos y total. Nunca datos personales del comprador.

**Estados del pedido**: pendiente de pago, pagado, requiere revisión (pago aprobado sin stock suficiente o con monto distinto), entregado, vencido, cancelado, reembolsado.

**Panel administrativo: pedidos**
- Listado filtrable por estado y fecha; detalle con comprador, entrega, productos y pagos.
- Marcar como entregado, cancelar un pedido pendiente, resolver pedidos en revisión y marcar como reembolsado (el reembolso se hace en Mercado Pago).
- Configurar las credenciales de Mercado Pago de la perfumería (access token, secreto del webhook, modo prueba o producción, activo). Nunca se vuelven a mostrar después de guardarlas.

## Panel administrativo
- **Productos**: crear, editar, eliminar, activar/desactivar, modificar stock y precio, subir/cambiar imagen, asignar marca, categoría y género. Un producto con pedidos no se elimina: se desactiva.
- **Marcas**: crear, editar, eliminar, listar.
- **Categorías**: crear, editar, eliminar, listar.
- **Pedidos y Mercado Pago**: ver "Carrito, pedidos y pagos".
- **Configuración de la perfumería**: nombre, logo, WhatsApp, Instagram, dirección, descripción, colores.

## Autenticación y autorización
- Spring Security, login, usuarios administradores, roles (ROLE_ADMIN, extensible).
- Contraseñas con hashing seguro.
- Explicar antes de implementar: cómo funciona la autenticación, qué pasa al iniciar sesión, cómo se protege una ruta y un endpoint, cómo se determina qué perfumería administra un usuario. Si se usa JWT, explicar primero cómo funciona.

## API REST
- Recursos bien organizados, HTTP status codes correctos, errores claros y uniformes.
- Capas: Controller → Service → Repository → Base de datos, con DTOs de creación, actualización y respuesta donde tenga sentido.
- Validaciones con Bean Validation (obligatorios, precio no negativo, mililitros válidos, email válido, longitudes máximas).
- Manejo global de excepciones con @ControllerAdvice.
- Documentación con Swagger/OpenAPI: endpoints, parámetros, request body, responses, errores y autenticación.

## Búsqueda y filtros
- Buscar por nombre; filtrar por marca, género, categoría y precio; ordenar por precio y alfabéticamente.
- Analizar qué se resuelve en backend y qué en frontend.
- Paginación y filtros por query parameters: no traer cientos o miles de productos al navegador.

## WhatsApp
- Mensaje generado automáticamente con datos del producto. Ejemplo: "Hola, quisiera consultar por [producto], [ml], precio $XXX."
- Pedido desde el carrito: lista de productos con ml, cantidad y precio, más el total.
- Coordinar entrega después de pagar: mensaje con el código del pedido.
- El número sale de la configuración de la perfumería, nunca hardcodeado en el frontend.

## Migración del primer cliente
- Importar productos, marcas, categorías e imágenes desde el Firebase actual con un script único (Firebase solo como origen; nunca como base de datos principal).
- Stock: no disponible → 0; disponible → cantidad del conteo que haga el cliente, o 1 si no hay dato (D-017).
- Probar el pago completo en modo prueba antes de activar el modo producción.
- El sitio actual sigue funcionando hasta que el nuevo esté probado en producción.

## Imágenes
- No se guardan imágenes en MySQL: solo URL/referencia.
- Analizar Cloudinary, Firebase Storage, S3 u otro, y elegir uno adecuado para un proyecto de portfolio explicando por qué.
- Tratamiento visual consistente aunque las imágenes originales tengan distintas proporciones.

## SEO y performance
Los catálogos son para perfumerías reales que quieren conseguir clientes desde Google.

- SEO configurable por perfumería: nombre comercial, título, descripción, logo, URL pública y datos de contacto. La configuración de una perfumería nunca se filtra a otra.
- URLs claras y estables para productos y categorías.
- Títulos y metadescripciones únicos por página indexable.
- Contenido principal de productos y categorías accesible para buscadores sin depender innecesariamente de JavaScript (ver D-001).
- Canonical cuando corresponda; sitemap XML y robots.txt.
- JSON-LD de Schema.org (Product, BreadcrumbList) sin inventar valoraciones, disponibilidad ni datos inexistentes.
- Open Graph para compartir productos en redes y mensajería.
- Imágenes optimizadas, texto alternativo descriptivo, dimensiones consistentes y carga diferida.
- Performance, accesibilidad, mobile-first y buenas prácticas de Core Web Vitals.
- Comportamiento coherente (usuarios y buscadores) para productos inactivos, agotados o eliminados.
- Tratamiento correcto de búsquedas, filtros, parámetros de URL, paginación y contenido duplicado. No generar cientos de páginas indexables sin contenido útil.
- Posibilidad de conectar Google Search Console y analítica por dominio, respetando privacidad y la configuración de cada cliente.
- Datos locales (dirección, horarios) solo si la perfumería los proporciona.
- No se garantizan posiciones ni resultados en Google.

## Diseño
Moderno, minimalista, elegante, profesional, responsive, fácil de usar, orientado a perfumería.
Prioridades: 1) usabilidad, 2) presentación de productos, 3) responsive, 4) velocidad, 5) estética.
Evitar: diseño recargado, animaciones innecesarias, gradientes exagerados, muchas sombras, aspecto de plantilla genérica.

## Infraestructura
- Git + GitHub con historial que muestre la evolución real.
- Docker solo si aporta valor real (idealmente levantar backend + MySQL con una configuración sencilla).
- Deploy de frontend, backend y MySQL administrado, eligiendo por costo, facilidad, límites gratuitos, mantenimiento y valor para portfolio, no por popularidad.

## Fuera de alcance
Microservicios, arquitectura enterprise innecesaria, funcionalidades que no aporten al producto, librerías o tecnologías agregadas para "parecer profesional".

## Preguntas abiertas (resueltas en la Etapa 1)
- ¿Qué entra en el MVP y qué queda para una segunda versión?
  → Definido en docs/roadmap.md, secciones "MVP" y "Segunda versión".
- ¿Cómo se identifica la perfumería en el catálogo público: slug en la URL, subdominio o dominio propio?
  → Slug en la ruta: `/{slug}/productos/{producto}` (D-004).
- ¿Los dominios propios por cliente son parte del MVP?
  → No. Subdominios y dominios propios quedan para la segunda versión (D-004).

## Preguntas abiertas
- ¿El primer cliente necesita su propio dominio al migrar? Si ya tiene uno y su sitio actual está posicionado en Google, el dominio propio (hoy en la segunda versión, D-004) podría tener que adelantarse a la etapa de migración.
