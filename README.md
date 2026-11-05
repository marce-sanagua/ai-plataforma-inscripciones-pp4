# Plataforma de Inscripciones

Sistema web de gestión académica que permite administrar carreras, materias, inscripciones, correlatividades, equivalencias, calificaciones y mesas de examen. Incluye un asistente de inteligencia artificial que permite realizar consultas y ejecutar determinadas operaciones académicas.

## Tecnologías utilizadas

- **Frontend:** Next.js, React, TypeScript.
- **Backend:** Node.js y Express.
- **Base de datos:** MySQL 8.0.
- **Comunicación entre servicios:** API REST.
- **Autenticación:** JWT y bcrypt.
- **Contenedores:** Docker y Docker Compose.
- **Inteligencia artificial:** Gemini mediante Google AI Studio y Ollama como alternativa.

## Arquitectura del sistema

El proyecto está organizado mediante una arquitectura de microservicios. Cada servicio tiene responsabilidades específicas y se comunica con los demás mediante solicitudes HTTP.

```text
                  FRONTEND
                 Next.js
                Puerto 3000
                     |
        +------------+------------+
        |            |             |
        v            v             v
   USERS SERVICE  ACADEMIC      AI SERVICE
     Puerto 3001   SERVICE       Puerto 5000
                   Puerto 4000
        |            |             |
        +------------+-------------+
                     |
              MySQL 8.0
              Puerto 3306
          +----------+----------+
          |                     |
       users_db             academic_db

              Ollama
        Puerto interno 11434
```

### Componentes

**Frontend – plataforma-inscripciones-frontend**

Aplicación web desarrollada con Next.js y React. Permite a los usuarios interactuar con las funcionalidades del sistema.

Consume las API de los servicios del backend mediante Axios. Utiliza Zustand para el manejo de estado y React Hook Form junto con Zod para la gestión y validación de formularios.

**Users Service**

Servicio encargado de la gestión de usuarios.

Sus principales funcionalidades son:
- Registro de usuarios.
- Inicio de sesión.
- Autenticación mediante JWT.
- Gestión de usuarios.
- Administración de roles y permisos.
- Encriptación de contraseñas mediante bcrypt.

Utiliza la base de datos `users_db`.

**Academic Service**

Servicio encargado de la gestión académica.

Sus principales funcionalidades son:
- Gestión de carreras y materias.
- Administración de correlatividades.
- Gestión de equivalencias.
- Administración de períodos de inscripción.
- Inscripción a cursadas.
- Registro de calificaciones.
- Gestión de mesas de examen.
- Inscripción a finales.
- Consulta de la situación académica de los alumnos.

Utiliza la base de datos `academic_db` y se comunica con Users Service para validar información de los usuarios.

**AI Service**

Servicio que incorpora un asistente de inteligencia artificial para realizar consultas en lenguaje natural sobre la información académica.

Puede consultar el plan de estudios, verificar si un alumno puede cursar una materia o rendir un final, consultar mesas de examen y realizar determinadas operaciones de inscripción.

Para ejecutar acciones que modifican información, solicita confirmación al usuario.

El servicio utiliza Gemini mediante la API de Google AI Studio, configurada a través de variables de entorno. También contempla Ollama como alternativa.

**Base de datos MySQL**

El sistema utiliza una instancia de MySQL 8.0 con dos bases de datos independientes:

- `users_db`: almacena la información de los usuarios.
- `academic_db`: almacena la información académica.

Los servicios acceden a sus respectivas bases de datos. La comunicación entre servicios se realiza mediante API REST.

Los scripts SQL ubicados en `init-scripts/` permiten inicializar la estructura de las bases de datos cuando se crea un volumen nuevo.

Los datos se conservan mediante el volumen de Docker `mysql_data`.

## Motor de reglas académicas

El sistema cuenta con reglas para determinar la situación académica de los alumnos.

### Estados de las materias

Una materia puede encontrarse en los siguientes estados:

- `no_cursada`
- `cursando`
- `regular`
- `libre`
- `aprobada`

### Regularidad y promoción

- Para obtener la regularidad se requiere una calificación mínima de 4 en las evaluaciones correspondientes.
- Para promocionar se requiere una calificación mínima de 7 en las evaluaciones correspondientes y cumplir con las correlatividades.
- Para aprobar un examen final se requiere una calificación mínima de 4.

### Correlatividades y equivalencias

El sistema contempla correlatividades para cursar materias y para rendir finales. También permite configurar equivalencias entre materias, que pueden ser unidireccionales o bidireccionales.

### Otras funcionalidades

- Administración de períodos académicos.
- Historial de cursadas.
- Gestión de mesas de examen.
- Control de conflictos de horarios para docentes.
- Configuración de reglas académicas desde el módulo de administración.

## Inteligencia artificial

El asistente utiliza herramientas que le permiten consultar información y realizar operaciones a través de los servicios del sistema.

Entre las consultas disponibles se encuentran:

- Ver el plan de estudios.
- Consultar información de una materia.
- Verificar si se puede cursar una materia.
- Verificar si se puede rendir un final.
- Consultar materias habilitadas para cursar.
- Consultar mesas de examen.
- Consultar la situación académica del alumno.

Las operaciones de inscripción requieren confirmación antes de ejecutarse.

### Configuración del proveedor de IA

La configuración principal utiliza Gemini mediante Google AI Studio.

En el archivo `.env` ubicado en la raíz del proyecto se configuran:

- `AI_BASE_URL`: dirección de la API de Gemini.
- `AI_MODEL`: modelo principal.
- `AI_MODEL_RESPALDO`: modelo de respaldo.
- `AI_API_KEY`: clave de API de Google AI Studio.

El proyecto también incluye Ollama y el modelo `qwen2.5:1.5b` en Docker Compose como alternativa. Cuando se configura la URL de Gemini, el servicio de IA utiliza ese proveedor para las consultas, aunque el contenedor de Ollama continúa incluido en la configuración de Docker Compose.

## Variables de entorno

El proyecto utiliza dos archivos `.env`.

### 1. Archivo `.env` en la raíz del proyecto

```env
AI_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai
AI_MODEL=gemini-2.5-flash
AI_MODEL_RESPALDO=gemini-3.1-flash-lite
AI_API_KEY=tu_clave_de_google_ai_studio
CODIGO_PROFESOR=tu_codigo_de_registro
```

- `AI_API_KEY`: clave personal obtenida desde Google AI Studio.
- `CODIGO_PROFESOR`: código utilizado para autorizar el registro de usuarios con rol de profesor.

### 2. Archivo `.env` dentro de `plataforma-inscripciones-frontend`

```env
NEXT_PUBLIC_USERS_API=http://localhost:3001
NEXT_PUBLIC_ACADEMIC_API=http://localhost:4000
NEXT_PUBLIC_AI_API=http://localhost:5000
```

Estas direcciones corresponden a la configuración para ejecutar el frontend en un entorno local.

Los servicios del backend no necesitan archivos `.env` individuales. Docker Compose les proporciona las variables de entorno necesarias.

La autenticación JWT utiliza un valor predeterminado definido en el código cuando no se configura una variable específica.

**Importante:** no publicar archivos `.env` ni claves de API reales en GitHub.

## Ejecución del proyecto con Docker

### Requisitos

- Docker.
- Docker Compose.

### 1. Clonar el repositorio

```bash
git clone URL_DEL_REPOSITORIO
cd pp4-plataforma-inscripciones-docker
```

### 2. Configurar las variables de entorno

Crear los dos archivos `.env` indicados en la sección anterior y completar la clave de Google AI Studio y el código de registro de profesores.

### 3. Construir e iniciar los servicios

```bash
docker compose up -d --build
```

### 4. Verificar los contenedores

```bash
docker compose ps
```

### 5. Detener los servicios

```bash
docker compose down
```

**Importante:** para conservar los datos de MySQL, no utilizar `docker compose down -v`, ya que ese comando elimina también los volúmenes asociados.

## Puertos utilizados

| Servicio | Puerto |
|---|---:|
| Frontend | 3000 |
| Users Service | 3001 |
| Academic Service | 4000 |
| AI Service | 5000 |
| MySQL | 3306 |
| Ollama | 11434 (interno) |

## Despliegue

El proyecto fue desplegado en un servidor virtual de Oracle Cloud utilizando Ubuntu y Docker Compose.

Los servicios se ejecutan mediante contenedores Docker, lo que permite centralizar su administración y facilitar el despliegue de la aplicación.

## Documentación de los servicios

- [Users Service](./users-service/README.md)
- [Academic Service](./academic-service/README.md)
- [AI Service](./ai-service/README.md)
- [Frontend](./plataforma-inscripciones-frontend/README.md)