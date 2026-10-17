# Microservicio de Áreas y Clientes

API REST para la gestión de **zonas de trabajo** y **clientes**, con asignación de zonas a usuarios, flujo de solicitudes de creación de clientes y exposición de datos hacia otros microservicios (usuarios, preventa y producto).

![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)
![Express](https://img.shields.io/badge/Express-4.x-000000?logo=express&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.x-4479A1?logo=mysql&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-App%20Service-0078D4?logo=microsoftazure&logoColor=white)

---

## Descripción

Servicio backend construido con **Express + TypeScript** que centraliza la información de áreas (zonas de trabajo) y clientes. Implementa autenticación mediante **JWT**, control de acceso basado en roles (`ADMINISTRADOR`, `COLABORADOR`), validación de datos con `express-validator` y documentación interactiva con **Swagger**. La persistencia se realiza sobre **MySQL** usando un pool de conexiones.

## Características

- CRUD completo de **zonas de trabajo** y **clientes**.
- Autenticación y autorización por **JWT** + roles y permisos.
- **Flujo de solicitudes** de creación de clientes con estado `Pendiente`, aprobación o rechazo por parte del administrador.
- Asignación de zonas de trabajo a usuarios (`usuario_zona`).
- Endpoints de integración para consultar datos de zonas, clientes y clientes desde la perspectiva de otros microservicios.
- Validación de entrada centralizada en middlewares.
- Documentación **OpenAPI 3.0** servida en `/api-docs`.
- Configuración de **CORS** para entornos locales y de producción.

## Stack tecnológico

| Capa | Tecnología |
| --- | --- |
| Runtime | Node.js 18+ |
| Lenguaje | TypeScript 5 |
| Framework HTTP | Express 4 |
| Base de datos | MySQL (`mysql2` con pool) |
| Autenticación | JSON Web Tokens (`jsonwebtoken`) |
| Validación | `express-validator` |
| Documentación | OpenAPI 3.0 + `swagger-ui-express` |
| Contenedor de despliegue | Azure App Service (GitHub Actions) |

## Requisitos previos

- **Node.js v18 o superior** (el pipeline de CI/CD usa `18.x`).
- **npm**.
- Una instancia de **MySQL** accesible.

## Puesta en marcha

### 1. Clonar e instalar dependencias

```bash
git clone https://github.com/JhoamSebastianMunoz/microservice-backend-area-client.git
cd microservice-backend-area-client
npm install
```

### 2. Configurar variables de entorno

Crea un archivo `.env` en la raíz del proyecto:

```env
# Servidor
PORT=8080

# Base de datos
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=microservice_area_client
DB_USERNAME=usuario
DB_PASSWORD=contraseña
DB_SSL=false

# Autenticación
JWT_SECRET=tu_clave_secreta
```

| Variable | Descripción | Valor por defecto |
| --- | --- | --- |
| `PORT` | Puerto en el que escucha el servidor | `8080` |
| `DB_HOST` | Host de la base de datos | — |
| `DB_PORT` | Puerto de MySQL | `3306` |
| `DB_DATABASE` | Nombre de la base de datos | — |
| `DB_USERNAME` | Usuario de la base de datos | — |
| `DB_PASSWORD` | Contraseña de la base de datos | — |
| `DB_SSL` | Habilita SSL (`true`/`false`) | `false` |
| `JWT_SECRET` | Clave para firmar y verificar los tokens | — |

### 3. Crear el esquema de la base de datos

Ejecuta el script `microservice_area_client.sql` sobre tu servidor MySQL. Este crea la base de datos y las tablas `zonas_de_trabajo`, `clientes` y `usuario_zona`.

### 4. Compilar y ejecutar

```bash
# Compilación en modo observador (desarrollo)
npx tsc -w

# En otra terminal, iniciar el servidor
node ./dist/app.js
```

Para un único build de producción:

```bash
npm run build   # compila TypeScript a ./dist
npm start       # ejecuta node ./dist/app.js
```

El servidor estará disponible en `http://localhost:8080` y la documentación en `http://localhost:8080/api-docs`.

## API

La documentación completa y actualizada está en **Swagger UI** (`/api-docs`). Resumen de rutas principales:

| Método | Ruta | Rol requerido | Descripción |
| --- | --- | --- | --- |
| `GET` | `/` | Público | Healthcheck del servidor |
| `GET` | `/api-docs` | Público | Documentación Swagger |
| `POST` | `/register-area` | Administrador | Registrar zona de trabajo |
| `GET` | `/get-areas` | Administrador / Colaborador | Listar zonas |
| `GET` | `/get-area/:id_zona_de_trabajo` | Administrador / Colaborador | Obtener zona por ID |
| `PUT` | `/update-area/:id_zona_de_trabajo` | Administrador | Actualizar zona |
| `DELETE` | `/delete-area/:id_zona_de_trabajo` | Administrador | Eliminar zona |
| `POST` | `/register-client` | Administrador | Registrar cliente |
| `GET` | `/get-clients` | Administrador / Colaborador | Listar clientes |
| `GET` | `/get-client/:id_cliente` | Administrador / Colaborador | Obtener cliente por ID |
| `PUT` | `/update-client/:id_cliente` | Administrador | Actualizar cliente |
| `DELETE` | `/delete-client/:id_cliente` | Administrador | Eliminar cliente |
| `POST` | `/request-create-cliente` | Colaborador | Solicitar creación de cliente |
| `GET` | `/get-Pending-Request` | Administrador / Colaborador | Listar solicitudes pendientes |
| `PUT` | `/accept-Reject-Request/:id_client` | Administrador | Aprobar una solicitud |
| `GET` | `/get_dataArea/:id_area` | Interno | Datos de zona para microservicio de usuarios |
| `GET` | `/get_clientArea/:id_area` | Interno | Clientes por zona para microservicio de usuarios |
| `GET` | `/client/:id_client` | Interno | Datos de cliente para microservicio de preventa |

Todas las rutas protegidas esperan el encabezado `Authorization: Bearer <token>`.

## Autenticación y roles

1. El token JWT es verificado por el middleware `verifyToken`, que inyecta en la petición `cedula`, `role`, `id_usuario` y `contraseña`.
2. `checkRoleAndPermission` restringe el acceso según el rol (`ADMINISTRADOR` para operaciones de escritura, `ADMINISTRADOR` y `COLABORADOR` para consultas).
3. Los colaboradores pueden **solicitar** la creación de clientes; solo el administrador puede **aprobar** (`estado = Pendiente` → `Activo`) o gestionar directamente.

## Estructura del proyecto

```
.
├── app.ts                     # Punto de entrada y montaje de rutas
├── config/                    # Configuración del pool de MySQL
├── controllers/               # Controladores por dominio
├── Dto/                       # Objetos de transferencia de datos
├── Helpers/                   # Utilidades (hash, generación de tokens)
├── middleware/                # Validadores, verificación de token y roles
├── repositories/              # Acceso a datos (SQL)
├── routes/                    # Definición de rutas por módulo
├── services/                  # Lógica de negocio
├── swagger.yaml               # Especificación OpenAPI 3.0
└── microservice_area_client.sql  # Script de base de datos
```

## Base de datos

| Tabla | Descripción |
| --- | --- |
| `zonas_de_trabajo` | Áreas o zonas de trabajo (`id_zona_de_trabajo`, `nombre_zona_trabajo`, `descripcion`). |
| `clientes` | Clientes con campos fiscales y estado (`Activo`, `Inactivo`, `Pendiente`), vinculados opcionalmente a una zona. |
| `usuario_zona` | Relación muchos-a-muchos entre usuarios y zonas de trabajo. |

## Despliegue

El despliegue está automatizado con **GitHub Actions** (`.github/workflows/master_backendareasandclients.yml`): en cada push a `master` se compila el proyecto y se publica en **Azure Web App** (`backendareasandclients`, región East US 2).

- **Aplicación en producción:** `https://backendareasandclients-apgba5dxbrbwb2ex.eastus2-01.azurewebsites.net`
- **Frontend autorizado (CORS):** `https://ambitious-sky-070d67b0f.4.azurestaticapps.net`

> [!CAUTION]
> **Despliegue en pausa por estado de créditos de Azure.** El pipeline y el despliegue en Azure están correctamente implementados y funcionando a nivel de configuración; sin embargo, el servicio se encuentra **temporalmente suspendido** porque el **pago de los créditos de los servicios de Azure está pausado momentáneamente**. La URL de producción puede no responder hasta que se reactiven dichos créditos. Para trabajar durante este periodo, ejecuta el proyecto de forma local siguiendo la sección [Puesta en marcha](#puesta-en-marcha).

## Integración con otros microservicios

Este servicio forma parte de una arquitectura de microservicios y expone información a otros componentes:

- **Usuarios:** consulta de zonas (`/get_dataArea/:id_area`) y clientes por zona (`/get_clientArea/:id_area`).
- **Preventa:** consulta de datos del cliente (`/client/:id_client`).
- **Producto:** validación y consulta de productos (integración definida en el código).
