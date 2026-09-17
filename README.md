# Proyección de Headcount — Canales Comerciales (CR · NI · PA · SV)

Aplicación web para la proyección mensual de headcount de los canales comerciales de
Instacredit en Costa Rica, Nicaragua, Panamá y El Salvador. Flujo: captura (gerente
país) → aprobación regional comercial → aprobación regional financiero → reporte
consolidado a RRHH/CEO. Rolling de 6 meses por país, con auditoría y notificaciones.

**Stack:** PHP 8 puro (MVC, sin framework) · SQL Server 2019 · PDO (pdo_sqlsrv) ·
Bootstrap 5 · jQuery 3 · DataTables · SweetAlert2 · AJAX.

---

## 1. Requisitos

- **PHP 8.0 o superior** (probado con 8.1+).
- **SQL Server 2019** (instancia `SRV-CARAGRAL\CARAGAL`, collation `Modern_Spanish_CI_AS`).
- **Composer** 2.x.
- **Servidor web:** Apache o IIS (con reescritura de URL).
- **Node/npm:** NO se requiere (el frontend es server-side con Bootstrap/jQuery por CDN).

### Extensiones de PHP necesarias

Habilitar en `php.ini`:

```
extension=pdo_sqlsrv
extension=sqlsrv
extension=mbstring
extension=openssl
extension=fileinfo
```

### Instalación del controlador SQL Server para PHP

1. Descargar **Microsoft Drivers for PHP for SQL Server** (versión acorde a tu PHP y
   arquitectura) desde el sitio de Microsoft.
2. Copiar `php_pdo_sqlsrv_XX_ts_x64.dll` (y `php_sqlsrv_...`) a la carpeta `ext` de PHP.
3. Agregar las líneas `extension=...` de arriba en `php.ini`.
4. Instalar **ODBC Driver 18 for SQL Server** en el servidor.
5. Verificar: `php -m | findstr sqlsrv` debe listar `pdo_sqlsrv` y `sqlsrv`.

---

## 2. Configuración (`.env`)

Copiar `.env.example` a `.env` y completar:

```ini
APP_URL=http://localhost/ProyeccionHC/public
APP_ENV=production
APP_DEBUG=false
APP_KEY=<cadena-larga-aleatoria>          # generar (ver abajo)

DB_HOST=SRV-CARAGRAL\CARAGAL
DB_PORT=1433
DB_DATABASE=ProyeccionHC
DB_USERNAME=<usuario-asignado-por-DBA>    # NO 'sa'
DB_PASSWORD=<clave-asignada-por-DBA>
DB_ENCRYPT=false
DB_TRUST_SERVER_CERTIFICATE=true
```

Generar `APP_KEY`:

```bash
php -r "echo bin2hex(random_bytes(32));"
```

---

## 3. Creación de la base de datos

Los scripts están en `database/` y cumplen el **Estándar de Entrega de Bases de Datos**
del DBA (esquemas fuera de `dbo`, PK/FK/CHECK, nomenclatura, collation, etc.).

**Opción A — Modo SQLCMD (SSMS: Consulta → Modo SQLCMD):**

```
:r database/InstalarTodo.sql
```

**Opción B — Manual, en este orden:**

```
database/scripts/00_CrearBaseDatos.sql
database/scripts/01_Esquemas.sql
database/scripts/02_Tablas.sql
database/scripts/03_Funciones.sql
database/scripts/04_Vistas.sql
database/scripts/05_DatosIniciales.sql
database/scripts/06_Seguridad_Permisos.sql
database/procedures/*.sql        (todos)
```

Luego, el **DBA** crea el usuario de aplicación y lo agrega al rol
`rol_app_proyeccionhc` (permiso mínimo: `db_datareader` + `db_datawriter` + `EXECUTE`;
ver instrucciones en `06_Seguridad_Permisos.sql`).

Entregables para el DBA: `database/DER.md` (diagrama entidad-relación) y
`database/Catalogo_de_Datos.md` (diccionario de datos).

> **Usuarios semilla** (contraseña inicial `Instacredit.2026`, **debe cambiarse en el
> primer ingreso**): `admin` (super), `gerente.cr` / `gerente.ni` / `gerente.pa` /
> `gerente.sv`, `comercial`, `financiero`, `rrhh`, `ceo`.
> Cada usuario tiene su **propio hash bcrypt con salt único** (estándar DBA, punto 9);
> la contraseña en claro NO está embebida en los scripts SQL, solo aquí para la entrega.

---

## 4. Instalación de dependencias (Composer)

```bash
composer install
composer dump-autoload -o
```

Instala phpdotenv, PHPMailer, mPDF y PhpSpreadsheet, y genera el autoload PSR-4.

### Validación de sintaxis (recomendado antes del primer despliegue)

El equipo de desarrollo no tenía PHP instalado localmente, por lo que se pide a TI
ejecutar una verificación de sintaxis sobre todos los archivos (no requiere BD):

```bash
for /R app %f in (*.php) do @php -l "%f"
php -l public/index.php
```

Debe reportar `No syntax errors detected` en cada archivo.

---

## 5. Configuración del servidor web

El **punto de entrada único** es `public/index.php` (patrón Front Controller). El
document root debe apuntar a `public/`.

### Apache

- Habilitar `mod_rewrite`.
- El archivo `public/.htaccess` ya reescribe todas las rutas a `index.php`.
- Ejemplo de VirtualHost: `DocumentRoot ".../ProyeccionHC/public"` y
  `<Directory>` con `AllowOverride All`.

### IIS

- Instalar el módulo **URL Rewrite**.
- El archivo `public/web.config` ya contiene las reglas de reescritura.
- Configurar el sitio con la ruta física en `public/` y el manejador de PHP (FastCGI).

---

## 6. Permisos de carpetas

La carpeta `storage/` debe tener permiso de **escritura** para el usuario del
servicio web (logs, cache, uploads):

- Windows: dar "Modificar" a `IIS_IUSRS` (o al usuario del pool) sobre `storage/`.
- Linux: `chmod -R 775 storage` y el propietario del proceso web.

---

## 7. Ejecución local

- Abrir `APP_URL` en el navegador (p. ej. `http://localhost/ProyeccionHC/public`).
- Servidor embebido de PHP para pruebas rápidas:
  ```bash
  php -S localhost:8000 -t public
  ```
  (Nota: requiere `pdo_sqlsrv` habilitado también en esta CLI de PHP.)

---

## 8. Tareas programadas (notificaciones)

Recordatorios, escalamiento al CEO y reporte a RRHH. Programar a diario:

```bash
php scripts/notificaciones.php            # envía
php scripts/notificaciones.php --dry-run  # solo simula
```

El envío requiere configurar el relay SMTP en la pantalla **SMTP** (súper usuario) y
marcarlo como activo.

---

## 9. Solución de errores comunes

| Síntoma | Causa / solución |
|---|---|
| `could not find driver` | Falta `pdo_sqlsrv`. Habilitar la extensión y ODBC Driver 18. |
| `Login failed for user` | Usuario/clave del `.env` incorrectos o sin permiso; el DBA debe agregarlo al rol `rol_app_proyeccionhc`. |
| Página en blanco / 500 | Ver `storage/logs/app.log`. Con `APP_DEBUG=true` (solo dev) muestra el detalle. |
| Rutas dan 404 | Falta `mod_rewrite` (Apache) o URL Rewrite (IIS), o el document root no apunta a `public/`. |
| El símbolo `₡` se ve mal | Asegurar collation `Modern_Spanish_CI_AS` y columnas NVARCHAR (ya definidas). |
| CSRF inválido | Recargar la página para renovar el token. |

---

## 10. Cómo crear un nuevo módulo MVC

1. **Tabla + SP:** crear la tabla en `database/scripts/02_Tablas.sql` (esquema lógico) y
   los procedimientos `proc_*` en `database/procedures/`.
2. **Repository** (`app/Repositories/MiRepository.php`): métodos que llaman a los SPs vía
   `Database::getInstance()->sp('esquema.proc_...', ['@Param' => valor])`.
3. **Service** (`app/Services/MiService.php`): reglas de negocio, validaciones y auditoría.
4. **Controller** (`app/Controllers/MiController.php`): extiende `App\Core\Controller`;
   métodos de página (`$this->view(...)`) y AJAX (`$this->manejar(fn () => Response::success(...))`).
5. **Vista** (`app/Views/mimodulo/index.php`): usa el layout (Bootstrap 5). El JS del
   módulo se autocarga si existe `public/assets/js/mimodulo.js`.
6. **Rutas:** registrar en `routes/web.php` (páginas) y `routes/api.php` (JSON), con los
   middleware `auth` y `csrf` según corresponda.

### Cómo registrar nuevas rutas

```php
// routes/web.php  (dentro del grupo con middleware 'auth')
$router->get('/mimodulo', [MiController::class, 'index']);

// routes/api.php  (grupo prefix '/api', middleware 'auth')
$router->get('/mimodulo',        [MiController::class, 'listar']);
$router->post('/mimodulo',       [MiController::class, 'guardar'], ['csrf']);
$router->put('/mimodulo/{id}',   [MiController::class, 'actualizar'], ['csrf']);
$router->delete('/mimodulo/{id}',[MiController::class, 'eliminar'], ['csrf']);
```

Toda respuesta AJAX/API usa el formato estándar:
`Response::success($data, $mensaje)` · `Response::error($mensaje, $codigo)` ·
`Response::validationError($errores)`.

---

## 11. Estructura del proyecto

```
ProyeccionHC/
├── app/
│   ├── Controllers/   Controladores (uno por módulo)
│   ├── Models/        Estructura de datos (DTOs)
│   ├── Services/      Reglas de negocio (incluye CalculadoraHeadcount)
│   ├── Repositories/  Acceso a datos (stored procedures)
│   ├── Core/          Núcleo MVC (Router, Database, Response, Auth, ...)
│   ├── Helpers/       Utilidades (Url, Security, Response, Formato)
│   ├── Middleware/    AuthMiddleware, CsrfMiddleware
│   ├── Views/         Vistas Bootstrap 5 (layout + módulos + errores)
│   └── helpers.php    Funciones globales (env, config, e, csrf, ...)
├── config/            app.php, database.php, mail.php, routes.php
├── database/          scripts/, procedures/, DER.md, Catalogo_de_Datos.md, InstalarTodo.sql
├── public/            index.php (Front Controller), .htaccess, web.config, assets/
├── routes/            web.php, api.php
├── scripts/           notificaciones.php (cron)
├── storage/           logs/, cache/, uploads/
├── .env / .env.example
├── composer.json
└── README.md
```

---

¡Apoyándote siempre! · Instacredit
