# Prueba Técnica — Desarrollador Backend C# .NET + Oracle

**Empresa:** Celsia (Sector Energético)
**Cargo:** Desarrollador Backend C# / .NET / Oracle
**Versión del documento:** 1.0
**Tipo de prueba:** Individual, no supervisada, entrega asincrónica

---

## 1. Introducción

### 1.1 Objetivo

Esta prueba técnica busca evaluar la capacidad real de un candidato para diseñar, construir y sustentar una API backend de nivel productivo utilizando C#, .NET y Oracle Database, sobre un dominio de negocio representativo del sector eléctrico.

No se evalúa memorización de sintaxis. Se evalúa:

- Criterio de arquitectura.
- Calidad y mantenibilidad del código.
- Modelado correcto de datos relacionales en Oracle.
- Buenas prácticas de ingeniería de software.
- Capacidad de justificar decisiones técnicas.
- Seguridad y rendimiento aplicados de forma consciente, no accidental.

### 1.2 Duración recomendada

**4 a 6 horas efectivas de trabajo.** No es necesario resolver el 100% de lo solicitado; se prioriza la calidad de lo entregado sobre la cantidad. Es preferible una porción del alcance resuelta con excelencia arquitectónica que el alcance completo resuelto de forma superficial.

### 1.3 Reglas

1. El desarrollo debe ser individual. Se permite el uso de documentación oficial y buscadores.
2. Se permite el uso de asistentes de IA como apoyo, pero el candidato debe poder explicar y defender cada línea de código en la sustentación técnica posterior.
3. El proyecto debe compilar y ejecutar sin pasos manuales no documentados.
4. Todo el código debe subirse a un repositorio Git con historial de commits real (no un único commit "initial commit").
5. Está prohibido copiar soluciones completas de repositorios públicos existentes para este mismo caso de negocio.
6. Cualquier supuesto no especificado en este documento debe quedar documentado explícitamente en el README (decisiones de diseño propias).

### 1.4 Qué se evaluará

| Dimensión | Se evalúa |
|---|---|
| Arquitectura | Separación de capas, bajo acoplamiento, alta cohesión |
| Modelado de datos | Normalización, integridad referencial, uso correcto de Oracle |
| Calidad de código | Legibilidad, SOLID, manejo de errores |
| Dominio de C#/.NET | Uso idiomático del lenguaje y del framework |
| Dominio de Oracle | SQL avanzado, PL/SQL, rendimiento |
| Seguridad | Prevención de vulnerabilidades comunes (OWASP) |
| Rendimiento | Consultas eficientes, paginación, manejo de N+1 |
| Pruebas | Cobertura de casos relevantes, no triviales |
| Comunicación técnica | Documentación y claridad del README |

### 1.5 Tecnologías permitidas

- C# 12 / .NET 8 o superior
- ASP.NET Core Web API
- Oracle Database (11g XE, 19c o 21c XE)
- Entity Framework Core (proveedor Oracle.EntityFrameworkCore)
- LINQ
- Swagger / OpenAPI
- Git
- Docker (opcional, recomendado)
- FluentValidation, AutoMapper, Serilog, MediatR, xUnit, Redis (opcionales, ver sección Bonus)

### 1.6 Tecnologías prohibidas

- ORMs distintos a Entity Framework Core como mecanismo principal de acceso a datos (Dapper puede usarse como complemento puntual y justificado, no como reemplazo total).
- Frameworks de bases de datos no relacionales como almacenamiento primario del dominio.
- Generadores de proyectos "todo en uno" tipo scaffolding automático de CRUD sin intervención del candidato (ej. plantillas comerciales que generan la solución completa).
- Copiar y pegar un proyecto público existente que resuelva este mismo caso de negocio.

### 1.7 Criterios de evaluación

La evaluación es cualitativa y cuantitativa. Se combina:

- Rúbrica de 100 puntos (ver sección 17).
- Revisión de código línea a línea por parte de un ingeniero senior.
- Sustentación técnica oral de 30 a 45 minutos donde el candidato explica y defiende sus decisiones.

---

## 2. Requisitos Técnicos

### 2.1 Obligatorio

- C#
- .NET 8 o superior
- ASP.NET Core Web API
- Oracle Database
- Entity Framework Core
- LINQ
- Swagger
- Git

### 2.2 Opcional (suma puntos adicionales, ver sección 19 — Bonus)

- Docker
- FluentValidation
- AutoMapper
- Serilog
- MediatR
- xUnit
- Redis

---

## 3. Caso de Negocio

Celsia requiere una API REST interna para la **administración de infraestructura eléctrica de distribución**. El sistema debe permitir el control operativo y de mantenimiento de los activos físicos de la red, así como el trabajo de campo de los técnicos.

El sistema debe administrar las siguientes entidades de negocio, con relaciones reales entre ellas:

- **Subestaciones**: instalaciones que transforman y distribuyen energía a una zona geográfica.
- **Transformadores**: equipos físicos ubicados dentro de una subestación.
- **Circuitos**: líneas de distribución que salen de un transformador hacia zonas de consumo.
- **Técnicos**: personal de campo encargado de inspecciones y mantenimientos.
- **Inspecciones**: revisiones técnicas periódicas realizadas sobre transformadores o circuitos.
- **Órdenes de Mantenimiento**: trabajos correctivos o preventivos generados sobre un activo, que pueden originarse a partir de una inspección.

**Restricción explícita:** el dominio debe modelarse fielmente como un sistema de infraestructura eléctrica empresarial. No se aceptan soluciones que reutilicen un dominio genérico de tienda, biblioteca o catálogo de películas disfrazado con otros nombres.

### 3.1 Reglas de negocio mínimas que la API debe respetar

1. Una subestación tiene uno o más transformadores; un transformador pertenece exactamente a una subestación.
2. Un transformador puede alimentar uno o más circuitos; un circuito pertenece exactamente a un transformador.
3. Una inspección se realiza sobre un transformador **o** sobre un circuito (no ambos a la vez), y es ejecutada por un técnico.
4. Una orden de mantenimiento puede generarse manualmente o a partir de una inspección con hallazgos; siempre está asociada a un técnico responsable y a un activo (transformador o circuito).
5. Los activos (subestación, transformador, circuito) deben poder activarse/inactivarse sin perder su historial de inspecciones u órdenes (soft delete).
6. Un técnico inactivo no puede ser asignado a nuevas inspecciones ni órdenes, pero conserva su historial.

---

## 4. Modelo de Base de Datos Oracle

### 4.1 Requisitos del modelo

El modelo debe incluir:

- Primary Keys explícitas en todas las tablas.
- Foreign Keys con las reglas de integridad referencial correctas (`ON DELETE` según corresponda a cada relación; se sugiere `RESTRICT` para preservar historial).
- Restricciones (`CHECK`, `NOT NULL`, `UNIQUE`) donde el dominio lo exija.
- Índices en columnas de búsqueda frecuente y en todas las Foreign Keys.
- Secuencias Oracle (`SEQUENCE`) o `GENERATED AS IDENTITY` para las claves primarias — el candidato debe justificar cuál usó y por qué.
- Normalización mínima hasta 3FN.
- Tipos de datos nativos de Oracle (`NUMBER`, `VARCHAR2`, `DATE`/`TIMESTAMP`, `CHAR(1)` para flags booleanos, etc. — no usar tipos genéricos ANSI sin justificar).

### 4.2 Entidades mínimas esperadas

- `SUBESTACION`
- `TRANSFORMADOR`
- `CIRCUITO`
- `TECNICO`
- `INSPECCION`
- `ORDEN_MANTENIMIENTO`
- Tablas de catálogo/apoyo que el candidato considere necesarias (ej. `ESTADO_ORDEN`, `TIPO_INSPECCION`, `ESPECIALIDAD_TECNICO`), justificando su existencia frente a usar simples `CHECK` constraints.

### 4.3 Diagrama relacional (referencial, no vinculante)

El siguiente diagrama es una guía de cardinalidad mínima esperada. El candidato puede enriquecerlo, pero no simplificarlo por debajo de esto:

```
SUBESTACION (1) ──< (N) TRANSFORMADOR (1) ──< (N) CIRCUITO
                              │                        │
                              │                        │
                              ▼                        ▼
                         INSPECCION  ──────────────────┘
                              │  (N) ──── (1) TECNICO
                              │
                              ▼ (origina, opcional)
                    ORDEN_MANTENIMIENTO
                              │  (N) ──── (1) TECNICO
                              │  (N) ──── (1) TRANSFORMADOR o CIRCUITO
```

**Notas de diseño esperadas del candidato:**

- Cómo modelará la relación polimórfica "una inspección/orden aplica a Transformador **o** Circuito" (alternativas válidas: tabla `ACTIVO` genérica con discriminador, o dos Foreign Keys nulables con `CHECK` que garantice exactamente una no nula). Se espera que el candidato elija una, la justifique y la documente.
- Manejo de auditoría mínima (`FECHA_CREACION`, `FECHA_MODIFICACION`, `USUARIO_CREACION`) en todas las tablas transaccionales.
- Manejo de soft delete (`ACTIVO CHAR(1) DEFAULT 'S'` o similar) en las tablas de activos y técnicos.

---

## 5. Historias de Usuario

| # | Historia | Prioridad | Criterios de aceptación |
|---|---|---|---|
| 1 | Como administrador, quiero registrar una nueva subestación con su ubicación y capacidad, para llevar el inventario de infraestructura. | Alta | Debe validar campos obligatorios; código de subestación único; retorna 201 con el recurso creado. |
| 2 | Como administrador, quiero listar subestaciones con paginación y filtro por estado (activa/inactiva), para consultar el inventario de forma eficiente. | Alta | Respuesta paginada con metadatos (total, página, tamaño); filtro combinable con búsqueda por nombre. |
| 3 | Como administrador, quiero inactivar una subestación sin eliminarla físicamente, para preservar el historial de mantenimientos asociados. | Alta | La subestación no aparece en listados activos por defecto; sus transformadores no se eliminan; operación auditable. |
| 4 | Como ingeniero de planta, quiero registrar un transformador asociado a una subestación existente, para reflejar la topología real de la red. | Alta | Valida que la subestación exista y esté activa; valida capacidad y tipo de transformador. |
| 5 | Como ingeniero de planta, quiero consultar todos los transformadores de una subestación específica, para planear mantenimientos por zona. | Media | Endpoint anidado o filtro por `subestacionId`; incluye conteo de circuitos asociados. |
| 6 | Como ingeniero de planta, quiero registrar un circuito asociado a un transformador, para modelar la distribución hacia zonas de consumo. | Alta | Valida existencia y estado activo del transformador; valida longitud y capacidad del circuito. |
| 7 | Como coordinador de campo, quiero registrar técnicos con su especialidad y estado, para asignarlos a inspecciones y órdenes. | Alta | Documento de identidad único; especialidad debe pertenecer a un catálogo válido. |
| 8 | Como coordinador de campo, quiero inactivar un técnico, para que no pueda ser asignado a nuevas tareas sin perder su historial. | Media | Técnico inactivo no aparece en combos de asignación, pero su historial de inspecciones/órdenes persiste. |
| 9 | Como técnico, quiero registrar una inspección sobre un transformador o un circuito, indicando hallazgos y estado resultante. | Alta | Exactamente un activo asociado (transformador o circuito); técnico debe estar activo; fecha no puede ser futura. |
| 10 | Como coordinador de campo, quiero generar automáticamente una orden de mantenimiento a partir de una inspección con hallazgos críticos, para agilizar la respuesta operativa. | Alta | Operación transaccional: si falla la creación de la orden, la inspección no debe quedar en estado inconsistente. |
| 11 | Como coordinador de campo, quiero crear una orden de mantenimiento manual (sin inspección previa), para atender reportes externos o correctivos urgentes. | Alta | Requiere técnico y activo válidos; estado inicial "Pendiente". |
| 12 | Como coordinador de campo, quiero actualizar el estado de una orden de mantenimiento (Pendiente, En Proceso, Finalizada, Cancelada), para hacer seguimiento operativo. | Alta | Transiciones de estado válidas controladas (no se puede pasar de Finalizada a Pendiente, por ejemplo). |
| 13 | Como supervisor, quiero consultar órdenes de mantenimiento con filtros por técnico, estado, rango de fechas y tipo de activo, para generar reportes operativos. | Alta | Filtros combinables; resultados paginados y ordenables. |
| 14 | Como supervisor, quiero consultar el historial completo de inspecciones y órdenes de un transformador específico, para evaluar su estado de salud. | Media | Endpoint agregado que retorna ambos historiales ordenados cronológicamente. |
| 15 | Como supervisor, quiero obtener un reporte de carga de trabajo por técnico (cantidad de órdenes activas), para balancear asignaciones. | Media | Consulta agregada (`GROUP BY`) con conteo por técnico y estado. |
| 16 | Como administrador del sistema, quiero que todas las operaciones de escritura queden auditadas con fecha y usuario, para trazabilidad. | Alta | Campos de auditoría poblados automáticamente, no manipulables por el cliente de la API. |
| 17 | Como administrador del sistema, quiero que la API valide y rechace datos inconsistentes (fechas inválidas, referencias inexistentes) con mensajes de error claros, para evitar corrupción de datos. | Alta | Respuestas de error estructuradas (código, mensaje, detalle de validación por campo). |
| 18 | Como supervisor, quiero buscar circuitos por múltiples parámetros combinados (subestación, estado, rango de capacidad), para análisis operativo avanzado. | Media | Endpoint de búsqueda flexible con parámetros opcionales combinables vía query string. |

---

## 6. API REST

El candidato debe construir endpoints que cubran, como mínimo, sobre las entidades principales:

- **CRUD completo** para Subestación, Transformador, Circuito, Técnico y Orden de Mantenimiento.
- **Paginación** (parámetros `pageNumber` y `pageSize`, con metadatos de respuesta: total de registros, total de páginas).
- **Búsqueda** por texto libre (ej. nombre de subestación, identificación de técnico).
- **Filtros** combinables (ej. órdenes por estado + técnico + rango de fechas).
- **Ordenamiento** dinámico (`sortBy`, `sortDirection`) sobre al menos dos campos por entidad.
- **Búsqueda por múltiples parámetros** simultáneos vía query string.
- **Soft Delete** en entidades de infraestructura y técnicos (no eliminación física).
- **Activar/Inactivar** registros como operación explícita (endpoint dedicado, no solo un PATCH genérico).
- **Validaciones** de negocio y de formato en cada operación de escritura.
- **Relación entre entidades** reflejada correctamente en los DTOs de respuesta (sin sobre-exponer ni causar N+1).
- **Operaciones transaccionales**: como mínimo, la generación de una orden de mantenimiento a partir de una inspección debe ser atómica.

Todos los endpoints deben estar documentados en Swagger con ejemplos de request/response y códigos de estado HTTP correctos y consistentes.

---

## 7. Oracle

El candidato debe demostrar dominio real de Oracle, incluyendo (como entregable independiente, en un script `.sql`, además de lo implementado vía EF Core):

- Consultas SQL con `JOIN` y `LEFT JOIN` entre al menos 4 tablas.
- `GROUP BY` y `HAVING` para reportes agregados (ej. cantidad de órdenes por técnico y estado).
- Subconsultas correlacionadas y no correlacionadas.
- Funciones Oracle (de fecha, cadena y agregación) aplicadas en un caso real del dominio.
- Sentencia `MERGE` para un caso de sincronización o upsert (ej. actualizar estado de activo si existe, insertarlo si no).
- Al menos un procedimiento almacenado que encapsule una operación transaccional del dominio (ej. generar orden de mantenimiento desde inspección).
- Al menos una función almacenada reutilizable (ej. calcular antigüedad de un activo).
- Al menos un trigger que garantice una regla de integridad no expresable solo con constraints (ej. impedir inspección con fecha futura, o mantener campos de auditoría).
- Uso justificado de un cursor explícito en un caso donde el procesamiento fila a fila sea genuinamente necesario.
- Al menos un paquete (`PACKAGE`/`PACKAGE BODY`) que agrupe funciones y procedimientos relacionados con un mismo dominio (ej. `PKG_MANTENIMIENTO`).
- Manejo explícito de transacciones (`COMMIT`, `ROLLBACK`, manejo de excepciones PL/SQL) en el procedimiento de generación de orden de mantenimiento.
- Definición de al menos dos índices adicionales justificados por un patrón de consulta real, mostrando el plan de ejecución (`EXPLAIN PLAN`) antes y después.
- Documentar en el README qué consultas fueron optimizadas y por qué (con evidencia del plan de ejecución).

---

## 8. C#

El candidato debe demostrar dominio idiomático de:

- Programación Orientada a Objetos aplicada correctamente al dominio (no solo clases anémicas).
- Principios SOLID, con ejemplos identificables en el propio código.
- Interfaces bien diseñadas (segregadas, no "interfaces de Dios").
- Generics donde aporten reutilización real.
- Delegates y Events donde el desacoplamiento lo justifique.
- LINQ (métodos de extensión, sintaxis de consulta, proyecciones, agregaciones).
- Extension Methods propios donde tengan sentido.
- `async`/`await` correctamente aplicado de punta a punta (sin bloqueos síncronos sobre código asíncrono).
- Uso de `Task` y `CancellationToken` en operaciones de I/O.
- Inyección de dependencias (constructor injection, lifetimes correctos: `Scoped`, `Singleton`, `Transient`).
- Middleware personalizado (al menos uno, ej. manejo global de excepciones).
- Manejo de excepciones estructurado (excepciones de dominio propias, no solo `try/catch` genérico).
- Logging estructurado.
- DTOs claramente separados de entidades de dominio.
- Records para modelos inmutables donde aplique.
- Pattern Matching en al menos un caso real (no forzado).
- Nullable Reference Types habilitado y respetado en todo el proyecto.
- Uso correcto de colecciones (`IEnumerable` vs `List` vs `IReadOnlyCollection` según el contexto de uso).

El candidato debe poder explicar, para cada elección relevante, **cuándo usarla y cuándo no**.

---

## 9. .NET

El candidato debe demostrar dominio de:

- Minimal APIs **o** Controllers (elección justificada; se permite mezclar si hay una razón de diseño clara).
- Configuración de Dependency Injection en el contenedor nativo de .NET.
- `IOptions<T>` para tipado fuerte de configuración.
- Gestión de `appsettings.json` por ambiente y variables de entorno para secretos (cadena de conexión Oracle nunca hardcodeada).
- Middleware pipeline correctamente ordenado.
- Al menos un Background Service (`IHostedService`/`BackgroundService`) con un caso de uso real del dominio (ej. job que marca inspecciones vencidas).
- Caching (in-memory como mínimo; Redis como bonus).
- Autenticación con JWT.
- Autorización basada en roles/claims (ej. rol `Tecnico` vs `Supervisor` vs `Administrador` con permisos distintos).
- Filtros de acción o de excepción donde correspondan.
- Swagger/OpenAPI con seguridad JWT configurada (candado en la UI).
- Health Checks (`/health`) incluyendo verificación de conectividad a Oracle.
- Versionado de API (al menos estrategia definida, aunque solo exista v1).

---

## 10. Entity Framework Core

El candidato debe demostrar dominio de:

- Fluent API para configurar el modelo (no depender solo de Data Annotations en un dominio de este tamaño).
- Migraciones correctamente versionadas y aplicables desde cero.
- Uso consciente de `Tracking` vs `AsNoTracking()` según el tipo de operación (lectura vs escritura).
- Explicación razonada sobre Lazy Loading, Eager Loading (`Include`) y Explicit Loading, y cuál usó en cada caso y por qué.
- Manejo de concurrencia (optimista, con `RowVersion`/`Timestamp` o mecanismo equivalente) en al menos una entidad crítica (ej. Orden de Mantenimiento).
- Transacciones explícitas con EF Core (`BeginTransaction`) para la operación de generación de orden desde inspección.
- LINQ avanzado traducido correctamente a SQL (evitar traer datos a memoria innecesariamente).
- Evidencia de que las consultas están optimizadas (sin N+1, con proyecciones `Select` en listados en lugar de traer entidades completas).

---

## 11. Arquitectura

Se solicita explícitamente:

- **Clean Architecture**: separación clara en capas (Domain, Application, Infrastructure, API/Presentation), con dependencias apuntando siempre hacia el dominio.
- **Repository Pattern**: abstracción del acceso a datos detrás de interfaces definidas en la capa de dominio/aplicación.
- **Unit of Work**: coordinación de transacciones cuando una operación afecta múltiples repositorios (ej. inspección + orden de mantenimiento).
- **CQRS** (opcional, ver Bonus): si se implementa, debe estar genuinamente justificado por la complejidad del caso de uso, no aplicado de forma cosmética.
- Separación de responsabilidades clara entre controladores, casos de uso/servicios de aplicación y lógica de dominio.
- Inversión de dependencias real (el dominio no debe depender de Entity Framework ni de ningún detalle de infraestructura).
- El candidato debe poder justificar cada capa: qué problema resuelve y qué costo de complejidad agrega. **No se premia sobreingeniería.**

---

## 12. Rendimiento

El candidato debe estar preparado para responder y, donde aplique, demostrar en el código:

- Estrategia de escalabilidad horizontal de la API (statelessness, afinidad de sesión, etc.).
- Uso de caché para reducir carga sobre Oracle en consultas de catálogo (ej. especialidades, estados).
- Paginación obligatoria en cualquier endpoint que pueda retornar colecciones grandes.
- Uso correcto de índices en las columnas que soportan los filtros expuestos por la API.
- Identificación y solución del problema N+1 en al menos un listado con relaciones (ej. transformadores con conteo de circuitos).
- Estrategia de optimización ante consultas lentas (identificación vía plan de ejecución, no solo intuición).
- Configuración de connection pooling hacia Oracle y justificación de los parámetros elegidos.
- Gestión consciente del ciclo de vida de la conexión (`DbContext` con lifetime `Scoped`, sin fugas de conexión).

---

## 13. Seguridad

El candidato debe aplicar y estar preparado para explicar:

- Autenticación basada en JWT (emisión, validación, expiración, refresh si aplica).
- Autorización basada en roles y/o claims a nivel de endpoint.
- Validación estricta de entrada en todos los endpoints de escritura (rechazo de payloads malformados o incompletos).
- Prevención de SQL Injection (uso exclusivo de parámetros vía EF Core / comandos parametrizados; ningún SQL concatenado con input de usuario).
- Prevención de XSS en cualquier dato que eventualmente se renderice (sanitización, encoding de salida).
- Mitigación de CSRF si la API expone algún flujo basado en cookies.
- Gestión de secretos (cadena de conexión, claves JWT) exclusivamente vía variables de entorno o un gestor de secretos, nunca en el repositorio.
- Configuración correcta de CORS (no `AllowAnyOrigin` en un entorno productivo simulado, salvo justificación explícita para el entorno de pruebas).
- Rate Limiting en al menos los endpoints de autenticación.

---

## 14. Pruebas

Se solicita:

- Unit Tests sobre la lógica de dominio y de aplicación (casos de uso), con mocking de dependencias externas (repositorios, servicios).
- Integration Tests sobre al menos dos endpoints críticos (ej. creación de orden de mantenimiento, listado paginado con filtros).
- Uso de mocking (ej. `Moq`/`NSubstitute`) para aislar dependencias en pruebas unitarias.
- Cobertura razonada: no se exige un porcentaje arbitrario, pero sí que estén cubiertos los casos de negocio críticos y al menos dos casos borde por funcionalidad clave (ej. inspección sin activo asociado, orden con técnico inactivo).
- El candidato debe poder explicar qué NO probó y por qué, si el tiempo no alcanzó.

---

## 15. Git

Se evaluará:

- Commits atómicos con mensajes descriptivos (no `"fix"`, `"cambios"`, `"wip final v2"`).
- Uso de ramas con un propósito identificable (ej. `feature/`, `fix/`), aunque el flujo final se integre a `main`/`develop`.
- Historial que permita reconstruir el proceso de construcción de la solución, no un volcado único de código.
- Buenas prácticas generales: `.gitignore` correcto (sin `bin/`, `obj/`, secretos, ni archivos de IDE innecesarios versionados).

---

## 16. Preguntas Técnicas

> 30 preguntas abiertas de nivel intermedio-avanzado. Cada una incluye la respuesta esperada (nivel de profundidad mínimo aceptable), los puntos clave que el evaluador debe verificar, y el nivel de dificultad.

| # | Pregunta | Respuesta esperada (resumen) | Puntos clave a verificar | Nivel |
|---|---|---|---|---|
| 1 | ¿Cuál es la diferencia entre `IEnumerable<T>` e `IQueryable<T>` en el contexto de EF Core? | `IQueryable` difiere la ejecución y traduce la expresión a SQL; `IEnumerable` fuerza evaluación en memoria una vez materializado. | Menciona ejecución diferida, traducción a SQL, riesgo de traer datos innecesarios a memoria. | Intermedio |
| 2 | ¿Cuándo usarías `AsNoTracking()` y por qué mejora el rendimiento? | En consultas de solo lectura; evita que EF Core mantenga el snapshot de cambios para detección de modificaciones. | Explica el costo del change tracker y su relación con memoria/CPU. | Intermedio |
| 3 | Explica la diferencia entre concurrencia optimista y pesimista, y cuándo usarías cada una en este dominio. | Optimista: valida versión al guardar (ej. `RowVersion`); pesimista: bloquea el registro durante la transacción. | Relaciona con el caso de Orden de Mantenimiento y contención esperada. | Avanzado |
| 4 | ¿Qué problema resuelve el patrón Unit of Work sobre el Repository Pattern por sí solo? | Coordina múltiples repositorios bajo una sola transacción, evitando estados inconsistentes. | Ejemplo concreto: inspección + orden generada simultáneamente. | Intermedio |
| 5 | ¿Por qué el dominio no debería depender de Entity Framework directamente? | Para mantener el núcleo de negocio independiente de detalles de infraestructura (inversión de dependencias, testabilidad). | Menciona Clean/Hexagonal Architecture y facilidad de reemplazo de infraestructura. | Intermedio |
| 6 | Explica qué es el problema N+1 y cómo lo evitarías en un listado de transformadores con su conteo de circuitos. | Se genera una consulta adicional por cada fila del resultado principal. | Menciona `Include`, proyección con `Select` y agregación en la propia consulta SQL. | Intermedio |
| 7 | ¿Qué ventajas tiene usar `record` sobre `class` para un DTO? | Igualdad por valor, inmutabilidad por defecto, sintaxis concisa. | Explica cuándo NO conviene (entidades con identidad y ciclo de vida mutable). | Básico-Intermedio |
| 8 | ¿Cómo prevendrías SQL Injection incluso si en algún punto necesitaras SQL dinámico? | Uso de parámetros ligados (`bind variables`), nunca concatenación de input de usuario. | Menciona parametrización explícita y validación de entrada. | Básico |
| 9 | ¿Qué diferencia hay entre `Task` y `Task<T>`, y por qué es importante evitar `async void`? | `Task` representa una operación sin valor de retorno; `async void` no permite propagar excepciones ni ser esperado (`await`). | Explica el riesgo de excepciones no observadas. | Intermedio |
| 10 | Explica el propósito de un `CancellationToken` y en qué escenario de este dominio lo usarías. | Permite cancelar cooperativamente una operación en curso. | Ejemplo: cancelar una consulta larga si el cliente cierra la petición. | Intermedio |
| 11 | ¿Qué es una secuencia (`SEQUENCE`) en Oracle y en qué se diferencia de `GENERATED AS IDENTITY`? | Secuencia es un objeto independiente reutilizable; `IDENTITY` está ligado a la columna directamente. | Explica portabilidad, control manual vs automático. | Intermedio |
| 12 | ¿Cuándo usarías un `MERGE` en lugar de un `INSERT`/`UPDATE` separados? | Cuando se necesita upsert atómico basado en una condición de coincidencia. | Ejemplo aplicado al dominio (sincronizar estado de activo). | Intermedio |
| 13 | ¿Qué diferencia hay entre un procedimiento almacenado y una función almacenada en Oracle? | La función retorna un valor y puede usarse en una expresión SQL; el procedimiento no retorna valor directamente y se invoca como sentencia. | Menciona uso de `OUT` parameters en procedimientos. | Básico-Intermedio |
| 14 | ¿Cuándo es apropiado usar un trigger y cuándo es un antipatrón? | Apropiado para reglas de integridad no expresables con constraints; antipatrón cuando reemplaza lógica de negocio que debería estar explícita en la aplicación. | Explica el riesgo de lógica "oculta" y efectos en cascada difíciles de rastrear. | Avanzado |
| 15 | ¿Qué es un plan de ejecución y cómo lo usarías para optimizar una consulta lenta? | Muestra cómo el optimizador de Oracle ejecutará la consulta (accesos, joins, uso de índices). | Menciona `EXPLAIN PLAN`, costo, full table scan vs index range scan. | Avanzado |
| 16 | ¿En qué casos un índice puede perjudicar el rendimiento en lugar de mejorarlo? | Tablas con alta frecuencia de escritura, columnas de baja selectividad, exceso de índices que penalizan `INSERT`/`UPDATE`. | Explica el trade-off lectura vs escritura. | Avanzado |
| 17 | ¿Qué es la normalización y por qué podrías, deliberadamente, desnormalizar algo en este dominio? | Normalización reduce redundancia; desnormalización controlada mejora rendimiento de lectura en reportes específicos. | Ejemplo concreto justificado, no genérico. | Intermedio |
| 18 | Explica la diferencia entre autenticación y autorización, y cómo las implementarías para los roles Técnico/Supervisor/Administrador. | Autenticación verifica identidad; autorización verifica permisos. | Menciona JWT claims, `[Authorize(Roles=...)]` o políticas. | Básico-Intermedio |
| 19 | ¿Qué es CORS y por qué `AllowAnyOrigin` combinado con credenciales es peligroso? | CORS controla qué orígenes pueden consumir la API desde el navegador; combinarlo con credenciales expone a robo de sesión. | Explica el riesgo concreto de esa combinación. | Intermedio |
| 20 | ¿Cómo diseñarías el rate limiting para el endpoint de autenticación? | Limitar intentos por IP/usuario en una ventana de tiempo, con respuesta 429. | Menciona mitigación de fuerza bruta. | Intermedio |
| 21 | ¿Qué diferencia hay entre paginación por `OFFSET`/`FETCH` y paginación por keyset (cursor)? | `OFFSET` recorre y descarta filas (costoso en páginas altas); keyset usa el último valor visto como filtro. | Explica cuándo keyset es preferible (datasets grandes). | Avanzado |
| 22 | ¿Cómo asegurarías que la generación de una orden de mantenimiento desde una inspección sea atómica? | Transacción explícita que agrupe ambas operaciones, con rollback ante cualquier fallo. | Menciona manejo de excepciones dentro de la transacción. | Intermedio |
| 23 | ¿Qué es Nullable Reference Types y qué problema real previene? | Distingue en tiempo de compilación referencias que pueden ser `null` de las que no, reduciendo `NullReferenceException`. | Explica que es una ayuda del compilador, no una garantía en runtime absoluta. | Intermedio |
| 24 | ¿Cuándo usarías Minimal APIs en lugar de Controllers, y viceversa? | Minimal APIs para endpoints simples/microservicios pequeños; Controllers cuando hay mayor complejidad, filtros, convenciones compartidas. | Justifica según tamaño y complejidad del dominio. | Intermedio |
| 25 | ¿Qué es un Health Check y qué debería verificar en esta API además de "está viva"? | Endpoint que reporta el estado de la aplicación y sus dependencias críticas. | Debe mencionar verificación de conectividad a Oracle, no solo un 200 OK vacío. | Básico-Intermedio |
| 26 | Explica el principio de Responsabilidad Única aplicado a un controlador de esta API. | Un controlador no debería contener lógica de negocio ni acceso directo a datos; delega a la capa de aplicación. | Ejemplo concreto del dominio (controlador de Órdenes delegando a un servicio/caso de uso). | Básico-Intermedio |
| 27 | ¿Qué ventajas tiene usar DTOs en lugar de exponer directamente las entidades de dominio en la API? | Evita sobre-exposición de datos, desacopla el contrato público del modelo interno, previene problemas de serialización circular. | Menciona over-posting/under-posting como riesgo de seguridad. | Intermedio |
| 28 | ¿Cómo manejarías el versionado de esta API si en el futuro cambia el contrato de Órdenes de Mantenimiento? | Estrategia de versionado explícita (URL, header o media type), manteniendo compatibilidad hacia atrás mientras exista consumo de v1. | Explica impacto en consumidores existentes. | Intermedio |
| 29 | ¿Qué es el patrón Outbox y en qué escenario de este dominio podría aplicar? | Garantiza consistencia entre una escritura transaccional y la publicación de un evento (ej. notificar que se generó una orden crítica). | Explica el problema de "escritura dual" que resuelve. | Avanzado |
| 30 | Si tuvieras que escalar esta API horizontalmente, ¿qué aspectos del diseño actual revisarías primero? | Statelessness de la API, gestión de caché distribuida, pooling de conexiones a Oracle, idempotencia de operaciones críticas. | Respuesta debe ser específica al diseño que el propio candidato construyó, no genérica. | Avanzado |

---

## 17. Rúbrica de Evaluación (100 puntos)

| Categoría | Peso | Descripción de lo evaluado |
|---|---|---|
| Arquitectura | 15 | Separación de capas, bajo acoplamiento, inversión de dependencias, ausencia de sobreingeniería |
| Modelo de datos Oracle | 15 | Normalización, integridad referencial, tipos de datos correctos, uso de secuencias/índices |
| .NET / ASP.NET Core | 10 | Uso idiomático del framework, DI, middleware, configuración |
| C# | 10 | SOLID, manejo de async/await, LINQ, manejo de excepciones, nullable reference types |
| Entity Framework Core | 10 | Fluent API, migraciones, tracking consciente, manejo de concurrencia y transacciones |
| Código limpio | 10 | Legibilidad, nombres significativos, funciones pequeñas y cohesivas, ausencia de duplicación |
| Pruebas | 10 | Unit tests, integration tests, mocking, cobertura de casos borde |
| Seguridad | 10 | JWT, autorización por roles, prevención de OWASP Top 10, manejo de secretos |
| Rendimiento | 5 | Paginación, ausencia de N+1, uso correcto de índices, caching |
| Git y documentación | 5 | Historial de commits, README claro, entregables completos |
| **Total** | **100** | |

**Umbral orientativo de aprobación:** 70/100, con al menos 60% del puntaje individual obtenido en las categorías de Arquitectura, Modelo de Datos y Seguridad (no se compensa una debilidad crítica en estas áreas únicamente con puntaje alto en otras).

---

## 18. Entregables

El candidato debe entregar:

1. **Repositorio Git** (público o con acceso otorgado al evaluador) con historial de commits real.
2. **README.md** que incluya: instrucciones de ejecución, decisiones de arquitectura tomadas, supuestos asumidos, qué quedó pendiente y por qué, y cómo ejecutar las pruebas.
3. **Script Oracle** (`.sql`) con creación de esquema, secuencias, índices, triggers, procedimientos, funciones y paquetes, ejecutable de principio a fin sobre una instancia limpia.
4. **Colección Postman** (o archivo `.http`/Insomnia equivalente) con todos los endpoints documentados y ejemplos de uso, incluyendo el flujo de autenticación.
5. **Proyecto compilable** de principio a fin (`dotnet build` y `dotnet run` sin pasos manuales adicionales no documentados).
6. **Documentación técnica** de las decisiones de diseño relevantes (puede integrarse en el README o en un archivo `ARQUITECTURA.md` separado).

---

## 19. Bonus (retos opcionales para candidatos Senior)

Estos puntos **no son obligatorios** pero suman puntaje adicional y son especialmente valorados en candidatos que aspiran a un rol senior:

- **Event Driven**: publicar un evento de dominio (ej. `OrdenMantenimientoCriticaCreada`) cuando se genera una orden crítica.
- **RabbitMQ**: consumir/publicar el evento anterior a través de una cola, desacoplando el proceso de notificación.
- **Redis**: caché distribuido para catálogos de baja variabilidad (especialidades, estados) o para resultados de reportes agregados.
- **Docker**: `Dockerfile` y `docker-compose.yml` que levanten la API junto con Oracle (o el contenedor de Oracle XE correspondiente).
- **CI/CD**: pipeline (GitHub Actions u otro) que compile, ejecute pruebas y publique un artefacto/imagen.
- **Observabilidad / OpenTelemetry**: trazas distribuidas mínimas sobre al menos un flujo crítico (creación de orden de mantenimiento).
- **Serilog**: logging estructurado con sinks configurables (consola + archivo o servicio externo).
- **Health Checks avanzados**: verificación de dependencias externas (Oracle, Redis, RabbitMQ) con reporte detallado por dependencia.
- **Polly**: políticas de resiliencia (retry, circuit breaker) sobre llamadas a dependencias externas.
- **Rate Limiting** más allá de autenticación: aplicado también a endpoints de escritura críticos.
- **Cache Distribuido**: invalidación consciente de caché al modificar datos subyacentes.
- **Background Services**: job programado que marque automáticamente inspecciones vencidas o genere alertas.
- **Outbox Pattern**: garantizar consistencia entre la escritura transaccional de la orden y la publicación del evento asociado.

---

*Fin del documento. Este archivo debe entregarse al candidato tal cual, sin resolver, como enunciado oficial de la prueba técnica.*