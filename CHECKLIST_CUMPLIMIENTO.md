# Checklist de cumplimiento — Proyección de Headcount (PHP 8 MVC)

Verificación punto por punto de los dos documentos de TI.

---

## A. Documento 1 — PHP8_MVC_TEMPLATE

| # | Requisito | Cumplido | Dónde |
|---|---|---|---|
| 1 | PHP 8 puro, sin framework | ✅ | Sin Laravel/Symfony/etc. Núcleo propio en `app/Core/`. `composer.json` require php>=8.0 |
| 1 | Bootstrap 5, jQuery 3, DataTables, SweetAlert2, AJAX, .env, IIS/Apache | ✅ | `Views/layouts/*`, CDN en header/footer, `.env`, `.htaccess`, `web.config` |
| 1 | PDO con pdo_sqlsrv | ✅ | `app/Core/Database.php` (DSN `sqlsrv:`) |
| 2 | Estructura organizada (carpetas) | ✅ | Coincide con la plantilla: app/{Controllers,Models,Services,Repositories,Core,Helpers,Middleware,Views}, config, public, routes, storage, database |
| 3 | Front Controller (`public/index.php`), URL amigables, .htaccess + web.config, 404 | ✅ | `public/index.php`, `public/.htaccess`, `public/web.config`, `Router::notFound` + `Views/errors/404.php` |
| 4 | Router propio: GET/POST/PUT/DELETE, params, grupos, prefijos, middleware, web/API, JSON | ✅ | `app/Core/Router.php`, `routes/web.php`, `routes/api.php` |
| 5 | `Database.php` centralizada, PDO, excepciones, UTF-8, prepared, SPs, transacciones, commit/rollback, logs, reutilización | ✅ | `app/Core/Database.php` (+`Logger.php`) |
| 5 | No concatenar entrada del usuario en SQL | ✅ | Solo parámetros enlazados (`buildProcedureCall`) |
| 6 | Separación Controller/Service/Repository/Model/View; sin SQL en controladores/vistas | ✅ | Capas en `app/`; SQL solo en SPs vía Repositories |
| 7 | Respuesta estándar `{success,code,message,data,errors}` + helper | ✅ | `app/Core/Response.php` (success/error/validationError) + `Helpers/ResponseHelper.php` |
| 8 | Módulo CRUD de ejemplo (Usuarios): tabla indicada, SPs (listar/obtener/insertar/actualizar/activar/validar correo y usuario), listado/búsqueda/paginación/orden/CRUD/validaciones/confirmación/AJAX | ✅ | `seguridad.Usuarios`, `procedures/proc_Usuario.sql`, `Controllers/UsuarioController.php`, `Views/usuarios/*`, `assets/js/usuarios.js` |
| 9 | Plantilla admin Bootstrap 5: navbar, sidebar, menú colapsable, breadcrumb, contenedor, footer, responsive, modales, loading, SweetAlert2; layout reutilizable | ✅ | `Views/layouts/{header,navbar,sidebar,footer,main}.php` |
| 10 | DataTables Bootstrap 5: 10 por defecto, 10/20/30/50 (máx 50), búsqueda/paginación/orden, responsive, acciones, mensajes vacíos/carga | ✅ | `assets/js/site.js` (`DT_DEFAULTS` pageLength 10, lengthMenu [10,20,30,50]), tablas en usuarios/canales/auditoría |
| 11 | Formularios: etiquetas, obligatorios, validación Bootstrap, mensajes por campo, CSRF, limpieza, conservación, Guardar/Cancelar, anti doble envío, indicador, validación front+back | ✅ | `Views/*/create|edit`, `CsrfMiddleware`, `Validator`, JS de módulos |
| 12 | Seguridad: CSRF, prepared, htmlspecialchars, sanitización, sesiones seguras, regen id, password_hash/verify, middleware, anti XSS/SQLi, archivos, ocultar errores en prod, logs | ✅ | `SecurityHelper`, `Session`, `Middleware/*`, `Database`, `App::configurar`, `Logger` |
| 13 | Logs centralizados en `storage/logs/app.log` con fecha, nivel, mensaje, archivo, línea, ruta, usuario, excepción | ✅ | `app/Core/Logger.php` |
| 14 | Autenticación: login, logout, sesión, middleware, protección de rutas, clave cifrada, usuario activo, redirección a login; preparado para roles | ✅ | `AuthController`, `AuthService`, `Auth`, `AuthMiddleware`; roles en `seguridad.Roles` |
| 15 | JS/AJAX centralizado (usuarios.js), jQuery 3, SweetAlert2, recargar tablas, anti doble envío | ✅ | `assets/js/*.js` + `site.js` (Api/toast/confirmar) |
| 16 | PSR-4 autoload, PSR-12, namespaces, `declare(strict_types=1)`, tipos, SRP, sin credenciales en código | ✅ | `composer.json` (psr-4 App\), todas las clases con strict_types y tipos; credenciales solo en `.env` |
| 17 | README completo | ✅ | `README.md` (requisitos, extensiones, driver, .env, BD, scripts, composer, Apache/IIS, permisos, errores, cómo crear módulos/rutas) |
| 18 | Proyecto completo, funcional tras `composer install` + `dump-autoload`, con scripts SQL | ✅ | Proyecto entregado archivo por archivo |

---

## B. Documento 2 — Estándar de Entrega de Bases de Datos (DBA)

| # | Requisito | Cumplido | Dónde |
|---|---|---|---|
| 1 | SQL Server 2019 Standard; collation `Modern_Spanish_CI_AS`; sin features Enterprise; sin réplicas/Always On | ✅ | `00_CrearBaseDatos.sql` (COLLATE), sin particionamiento/compresión |
| 2 | Nombre de BD = nombre del sistema, sin espacios, CamelCase | ✅ | `ProyeccionHC` |
| 3 | 3FN; esquemas lógicos (nada en `dbo`) | ✅ | Esquemas `seguridad`, `catalogo`, `proyeccion`, `auditoria`, `config` (`01_Esquemas.sql`) |
| 4 | PK propia (IDENTITY); FK explícitas con ON DELETE/UPDATE; CHECK en dominios; NOT NULL | ✅ | `02_Tablas.sql` (PK IDENTITY, FK con ON DELETE, CK_*, NOT NULL) |
| 5 | Nomenclatura: `proc_`, `fn_`, `vw_`, `PK_`, `FK_`, `IX_`, `UQ_`, `CK_`, `DF_`; sin tildes/espacios; idioma consistente | ✅ | Todas las tablas/SPs/funciones/vistas; nombres de objetos sin tildes; español consistente |
| 6 | Filegroup PRIMARY; autogrowth 10% (datos y log) | ✅ | `00_CrearBaseDatos.sql` (ON PRIMARY, FILEGROWTH = 10%) |
| 7 | SPs/funciones/vistas documentados (propósito, parámetros, ejemplo); sin lógica compleja en triggers (no se usan); sin `SELECT *` | ✅ | Encabezados en cada objeto; columnas explícitas; sin triggers |
| 8 | Declarar permiso mínimo (datareader/datawriter/db_owner) justificado; no usar `sa`; sin credenciales embebidas | ✅ | `06_Seguridad_Permisos.sql` (rol con datareader+datawriter+EXECUTE, justificado); credenciales en `.env` |
| 9 | Contraseñas: nunca texto plano/reversible; hash en la app (bcrypt/Argon2/PBKDF2) + salt único; guardar hash + algoritmo + fecha; intentos/bloqueo | ✅ | `SecurityHelper::hashPassword` (bcrypt); columnas `ClaveHash`, `AlgoritmoHash`, `FechaActualizacionClave`, `IntentosFallidos`, `BloqueadoHasta`; SMTP con cifrado no reversible en app (`CryptoService`) |
| 10 | Entregables: DDL ejecutable, DML de catálogos, DER, catálogo de datos | ✅ | `scripts/*` + `InstalarTodo.sql`, `05_DatosIniciales.sql`, `DER.md`, `Catalogo_de_Datos.md` |

---

## C. Notas y decisiones (transparencia)

- **Conflicto plantilla vs. estándar DBA:** la plantilla ubicaba `Usuarios` en `dbo`, pero
  el estándar del DBA exige **nada en `dbo`**. Se usó `seguridad.Usuarios` (prevalece el DBA).
- **Nombres de objetos de BD** sin tildes (estándar). El **texto de datos** usa `NVARCHAR`
  para soportar símbolos como `₡`.
- **Vendor front (Bootstrap/jQuery/DataTables/SweetAlert2)** por CDN (documentado); la
  carpeta `public/assets/vendor/` queda disponible para alojarlos localmente si TI lo requiere
  (sin acceso a Internet en el servidor).
- **Validación de sintaxis PHP:** ejecutar en el servidor `php -l` sobre los archivos, y
  `composer install` para instalar dependencias antes del primer uso.

---

## D. Correcciones tras auditoría interna (2026-08-06)

Se realizó una auditoría archivo por archivo contra ambos documentos. Se detectaron y
**corrigieron** los siguientes puntos que estaban incompletos respecto a la exigencia literal:

### Estándar de BD (DBA)

| Punto | Hallazgo | Corrección |
|---|---|---|
| 9 — Salt único | Los 9 usuarios semilla compartían el mismo hash (mismo salt). | Cada usuario tiene ahora su **propio hash bcrypt `$2y$` con salt único** (`05_DatosIniciales.sql`). Verificado: los 9 validan la clave y los salts son distintos. |
| 8 / 9 — Credenciales en scripts | La contraseña por defecto aparecía en claro en un comentario del script SQL. | Removida del script; queda solo en el README de entrega (canal de entrega, no en la BD). |
| 7 — Ejemplo de uso | La mayoría de SPs y las 3 vistas no tenían "ejemplo de uso". | Agregado **un `Ejemplo` por objeto** en los 10 archivos de procedimientos, `03_Funciones.sql` y `04_Vistas.sql`, con nombres, esquemas y parámetros reales y valores dentro del dominio de cada CHECK. |
| 7 — Ejemplo roto | `03_Funciones.sql` citaba el esquema inexistente `dbo_no`. | Corregido a `catalogo.fn_EtiquetaMes(2026, 7)`. |
| 7 — Rowsets | (Refuerzo) Se confirmó `SET NOCOUNT ON` como primera sentencia en los 40 procedimientos. | Sin cambios (ya cumplía). |

### Plantilla PHP 8 MVC

| Req. | Hallazgo | Corrección |
|---|---|---|
| §5/§1 — Cierre de cursores | No se llamaba `closeCursor()`. | `Database::sp()` y `spExec()` ahora llaman `$stmt->closeCursor()` tras leer. |
| §11 — Validación frontend | Los formularios no ejecutaban validación Bootstrap ni mostraban mensajes por campo. | Helpers en `site.js` (`validarForm`, `pintarErrores`, `limpiarErrores`); aplicados en usuarios, canales y SMTP. Los errores `{campo:mensaje}` del backend se pintan bajo cada campo. |
| §11/§15 — Doble envío / indicador | El envío no bloqueaba el botón ni mostraba spinner. | Helper `bloquearBoton()` (spinner + `disabled`), aplicado a los envíos y acciones AJAX. |
| §10 — DataTables | `responsive` era no-op y faltaba el indicador de carga. | Añadida la extensión **Responsive** (CDN) y `processing: true` en `DT_DEFAULTS`. |
| §11 — CSRF en formulario | Los forms dependían solo del header. | Agregado `csrf_field()` en los formularios (defensa en profundidad; el header se mantiene). |
| §12 — Control de acceso por rol | El rol se verificaba solo dentro de cada controlador. | Nuevo `RoleMiddleware` (`role:super_usuario`) aplicado por ruta a Usuarios, SMTP, Auditoría y Notificaciones (web y API). |
| §16 — `declare(strict_types=1)` | Faltaba en las 20 vistas. | Agregado en las 20 vistas de `app/Views/`. |

### Pendiente para TI (no bloqueante, documentado)
- Ejecutar `php -l` sobre todos los `.php` y `composer install` (el equipo no tenía PHP local).
- Definir con TI si el servidor tiene salida a Internet; hoy los assets van por **CDN** (decisión confirmada). La carpeta `public/assets/vendor/` queda lista si se requiere modo offline.
- La app no incluye carga de archivos; el requisito de "restricción de extensiones/tamaño" (§12) no aplica por inexistencia de esa funcionalidad.
