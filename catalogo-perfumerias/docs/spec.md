# Especificación del producto

## Objetivos
1. Base de producto reutilizable y adaptable para distintas perfumerías (clientes reales).
2. Proyecto de portfolio técnicamente sólido: frontend, backend, API REST, base de datos, autenticación y panel administrativo, que pueda explicar en entrevistas.

No es una página visualmente bonita: es una aplicación real.

## Usuarios
- **Cliente final**: navega el catálogo público sin iniciar sesión.
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
- Diseño responsive, pensado primero para celulares

## Productos
Campos mínimos: nombre, marca, categoría, género, mililitros, tipo de producto, precio, imagen, descripción, notas de salida, notas de corazón, notas de fondo, stock, activo/inactivo.

- Categorías iniciales: Perfumes, Decants, Body Splash. Deben poder agregarse nuevas sin tocar código.
- Géneros: Hombre, Mujer, Unisex.
- Tipos: EDP, EDT, Extrait de parfum, Body Splash, Decant. Deben poder agregarse nuevos valores.

## Stock
- El admin controla si un producto está disponible.
- Sin stock: se muestra claramente "Sin stock" y nunca como disponible.
- El comportamiento del botón de WhatsApp sin stock es configurable.
- No se eliminan físicamente productos por quedarse sin stock (estado activo/inactivo + stock).
- Analizar ventajas y desventajas de las alternativas antes de implementar.

## Panel administrativo
- **Productos**: crear, editar, eliminar, activar/desactivar, modificar stock y precio, subir/cambiar imagen, asignar marca, categoría y género.
- **Marcas**: crear, editar, eliminar, listar.
- **Categorías**: crear, editar, eliminar, listar.
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
- El número sale de la configuración de la perfumería, nunca hardcodeado en el frontend.

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

## Preguntas abiertas (se resuelven en la Etapa 1)
- ¿Qué entra en el MVP y qué queda para una segunda versión?
- ¿Cómo se identifica la perfumería en el catálogo público: slug en la URL, subdominio o dominio propio?
- ¿Los dominios propios por cliente son parte del MVP?
