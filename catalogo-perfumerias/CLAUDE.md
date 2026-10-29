# Catálogo de Perfumerías

Plataforma de catálogos digitales para perfumerías: catálogo público + panel administrativo, preparada para múltiples perfumerías (multi-tenant).
Es un producto reutilizable para clientes reales y el proyecto principal de mi portfolio: tengo que poder explicar y defender cada decisión y cada archivo.

## Stack
- Backend: Java, Spring Boot, Spring Web, Spring Data JPA (Hibernate), Bean Validation, Spring Security, Maven
- Base de datos: MySQL, migraciones con Flyway
- Frontend: React + Vite + JavaScript, React Router — PENDIENTE de confirmar por SEO (ver docs/decisiones.md, D-001)
- Documentación de API: Swagger / OpenAPI (springdoc)

## Documentación del proyecto
Antes de proponer cambios, leé los documentos que apliquen a la tarea:
- docs/spec.md — qué tiene que hacer el producto (requisitos)
- docs/roadmap.md — etapas y en qué etapa estamos
- docs/decisiones.md — decisiones técnicas tomadas y su porqué
- docs/modelo-datos.md — modelo de datos (se crea en la etapa 3)

## Arquitectura backend
Paquetes por responsabilidad: controller, service, repository, entity, dto, mapper, exception, security, config.
No crear paquetes ni capas que no tengan una responsabilidad real.

- Controller: recibe la petición HTTP, valida con @Valid, delega al service y devuelve el status code correcto. Sin lógica de negocio.
- Service: lógica de negocio, transacciones y aislamiento por perfumería.
- Repository: acceso a datos. Las consultas de datos de una perfumería siempre filtran por perfumeria_id.
- Las entidades JPA nunca se devuelven desde la API: se usan DTOs de request y de response.
- Errores: excepciones propias + @RestControllerAdvice con un formato de error uniforme.

## Multi-tenant (regla crítica)
- Toda entidad que pertenece a una perfumería tiene perfumeria_id.
- En el panel admin, la perfumería sale del usuario autenticado. NUNCA de un parámetro, header o body enviado por el cliente.
- Pedir un recurso de otra perfumería responde 404 (no revelar que existe).
- Cada service que maneja datos de una perfumería tiene tests que verifican el aislamiento entre perfumerías.
- El aislamiento se garantiza en el backend. El frontend nunca es la barrera de seguridad.

## Arquitectura frontend
src/: components, pages, services (llamadas a la API), hooks, context, utils, styles.
Componentes chicos y con una responsabilidad. Nada hardcodeado: productos, WhatsApp y configuración vienen de la API.

## Prohibido
- Secrets o credenciales en el código o en Git (usar variables de entorno; .env en .gitignore)
- Contraseñas sin hash (usar BCrypt)
- Agregar dependencias, librerías o tecnologías sin preguntarme antes
- Microservicios, patrones o capas que no resuelvan un problema real de este proyecto
- Lógica de negocio en controllers o en React
- Archivos o componentes gigantes
- Firebase como base de datos principal

## Cómo trabajar conmigo
Soy desarrollador junior/intermedio y estoy aprendiendo. Actuá como desarrollador senior y mentor, no como alguien que hace todo por mí.

1. Antes de editar: explicá en pocas líneas qué vas a hacer y qué archivos vas a crear o modificar.
2. Tareas chicas: una tarea = un cambio que pueda revisar = un commit.
3. Después de editar: explicá las partes importantes, cómo probarlo y, si había una alternativa, cuál era y por qué no la elegiste.
4. Si lo que pido es demasiado complejo para la etapa actual, decímelo y proponé algo más simple.
5. Explicaciones prácticas y breves, no académicas.
6. No avances a la siguiente tarea sin mi aprobación.
7. Cuando tomemos una decisión técnica, proponé la entrada para docs/decisiones.md.
8. Al terminar una tarea, actualizá el estado en docs/roadmap.md.

## Git
- Una rama por funcionalidad (ej: feat/productos-crud).
- Commits chicos con prefijo: feat:, fix:, refactor:, docs:, test:, chore:
- Proponé el mensaje de commit; el commit lo hago yo después de revisar el diff.

## Testing
- Cada service nuevo se escribe junto con sus tests unitarios (JUnit 5 + Mockito).
- Controllers: @WebMvcTest para validaciones, status codes y permisos.
- Probar lógica de negocio, validaciones, permisos y aislamiento entre perfumerías. No probar getters/setters ni código trivial.

## Comandos
(completar cuando exista el proyecto)
- Backend: ./mvnw spring-boot:run | ./mvnw test
- Frontend: npm run dev | npm test
