# Sistema Bancario Core

Proyecto académico del sector financiero. Plantea el desarrollo del núcleo (core) de un sistema bancario y, a lo largo de diez semanas, aplica **doce patrones de diseño GoF** —cinco creacionales y siete estructurales— integrados en módulos reales del sistema, cada uno con su página en la aplicación.

- **Creacionales:** **Singleton**, **Factory Method**, **Abstract Factory**, **Builder** y **Prototype**.
- **Estructurales:** **Adapter**, **Bridge**, **Composite**, **Decorator** (dos aplicaciones), **Facade**, **Flyweight** y **Proxy**.

## Acerca del proyecto

Un core bancario es el software sobre el que se apoyan todas las operaciones de un banco: gestión de cuentas, transacciones, préstamos e inversiones, operando a través de múltiples canales (web, móvil, cajeros y sucursales), con detección de fraude en tiempo real y cumplimiento regulatorio (KYC/AML).

A diferencia de las primeras etapas —en las que varias de estas funciones quedaban planteadas como proyección a futuro—, en esta versión el sistema ya tiene **todos sus módulos operativos**:

- Inicio de sesión y registro de usuarios (autenticación por cookies) y **control de acceso por rol** sobre operaciones sensibles (Proxy).
- Un **registro de eventos y auditoría** centralizado, base del control de seguridad y del cumplimiento (Singleton).
- **Notificaciones** a los clientes por distintos canales (correo real por SMTP, SMS, push e interno).
- **Canales de atención** (web, móvil, cajero y sucursal), cada uno con su propio formato de mensaje.
- **Reportes de auditoría** a partir de plantillas, descargables en Excel, y **reportes decorados** (encabezado, firma, marca de confidencialidad y observaciones).
- **Gestión de cuentas**: catálogo de productos compartido entre miles de cuentas (Flyweight).
- **Transacciones**: pagos a través de pasarelas externas integradas con adaptadores (Adapter).
- **Préstamos**: simulación del costo con cargos y descuentos combinables (Decorator).
- **Estructura organizacional**: consolidado del banco por sucursales y cuentas (Composite).
- **Documentos del cliente**: estados de cuenta y certificados en distintos formatos (Bridge).
- **Operaciones seguras**: detección de fraude y cumplimiento KYC/AML coordinados tras una sola fachada (Facade).
- **Certificaciones oficiales**: emisión costosa y sensible, con control de acceso, caché y creación diferida (Proxy).

La interfaz usa un **menú lateral** desde el que se accede a cada módulo, y un panel de inicio con tarjetas que enlazan a su página correspondiente.

## Patrones de diseño aplicados

### Patrones creacionales

#### Singleton — Registro y auditoría (`Logger`)

Garantiza que exista una única instancia encargada de escribir la bitácora en toda la aplicación, lo que asegura un historial consistente y evita problemas cuando varias operaciones se registran al mismo tiempo. Maneja distintos niveles de registro: `INFO`, `WARN`, `ERROR` y `AUDIT` (este último para los eventos que exige la normativa bancaria, como accesos y registro de clientes). El resto de los módulos reutiliza este registro para dejar constancia de sus eventos.

#### Factory Method — Notificaciones

Una interfaz `INotificador` con notificadores concretos (Email, SMS, Push e Interno) y creadores que deciden cuál instanciar, de modo que agregar un canal nuevo no obliga a modificar el código existente. El correo se envía de verdad por SMTP, tomando los datos del servidor de una tabla de configuración editable desde la propia aplicación. Se usa en el módulo de acceso (bienvenida al registrarse, alerta al iniciar sesión y aviso interno ante un intento fallido).

#### Abstract Factory — Canales de atención

Una fábrica abstracta `ICanalFactory` crea familias coherentes de piezas por canal (web, móvil, cajero y sucursal): el notificador y el formateador del mensaje siempre corresponden al mismo canal. Reutiliza los notificadores del Factory Method.

#### Builder — Reportes de auditoría

Construye un objeto complejo (el reporte) por partes: encabezado, período, filtro por usuario, resumen, lista de eventos y nota legal. Usa un builder con interfaz fluida y un director. Con el Builder se arman las plantillas de reporte que luego usa el Prototype.

#### Prototype — Plantillas de reporte

Los reportes se generan a partir de plantillas predefinidas (Completo, Diario y Seguridad de accesos). Cada plantilla es un prototipo ya configurado que se **clona** (con copia profunda) y luego se rellena con los eventos y filtros elegidos, de modo que un reporte nunca modifica la plantilla original. Un registro de plantillas (Singleton) guarda los prototipos y entrega un clon con `ObtenerClon()`.

### Patrones estructurales

#### Adapter — Transacciones (pasarelas de pago)

Traduce la interfaz de cada pasarela de pago externa a una interfaz única del sistema (`IPasarelaPago`). El núcleo trabaja siempre contra esa interfaz y cada adaptador se encarga de hablar con su API concreta (por ejemplo, PayPal o transferencia), de modo que agregar una pasarela nueva no cambia el código cliente.

#### Bridge — Documentos del cliente

Separa la abstracción (el documento: estado de cuenta, certificado…) de su implementación (el formato de salida, `IFormatoDocumento`), para que ambas jerarquías varíen por separado. Así se puede combinar cualquier documento con cualquier formato sin multiplicar clases.

#### Composite — Estructura organizacional

Organiza el banco en un árbol parte-todo (banco → sucursales → cuentas) y permite tratar igual a una hoja (`Cuenta`) y a una composición (`GrupoFinanciero`). Las operaciones —saldo total, número de cuentas— se resuelven recursivamente, por lo que se calculan métricas de cualquier nivel con el mismo método.

#### Decorator — Costos de préstamo y reportes de auditoría

Añade responsabilidades a un objeto envolviéndolo en otros que comparten su interfaz, como alternativa flexible a la herencia. Se aplica en dos lugares:

- **Costos de préstamo** (`Services/Costos`): el costo base se envuelve con cargos y descuentos combinables (comisión, seguro, recargo, descuento).
- **Reportes de auditoría** (`Services/ReportesDecorados`): un reporte base se enriquece con encabezado institucional, pie con firma, marca de confidencialidad y observaciones, en cualquier combinación.

#### Facade — Operación segura (centro de operaciones)

Ofrece una puerta de entrada única (`CentroOperacionesFacade`) para autorizar una operación bancaria. Tras una sola llamada coordina los subsistemas de **cumplimiento KYC/AML**, **detección de fraude**, **límites por canal** y **autorización**, además de auditar y notificar. El controlador no conoce esos subsistemas por dentro.

#### Flyweight — Cartera de cuentas

Comparte objetos para ahorrar memoria cuando hay muchas instancias similares. El tipo de producto (`TipoProducto`, inmutable) es el estado intrínseco compartido; miles de cuentas (`CuentaBancaria`) guardan solo una referencia a él más su estado propio (número, titular, saldo). Un catálogo (`CatalogoProductos`) entrega siempre la misma instancia por producto desde una caché.

#### Proxy — Certificaciones

Coloca un intermediario (`CertificacionProxy`) con la misma interfaz que el servicio real, para controlar el acceso a una operación costosa y sensible (emitir una certificación oficial). Combina tres variantes: **protección** (verifica permisos por rol), **caché** (no regenera un documento ya emitido) y **virtual/lazy** (crea el servicio real solo cuando se necesita).

## Módulos de la aplicación

El sistema se navega desde un menú lateral. Los principales módulos son:

| Módulo (página)        | Patrón principal            |
|------------------------|-----------------------------|
| Panel                  | —                           |
| Cartera de cuentas     | Flyweight                   |
| Estructura             | Composite                   |
| Préstamos              | Decorator                   |
| Documentos             | Bridge                      |
| Transacciones          | Adapter                     |
| Canales de atención    | Abstract Factory            |
| Operación segura       | Facade                      |
| Auditoría              | Decorator (reportes) + Singleton |
| Reportes               | Builder + Prototype         |
| Certificaciones        | Proxy                       |
| Configuración de correo| Factory Method (SMTP)       |

## Tecnologías

- ASP.NET Core 7.0 (MVC)
- C#
- Entity Framework Core
- SQL Server
- Bootstrap 5 (interfaz con menú lateral)
- ClosedXML (exportación de reportes a Excel)
- System.Net.Mail (envío de correos por SMTP)

## Requisitos previos

- [.NET 7 SDK](https://dotnet.microsoft.com/download/dotnet/7.0)
- SQL Server (Express, LocalDB o completo)
- Visual Studio 2022 (recomendado)

## Instalación

1. Descarga el proyecto en formato **.zip** y descomprímelo en una carpeta.

2. Abre la solución `SistemaBancarioCore.sln` en Visual Studio 2022.

3. Al abrir la solución, Visual Studio restaura automáticamente los paquetes NuGet (incluye **ClosedXML**). Si prefieres hacerlo a mano, en la Consola del Administrador de paquetes:

   ```
   dotnet restore
   ```

## Configuración y ejecución

1. En `appsettings.json`, ajusta la cadena de conexión con el nombre de tu servidor de SQL Server:

   ```json
   "CadenaSQL": "Server=TU_SERVIDOR;Database=LoginDb;Trusted_Connection=True;MultipleActiveResultSets=true;Encrypt=false"
   ```

2. Crea la base de datos ejecutando las migraciones. En la Consola del Administrador de paquetes de Visual Studio:

   ```
   Update-Database
   ```

   Esto crea la tabla de usuarios y la tabla de configuración de correo (`ConfiguracionesCorreo`).

3. Ejecuta el proyecto (F5). La aplicación abre en la pantalla de inicio de sesión; desde ahí puedes registrar un usuario e ingresar. Tras iniciar sesión, el menú lateral da acceso a todos los módulos.

> **Envío de correos (opcional):** para que las notificaciones lleguen de verdad, entra a la página **Configuración de correo** dentro de la aplicación y define el servidor SMTP, el correo remitente y la contraseña de aplicación. Si no se configura, el resto del sistema funciona igual y el envío simplemente no se realiza.

Los registros de la aplicación se guardan en `bin/Debug/net7.0/Logs/`, en un archivo por día.

## Estructura del proyecto

```
SistemaBancarioCore/
├── Controllers/      Login, Home, ConfiguracionCorreo, Canales, Reportes,
│                     Pagos, Documentos, Estructura, Prestamos, Auditoria,
│                     Operaciones, Cartera y Certificaciones
├── Models/           Usuario, ConfiguracionCorreo y contexto de base de datos
├── Services/
│   ├── Logger.cs            Registro y auditoría centralizados (Singleton)
│   ├── Notificaciones/      Notificadores por canal (Factory Method)
│   ├── Canales/             Canales de atención (Abstract Factory)
│   ├── Reportes/            Reportes, plantillas y export a Excel (Builder y Prototype)
│   ├── Pagos/               Pasarelas de pago (Adapter)
│   ├── Documentos/          Documentos del cliente y formatos (Bridge)
│   ├── Estructura/          Estructura organizacional del banco (Composite)
│   ├── Costos/              Costo de préstamos con cargos combinables (Decorator)
│   ├── ReportesDecorados/   Reportes de auditoría decorados (Decorator)
│   ├── Operaciones/         Centro de operaciones: fraude, KYC/AML, límites (Facade)
│   ├── Cuentas/             Catálogo de productos y cartera de cuentas (Flyweight)
│   ├── Certificaciones/     Emisión de certificaciones con proxy (Proxy)
│   └── ...                  Servicios de usuario y de archivos
├── Views/            Vistas Razor (una por módulo) y el layout con menú lateral
├── Migrations/       Migraciones de Entity Framework
├── wwwroot/          Archivos estáticos (CSS, JS y fotos de perfil)
└── appsettings.json  Configuración (cadena de conexión)
```

## Documentación

- **Semana 1 – Planteamiento del Proyecto:** introducción, problemática, justificación y objetivos del sistema.
- **Semana 2 – Patrón Singleton:** registro y auditoría centralizados.
- **Semana 3 – Patrón Factory Method:** notificaciones por distintos canales.
- **Semana 4 – Patrón Abstract Factory:** familias de piezas por canal de atención.
- **Semana 5 – Patrón Builder:** construcción de reportes de auditoría.
- **Semana 6 – Patrón Prototype:** plantillas de reporte clonables.
- **Semana 7 – Patrones Adapter y Bridge:** integración de pasarelas de pago y separación de documentos y formatos.
- **Semana 8 – Patrones Composite y Decorator:** estructura organizacional y costos de préstamo combinables.
- **Semana 9 – Patrón Decorator (profundización) y Facade:** reportes de auditoría decorados y centro de operaciones seguras.
- **Semana 10 – Patrones Flyweight y Proxy:** catálogo de productos compartido y certificaciones con control de acceso y caché.
