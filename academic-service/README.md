# Academic Service

Microservicio encargado de gestionar la lógica académica de la plataforma de inscripciones. Administra materias, correlatividades, equivalencias, períodos académicos, inscripciones, calificaciones y mesas de finales.

También incorpora un motor de reglas académicas que permite determinar si un alumno está habilitado para cursar una materia o rendir un examen final, teniendo en cuenta su situación académica y las condiciones establecidas.

El servicio se comunica con `users-service` para consultar y validar información de usuarios, alumnos y profesores.

## Tecnologías utilizadas

- **Node.js 20**
- **Express 5**: framework para la creación de la API REST.
- **MySQL 8.0**: sistema de gestión de bases de datos.
- **mysql2**: conexión y consultas a MySQL mediante un pool de conexiones.
- **jsonwebtoken**: autenticación mediante tokens JWT.
- **axios**: comunicación con otros microservicios.
- **dotenv**: lectura de variables de entorno.
- **cors**: configuración de acceso entre orígenes.

## Requisitos previos

Para ejecutar el servicio se necesita:

- Node.js 20 o superior, si se ejecuta localmente.
- Docker y Docker Compose, si se ejecuta mediante contenedores.
- MySQL 8.0.
- El servicio `users-service` disponible para las operaciones que requieren consultar información de usuarios.

## Variables de entorno

El servicio utiliza las siguientes variables:

| Variable | Descripción |
|---|---|
| `DB_HOST` | Host de la base de datos. En Docker Compose: `mysql`. |
| `DB_USER` | Usuario de MySQL. |
| `DB_PASSWORD` | Contraseña del usuario de MySQL. |
| `DB_NAME` | Nombre de la base de datos: `academic_db` en Docker Compose. |
| `DB_PORT` | Puerto de MySQL: `3306`. |
| `PORT` | Puerto en el que se ejecuta el servicio: `4000`. |
| `USERS_SERVICE_URL` | URL base del servicio de usuarios. En Docker Compose: `http://users-service:3001`. |
| `JWT_SECRET` | Clave utilizada para verificar los tokens JWT. Si no se configura, el middleware utiliza una clave temporal definida en el código. |

**Importante:** el `docker-compose.yml` configura las variables de base de datos, puerto y URL del servicio de usuarios. No configura `JWT_SECRET`, por lo que el código utiliza su valor alternativo si la variable no está definida.

## Funcionalidades principales

### 1. Gestión de materias

Permite:

- Consultar las materias disponibles.
- Consultar una materia por su identificador.
- Crear materias.
- Administrar materias mediante las operaciones disponibles para administradores.
- Consultar los alumnos inscriptos en una materia.
- Asignar y desasignar profesores.
- Modificar horarios.
- Realizar asignaciones automáticas de profesores considerando su carga y los posibles conflictos de horarios.

El sistema también contempla restricciones para eliminar materias que tienen alumnos inscriptos o profesores asignados.

### 2. Gestión de inscripciones

Permite administrar las inscripciones de los alumnos a las materias.

Incluye:

- Consulta de períodos académicos.
- Creación de períodos.
- Apertura de períodos de inscripción.
- Inscripción de alumnos a materias.
- Consulta de inscripciones.
- Consulta de materias de un alumno.
- Consulta de alumnos inscriptos en una materia.
- Control de inscripciones duplicadas.
- Validación de las condiciones académicas necesarias para cursar.

Las inscripciones se realizan dentro de un período académico y el motor de reglas verifica si el alumno cumple las condiciones correspondientes.

### 3. Motor de reglas académicas

El motor de reglas es uno de los componentes principales del servicio.

Se encarga de evaluar la situación académica de cada alumno y determinar si puede cursar una materia o rendir su examen final.

Contempla los siguientes estados académicos:

- `no_cursada`: la materia todavía no fue cursada.
- `cursando`: el alumno se encuentra cursando la materia.
- `regular`: el alumno alcanzó las condiciones de regularidad.
- `libre`: el alumno no alcanzó las condiciones necesarias para mantener la regularidad.
- `aprobada`: la materia se encuentra aprobada.

También considera:

- Calificaciones de los parciales.
- Notas de los exámenes finales.
- Correlatividades.
- Equivalencias entre materias.
- Períodos académicos.
- Cuatrimestres.
- Horarios y posibles superposiciones.
- Inscripciones anteriores y situación académica del alumno.

#### Reglas de aprobación y regularidad

El motor contempla los siguientes criterios:

| Condición | Criterio |
|---|---|
| Regularidad | Ambos parciales con nota igual o superior a 4. |
| Promoción | Ambos parciales con nota igual o superior a 7 y cumplimiento de las correlatividades requeridas. |
| Aprobación del final | Nota igual o superior a 4. |

#### Correlatividades configurables

Las correlatividades permiten establecer qué materias deben estar regularizadas o aprobadas para poder cursar otra materia o rendir su final.

Se contemplan dos condiciones:

- `para_cursar`: estado requerido para poder cursar.
- `para_final`: estado requerido para poder rendir el final.

Los estados admitidos para estas condiciones son `regular` y `aprobada`.

Esto permite configurar reglas diferentes según las exigencias académicas de cada materia.

#### Equivalencias entre materias

El motor contempla equivalencias entre materias de distintas carreras.

Cuando una materia se encuentra aprobada y existe una equivalencia configurada, esa aprobación puede ser reconocida para la materia equivalente.

Las equivalencias pueden ser:

- **Bidireccionales:** la equivalencia se reconoce en ambos sentidos.
- **Unidireccionales:** la equivalencia se reconoce solamente desde la materia de origen hacia la materia de destino.

#### Validación para cursar y rendir

El motor ofrece operaciones para:

- Consultar el plan académico de un alumno.
- Consultar el estado de sus materias.
- Verificar si puede cursar una materia.
- Verificar si puede rendir un examen final.
- Informar los motivos por los cuales una operación está permitida o bloqueada.

### 4. Gestión de calificaciones

Permite:

- Consultar las calificaciones de los alumnos.
- Registrar notas de parciales.
- Actualizar calificaciones.
- Eliminar registros de calificaciones.
- Consultar información correspondiente a distintos períodos académicos.

El sistema contempla la relación entre las calificaciones y los períodos en los que fueron registradas.

### 5. Gestión de mesas de finales

Permite administrar las mesas de examen final.

Incluye:

- Consulta de mesas disponibles.
- Creación y eliminación de mesas por parte de administradores.
- Inscripción de alumnos a mesas.
- Consulta de alumnos inscriptos.
- Cancelación de inscripciones.
- Registro de notas de finales.

Antes de permitir que un alumno se inscriba, se verifica si está habilitado para rendir según las reglas académicas.

También se contemplan restricciones para cancelar inscripciones y eliminar mesas cuando existen alumnos inscriptos.

### 6. Gestión de profesores

El servicio permite consultar las materias asignadas a un profesor y los alumnos inscriptos en sus materias.

Para obtener información de los usuarios y verificar sus roles, se comunica con `users-service`.

### 7. Administración de reglas académicas

Los administradores pueden gestionar las reglas que determinan las condiciones académicas de cursada y aprobación.

Incluye:

- Consultar correlatividades.
- Crear correlatividades.
- Modificar condiciones de correlatividad.
- Eliminar correlatividades.
- Crear equivalencias.
- Eliminar equivalencias.
- Configurar equivalencias bidireccionales o unidireccionales.

El sistema contempla validaciones para evitar relaciones duplicadas, autorreferencias y ciclos en las correlatividades.

## Base de datos

El servicio utiliza la base de datos `academic_db`.

Entre las principales tablas se encuentran:

| Tabla | Descripción |
|---|---|
| `carreras` | Información de las carreras. |
| `materias` | Materias correspondientes a las carreras. |
| `correlativas` | Relaciones y requisitos entre materias. |
| `equivalencias` | Equivalencias entre materias. |
| `periodos` | Períodos académicos y cuatrimestres. |
| `inscripciones` | Inscripciones de alumnos a materias. |
| `calificaciones` | Notas y calificaciones de los alumnos. |
| `mesas_finales` | Mesas de examen final. |
| `inscripciones_finales` | Inscripciones de alumnos a mesas de finales. |

### Scripts SQL

La estructura y configuración de la base de datos se complementa con scripts de inicialización:

- `02-academic-schema.sql`: creación del esquema académico y sus tablas principales.
- `06-reglas-configurables.sql`: incorporación de condiciones configurables para las correlatividades y creación de equivalencias.
- `07-equivalencias-oficiales.sql`: configuración de equivalencias específicas entre materias.
- `09-periodos.sql`: creación y configuración de períodos académicos y asociación de inscripciones y calificaciones con dichos períodos.

Los scripts deben ejecutarse respetando el orden previsto por la inicialización de la base de datos.

## Autenticación y autorización

El servicio utiliza tokens JWT para autenticar las solicitudes.

El middleware de autenticación verifica el token recibido en el encabezado `Authorization` mediante el esquema `Bearer`.

También cuenta con un middleware de autorización por roles que permite restringir determinadas operaciones según el perfil del usuario.

Entre los roles contemplados se encuentran:

- `alumno`
- `profesor`
- `admin`

Las operaciones administrativas requieren los permisos correspondientes.

## Rutas principales de la API

Las rutas se organizan en los siguientes grupos:

| Prefijo | Funcionalidad |
|---|---|
| `/materias` | Consulta y gestión de materias y calificaciones. |
| `/admin` | Administración de materias, profesores y horarios. |
| `/inscripciones` | Inscripciones a materias y períodos académicos. |
| `/profesores` | Consultas relacionadas con profesores y sus materias. |
| `/motor` | Consulta del plan académico y validaciones para cursar y rendir. |
| `/finales` | Mesas de finales, inscripciones y calificaciones de finales. |
| `/reglas` | Administración de correlatividades y equivalencias. |

Las operaciones disponibles incluyen métodos HTTP `GET`, `POST`, `PUT` y `DELETE`, según el recurso y la acción.

## Pruebas del motor académico

El proyecto incluye pruebas del motor de reglas implementadas utilizando el módulo `assert` de Node.js.

Estas pruebas verifican distintos escenarios de funcionamiento:

- Una materia regularizada permite cursar la siguiente cuando la correlativa exige regularidad.
- La regularidad no alcanza para rendir cuando se exige tener aprobada la correlativa.
- La modificación de las condiciones de correlatividad cambia el resultado de las validaciones.
- La aprobación de un final habilita las condiciones académicas correspondientes.
- Las equivalencias permiten reconocer materias aprobadas en otras carreras.
- Se verifican equivalencias bidireccionales y unidireccionales.
- Se contemplan equivalencias circulares.
- Se mantienen las reglas predeterminadas cuando se utilizan datos anteriores sin las nuevas columnas.
- Se valida que una materia pertenezca a la carrera del alumno.
- Se controla el cuatrimestre correspondiente para cursar.
- Se contempla la posibilidad de recursar materias en estado libre cuando corresponde.

**Nota:** el script de pruebas utiliza `assert` y muestra el resultado de cada prueba en consola. El comando `npm test` todavía no está configurado para ejecutarlas: actualmente devuelve el mensaje de prueba no especificada definido en `package.json`.

## Ejecución del servicio

### Con Docker Compose

La forma recomendada de ejecutar el proyecto es desde su carpeta principal, donde se encuentra `docker-compose.yml`.

Para levantar el sistema completo:

```bash
docker compose up --build
```

Este comando inicia los servicios definidos en Docker Compose, incluyendo:

- MySQL.
- `users-service`.
- `academic-service`.
- `ai-service`.
- Ollama.
- Frontend.

Para iniciar específicamente el servicio académico junto con sus dependencias y el servicio de usuarios:

```bash
docker compose up --build academic-service users-service
```

Docker Compose también iniciará MySQL, requerido por ambos servicios.

### Ejecución local sin Docker

Desde una terminal:

```bash
cd academic-service
npm install
npm run dev
```

Para esta modalidad es necesario configurar las variables de entorno, disponer de una instancia MySQL con la base de datos correspondiente y tener `users-service` disponible en la URL configurada.

## Scripts disponibles

| Comando | Descripción |
|---|---|
| `npm install` | Instala las dependencias del servicio. |
| `npm run dev` | Inicia el servidor ejecutando `node server.js`. |
| `npm start` | Inicia el servidor ejecutando `node server.js`. |
| `npm test` | Actualmente no tiene pruebas configuradas en el script de npm. |

## Puerto

El servicio utiliza el puerto **4000**.

En Docker Compose se encuentra publicado mediante el mapeo:

```yaml
ports:
  - "4000:4000"
```

El servicio puede ser utilizado por otros componentes de la plataforma a través de su dirección correspondiente dentro de la red de Docker.