# Users Service

Microservicio encargado de la gestión de usuarios y autenticación de la plataforma de inscripciones.

## Tecnologías utilizadas

- Node.js 20
- Express 5
- MySQL 8.0
- mysql2 para la conexión con la base de datos
- JSON Web Token (JWT) para la autenticación
- bcrypt para el cifrado de contraseñas
- dotenv para la carga de variables de entorno
- CORS

## Funcionalidades

- Registro de usuarios.
- Inicio de sesión.
- Autenticación mediante tokens JWT.
- Consulta de usuarios.
- Consulta de un usuario por su ID.
- Edición de datos de usuarios.
- Eliminación de usuarios.
- Protección de rutas mediante middleware de autenticación.
- Validación de roles de usuario.
- Registro de contraseñas mediante hash con bcrypt.

Los roles contemplados son:
- Alumno.
- Profesor.
- Administrador.

El registro público permite crear alumnos y profesores. Para registrar un profesor se requiere un código de autorización configurado mediante la variable `CODIGO_PROFESOR`. Los administradores se crean manualmente en la base de datos.

## Estructura del proyecto

```text
users-service/
├── src/
│   ├── controllers/
│   │   └── userController.js
│   ├── db/
│   │   └── db.js
│   ├── middlewares/
│   │   └── authMiddleware.js
│   ├── routes/
│   │   └── userRoutes.js
│   └── app.js
├── generarHash.js
├── server.js
├── package.json
└── Dockerfile
```

## Endpoints

| Método | Ruta | Descripción | Autenticación |
|---|---|---|---|
| POST | `/acceso` | Inicio de sesión | No |
| POST | `/usuarios` | Registro de usuario | No |
| GET | `/usuarios` | Listado de usuarios | Sí |
| GET | `/usuarios/:id` | Consulta de usuario por ID | Sí |
| PUT | `/usuarios/:id` | Edición de usuario | Sí (solo administrador) |
| DELETE | `/usuarios/:id` | Eliminación de usuario | Sí |

Las rutas protegidas requieren enviar el token JWT en el encabezado:

```text
Authorization: Bearer TOKEN
```

Los tokens tienen una duración de 8 horas.

## Base de datos

El servicio utiliza MySQL y se conecta a la base de datos `users_db`.

La conexión se configura mediante variables de entorno que recibe desde Docker Compose. Se utiliza un pool de conexiones para administrar las conexiones con MySQL.

## Variables de entorno

Este microservicio no necesita un archivo `.env` propio. Las variables se proporcionan desde el archivo `docker-compose.yml`.

| Variable | Descripción |
|---|---|
| `DB_HOST` | Host de MySQL |
| `DB_USER` | Usuario de la base de datos |
| `DB_PASSWORD` | Contraseña de MySQL |
| `DB_NAME` | Nombre de la base de datos |
| `DB_PORT` | Puerto de MySQL |
| `PORT` | Puerto del servicio |
| `CODIGO_PROFESOR` | Código requerido para registrar profesores |

El servicio utiliza una clave JWT predeterminada si no se configura `JWT_SECRET`. Para un entorno de producción se recomienda establecer una clave propia, segura y privada.

## Ejecución con Docker Compose

Desde la raíz del proyecto:

```bash
docker compose up --build users-service
```

Para iniciar toda la plataforma:

```bash
docker compose up --build
```

Docker Compose proporciona las variables de entorno y administra la dependencia con el contenedor de MySQL.

## Ejecución local sin Docker

Desde la carpeta del servicio:

```bash
npm install
npm run dev
```

Para ejecutarlo localmente es necesario contar con una instancia MySQL accesible y configurar las variables de entorno requeridas.

El servicio utiliza el puerto `3001` de forma predeterminada.

## Scripts disponibles

| Comando | Descripción |
|---|---|
| `npm run dev` | Inicia el servicio |
| `npm start` | Inicia el servicio |

## Administración de usuarios

Los administradores no pueden registrarse mediante el formulario público. Su creación se realiza manualmente en la base de datos.

El archivo `generarHash.js` permite generar un hash de contraseña utilizando bcrypt para ese procedimiento.

## Integración con otros servicios

El microservicio forma parte de la arquitectura de la plataforma de inscripciones y se comunica con el frontend y con los demás servicios a través de su API REST.

