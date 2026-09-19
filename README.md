# Sistema Bancario Core

Proyecto académico del sector financiero. Plantea el desarrollo del núcleo (core) de un sistema bancario y, por etapas, implementa el módulo de acceso, un sistema de registro y auditoría centralizado, notificaciones multicanal, canales de atención y generación de reportes, aplicando cinco patrones de diseño creacionales: **Singleton**, **Factory Method**, **Abstract Factory**, **Builder** y **Prototype**.

## Acerca del proyecto

Un core bancario es el software sobre el que se apoyan todas las operaciones de un banco: gestión de cuentas, transacciones, préstamos e inversiones, operando a través de múltiples canales (web, móvil, cajeros y sucursales), con detección de fraude en tiempo real y cumplimiento regulatorio (KYC/AML).

En esta etapa el desarrollo se concentra en:

- Inicio de sesión y registro de usuarios (autenticación por cookies).
- Un registro de eventos y auditoría centralizado, que es la base del control de seguridad y del cumplimiento.
- Notificaciones a los clientes por distintos canales (correo real por SMTP, SMS, push e interno).
- Canales de atención (web, móvil, cajero y sucursal), cada uno con su propio formato de mensaje.
- Generación de reportes de auditoría a partir de plantillas, descargables en Excel.

Las demás funciones del banco (cuentas, transacciones, préstamos, inversiones y detección de fraude) forman parte del alcance conceptual del sistema y quedan planteadas como proyección a futuro.

## Patrones de diseño aplicados

### Singleton — Registro y auditoría (`Logger`)

Garantiza que exista una única instancia encargada de escribir la bitácora en toda la aplicación, lo que asegura un historial consistente y evita problemas cuando varias operaciones se registran al mismo tiempo. La clase maneja distintos niveles de registro: `INFO`, `WARN`, `ERROR` y `AUDIT` (este último para los eventos que exige la normativa bancaria, como accesos y registro de clientes). El resto de los módulos reutiliza este registro para dejar constancia de sus eventos.

### Factory Method — Notificaciones

Una interfaz `INotificador` con notificadores concretos (Email, SMS, Push e Interno) y creadores que deciden cuál instanciar, de modo que agregar un canal nuevo no obliga a modificar el código existente. El correo se envía de verdad por SMTP, tomando los datos del servidor de una tabla de configuración editable desde la propia aplicación. Se usa en el módulo de acceso (bienvenida al registrarse, alerta al iniciar sesión y aviso interno ante un intento fallido).

### Abstract Factory — Canales de atención

Una fábrica abstracta `ICanalFactory` crea familias coherentes de piezas por canal (web, móvil, cajero y sucursal): el notificador y el formateador del mensaje siempre corresponden al mismo canal. Reutiliza los notificadores del Factory Method.

### Builder — Reportes de auditoría

Construye un objeto complejo (el reporte) por partes: encabezado, período, filtro por usuario, resumen, lista de eventos y nota legal. Usa un builder con interfaz fluida y un director. Con el Builder se arman las plantillas de reporte que luego usa el Prototype.

### Prototype — Plantillas de reporte

Los reportes se generan a partir de plantillas predefinidas (Completo, Diario y Seguridad de accesos). Cada plantilla es un prototipo ya configurado que se **clona** (con copia profunda) y luego se rellena con los eventos y filtros elegidos, de modo que un reporte nunca modifica la plantilla original. Un registro de plantillas (Singleton) guarda los prototipos y entrega un clon con `ObtenerClon()`. Así se reutiliza la configuración y agregar una plantilla nueva es tan simple como registrarla una vez.

## Tecnologías

- ASP.NET Core 7.0 (MVC)
- C#
- Entity Framework Core
- SQL Server
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

3. Ejecuta el proyecto (F5). La aplicación abre en la pantalla de inicio de sesión; desde ahí puedes registrar un usuario e ingresar.

> **Envío de correos (opcional):** para que las notificaciones lleguen de verdad, entra a la página **Configuración de correo** dentro de la aplicación y define el servidor SMTP, el correo remitente y la contraseña de aplicación. Si no se configura, el resto del sistema funciona igual y el envío simplemente no se realiza.

Los registros de la aplicación se guardan en `bin/Debug/net7.0/Logs/`, en un archivo por día.

## Estructura del proyecto

```
SistemaBancarioCore/
├── Controllers/      Login, Home, ConfiguracionCorreo, Canales y Reportes
├── Models/           Usuario, ConfiguracionCorreo y contexto de base de datos
├── Services/
│   ├── Logger.cs            Registro y auditoría centralizados (Singleton)
│   ├── Notificaciones/      Notificadores por canal (Factory Method)
│   ├── Canales/             Canales de atención (Abstract Factory)
│   ├── Reportes/            Reportes, plantillas y export a Excel (Builder y Prototype)
│   └── ...                  Servicios de usuario y de archivos
├── Views/            Vistas Razor (Login, Home, ConfiguracionCorreo, Canales, Reportes)
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
