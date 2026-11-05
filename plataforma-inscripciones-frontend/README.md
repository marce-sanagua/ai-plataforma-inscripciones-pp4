# Plataforma de Inscripciones IFTS 16 – Frontend

Aplicación web desarrollada para gestionar la actividad académica del IFTS N.º 16. Permite a estudiantes, profesores y administradores acceder a diferentes funcionalidades según su rol.

El frontend está desarrollado con **Next.js, React y TypeScript**, y se comunica con los servicios de usuarios, académico e inteligencia artificial.

## Tecnologías utilizadas

- **Next.js 16:** framework para el desarrollo de la aplicación web.
- **React 19:** construcción de interfaces de usuario.
- **TypeScript:** tipado estático.
- **Axios:** comunicación con los servicios backend.
- **Zustand:** gestión del estado global de la aplicación y persistencia.
- **Tailwind CSS:** estilos y diseño de la interfaz.
- **Radix UI:** componentes de interfaz.
- **React Hook Form y Zod:** herramientas para formularios y validaciones.
- **Sonner:** notificaciones al usuario.
- **Nodemailer:** envío de correos electrónicos.
- **Lucide React y React Icons:** iconografía.

## Funcionalidades principales

### 1. Autenticación y usuarios

- Inicio de sesión con correo electrónico y contraseña.
- Registro de usuarios.
- Selección de rol: alumno o profesor.
- Validación de los datos ingresados.
- Manejo de tokens JWT para la autenticación.
- Cierre de sesión.

### 2. Panel de administración

Permite al administrador:

- Consultar usuarios registrados.
- Buscar, filtrar y ordenar usuarios.
- Editar datos de usuarios.
- Eliminar usuarios cuando se cumplen las condiciones establecidas.
- Consultar materias e inscripciones.
- Crear y administrar materias.
- Asignar y desasignar profesores.
- Configurar horarios.
- Realizar asignaciones automáticas de profesores.
- Administrar períodos académicos y mesas de finales.
- Configurar correlatividades y equivalencias.

### 3. Panel del profesor

Permite:

- Consultar las materias asignadas.
- Visualizar los alumnos inscriptos en cada materia.
- Consultar información de cursadas.
- Acceder a las pantallas de gestión de calificaciones.

### 4. Panel del alumno

Permite:

- Consultar el plan de estudios.
- Visualizar materias aprobadas, regulares, en curso, libres y pendientes.
- Consultar el avance académico.
- Visualizar el promedio.
- Consultar horarios de cursada.
- Ver materias disponibles para inscripción.
- Consultar mesas de finales y sus fechas.
- Inscribirse a cursadas y finales cuando cumple los requisitos.
- Consultar correlatividades y los motivos por los cuales puede o no realizar una inscripción.

### 5. Gestión de calificaciones

- Consulta de calificaciones por materia.
- Carga y edición de notas de los parciales por parte de profesores y administradores.
- Registro de comentarios.
- Visualización de calificaciones por parte de los alumnos.

### 6. Asistente de inteligencia artificial

Incluye un asistente conversacional integrado a la plataforma que permite realizar consultas relacionadas con la actividad académica.

Entre sus funcionalidades se encuentran:

- Consultar el plan de estudios.
- Consultar información de materias.
- Verificar requisitos para cursar o rendir finales.
- Consultar materias disponibles.
- Consultar mesas de finales.
- Realizar inscripciones mediante confirmación del usuario.

El asistente se comunica con el servicio de inteligencia artificial del proyecto.

### 7. Envío de correos electrónicos

Incluye una ruta API para enviar correos electrónicos de bienvenida a los usuarios cuando se solicita esta operación.

El envío se realiza mediante Nodemailer y el servicio SMTP de Gmail.

## Estructura del proyecto

```text
plataforma-inscripciones-frontend/
├── components/
│   └── ui/
├── public/
├── src/
│   ├── actions.ts
│   ├── app/
│   │   ├── api/
│   │   │   └── bienvenida/
│   │   ├── dashboard/
│   │   │   ├── inscripciones/
│   │   │   ├── materia/
│   │   │   ├── materias/
│   │   │   ├── periodos/
│   │   │   ├── plan/
│   │   │   ├── reglas/
│   │   │   └── usuarios/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   ├── forms/
│   │   ├── asistente-ia.tsx
│   │   ├── header.tsx
│   │   └── panel-alumno.tsx
│   ├── lib/
│   │   ├── api.ts
│   │   ├── plan.ts
│   │   └── utils.ts
│   └── store.ts
├── Dockerfile
├── package.json
├── next.config.ts
└── tsconfig.json
```

## Variables de entorno

El frontend utiliza las siguientes variables de entorno:

| Variable | Descripción |
|---|---|
| `NEXT_PUBLIC_USERS_API` | URL del servicio de usuarios. |
| `NEXT_PUBLIC_ACADEMIC_API` | URL del servicio académico. |
| `NEXT_PUBLIC_AI_API` | URL del servicio de inteligencia artificial. |
| `GMAIL_USER` | Cuenta de Gmail utilizada para enviar correos. |
| `GMAIL_PASS` | Contraseña de aplicación de Gmail. |

Las variables deben configurarse según el entorno de ejecución. No se deben publicar contraseñas, tokens ni credenciales reales en el repositorio.

## Instalación y ejecución local

### Requisitos previos

- Node.js 20 o superior.
- npm.
- Servicios backend de usuarios y académico disponibles.
- Servicio de inteligencia artificial disponible para utilizar el asistente.

### 1. Instalar dependencias

Desde la carpeta del frontend:

```bash
npm install
```

### 2. Configurar variables de entorno

Crear un archivo `.env.local` en la raíz del frontend y configurar las variables necesarias.

### 3. Ejecutar en modo desarrollo

```bash
npm run dev
```

La aplicación estará disponible en:

```text
http://localhost:3000
```

### 4. Generar la versión de producción

```bash
npm run build
```

### 5. Ejecutar en producción

```bash
npm start
```

## Ejecución con Docker

El proyecto cuenta con un `Dockerfile` basado en Node.js 20.

La imagen instala las dependencias, genera la compilación de Next.js y ejecuta la aplicación en modo producción.

El servicio expone el puerto:

```text
3000
```

En el entorno integrado del proyecto, el frontend se ejecuta junto con los servicios backend mediante Docker Compose.

## Comunicación con los servicios backend

El frontend se comunica con:

- **Users Service:** autenticación y gestión de usuarios.
- **Academic Service:** materias, inscripciones, planes de estudio, calificaciones, períodos, mesas de finales y reglas académicas.
- **AI Service:** consultas y operaciones académicas mediante el asistente de inteligencia artificial.

Las solicitudes autenticadas utilizan tokens JWT enviados mediante encabezados `Authorization: Bearer`.

## Scripts disponibles

| Comando | Descripción |
|---|---|
| `npm run dev` | Inicia el servidor de desarrollo. |
| `npm run build` | Genera la compilación de producción. |
| `npm start` | Ejecuta la aplicación en producción. |
| `npm run lint` | Ejecuta ESLint para analizar el código. |

## Institución

**IFTS N.º 16 – Plataforma de Inscripciones**

Proyecto académico desarrollado como plataforma web para la gestión de cursadas, inscripciones y actividad académica.