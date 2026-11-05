# AI Service – Plataforma de Inscripciones

## Descripción

AI Service es el microservicio encargado de brindar un asistente de inteligencia artificial para la Plataforma de Inscripciones.

Permite que los usuarios realicen consultas en lenguaje natural sobre su información académica y ejecuten determinadas operaciones mediante herramientas que se comunican con Users Service y Academic Service.

El servicio utiliza Gemini mediante la API de Google AI Studio. También contempla Ollama como alternativa de ejecución local.

## Tecnologías utilizadas

- Node.js 20
- Express 5
- JavaScript
- JWT para autenticación
- Axios para la comunicación entre servicios
- API compatible con OpenAI
- Gemini mediante Google AI Studio
- Ollama como alternativa local

## Estructura del proyecto

```text
ai-service/
├── src/
│   ├── app.js
│   ├── auth.js
│   └── tools.js
├── package.json
├── package-lock.json
├── server.js
└── Dockerfile
```

## Configuración

El servicio no utiliza un archivo `.env` propio. Las variables de entorno se proporcionan desde el `docker-compose.yml` y el archivo `.env` ubicado en la raíz del proyecto.

### Variables de entorno

| Variable | Descripción |
|---|---|
| `PORT` | Puerto del servicio. Por defecto, 5000. |
| `ACADEMIC_SERVICE_URL` | URL interna de Academic Service. |
| `USERS_SERVICE_URL` | URL interna de Users Service. |
| `AI_BASE_URL` | Dirección de la API del proveedor de inteligencia artificial. |
| `AI_MODEL` | Modelo principal utilizado. |
| `AI_MODEL_RESPALDO` | Modelo de respaldo. |
| `AI_API_KEY` | Clave de acceso a la API del proveedor. |
| `JWT_SECRET` | Clave utilizada para validar los tokens JWT. Si no se configura, el código utiliza un valor predeterminado. |

### Configuración de Gemini

La configuración utilizada actualmente es:

```env
AI_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai
AI_MODEL=gemini-2.5-flash
AI_MODEL_RESPALDO=gemini-3.1-flash-lite
AI_API_KEY=tu_clave_de_google_ai_studio
```

La clave de API se obtiene desde Google AI Studio y debe mantenerse privada.

### Alternativa con Ollama

El proyecto también incluye Ollama en Docker Compose, con el modelo `qwen2.5:1.5b`.

Si no se configura la dirección de Gemini, Docker Compose contempla los valores predeterminados para utilizar Ollama:

- URL: `http://ollama:11434/v1`
- Modelo: `qwen2.5:1.5b`
- Clave: `ollama`

Aunque Ollama está incluido en Docker Compose y su contenedor se inicia, cuando se configura Gemini mediante `AI_BASE_URL`, el servicio de IA utiliza Gemini para generar las respuestas.

## Funcionalidades

El asistente dispone de herramientas que permiten consultar información académica y realizar operaciones según el rol del usuario.

### Consultas disponibles

- `ver_mi_plan_de_estudios`: consulta el plan de estudios del alumno.
- `consultar_materia`: obtiene información de una materia.
- `puedo_cursar`: verifica si el alumno cumple los requisitos para cursar una materia.
- `puedo_rendir_final`: verifica si el alumno está habilitado para rendir un examen final.
- `materias_que_puedo_cursar`: consulta las materias habilitadas para cursar.
- `ver_mesas_de_final`: consulta las mesas de examen disponibles.
- `ver_situacion_de_alumno`: consulta la situación académica del alumno.

### Operaciones disponibles

- `inscribirme_a_cursar`: permite realizar una inscripción a una cursada.
- `anotarme_a_final`: permite realizar una inscripción a un examen final.

Las operaciones que modifican información requieren confirmación del usuario antes de ejecutarse.

Las herramientas disponibles dependen del rol y los permisos del usuario autenticado.

## Autenticación

El servicio utiliza tokens JWT para identificar y autenticar a los usuarios.

Las solicitudes protegidas deben incluir el token en el encabezado HTTP:

```http
Authorization: Bearer TOKEN
```

El token recibido se utiliza para validar al usuario y realizar solicitudes a los demás servicios.

Si el token es inválido o está vencido, el servicio devuelve un error de autenticación.

## Endpoints

### GET /

Permite verificar el estado del servicio.

### POST /chat

Procesa un mensaje enviado al asistente.

**Cuerpo de la solicitud:**

```json
{
  "mensaje": "¿Qué materias puedo cursar?",
  "historial": []
}
```

El campo `mensaje` contiene la consulta del usuario y `historial` permite enviar mensajes anteriores de la conversación.

**Respuesta:**

El servicio devuelve información que incluye:

- `respuesta`: texto generado por el asistente.
- `tarjeta`: información adicional para mostrar en la interfaz, cuando corresponde.
- `actualizo`: indica si se realizó una actualización.

El mensaje no puede estar vacío.

## Comunicación con otros servicios

AI Service se comunica con los siguientes componentes:

**Users Service**
- Obtiene información de los usuarios.
- Utiliza el token JWT recibido para realizar solicitudes autenticadas.

**Academic Service**
- Consulta planes de estudio.
- Verifica correlatividades y condiciones académicas.
- Consulta materias y mesas de examen.
- Realiza operaciones de inscripción.

**Proveedor de inteligencia artificial**
- Procesa las consultas en lenguaje natural.
- Interpreta la intención del usuario.
- Selecciona las herramientas disponibles.
- Genera respuestas en español.

## Procesamiento de las consultas

El funcionamiento general del asistente es el siguiente:

1. Recibe el mensaje y el historial de conversación.
2. Valida la autenticación del usuario.
3. Determina las herramientas disponibles según el rol.
4. Procesa la consulta mediante el proveedor de inteligencia artificial configurado.
5. Ejecuta las herramientas necesarias para consultar o modificar información.
6. Devuelve la respuesta al frontend.

El historial conserva los últimos ocho mensajes de usuario y asistente para mantener el contexto de la conversación.

El servicio contempla reintentos ante determinados errores y un tiempo máximo de espera de 30 segundos.

## Manejo de errores

El servicio utiliza los siguientes códigos HTTP:

| Código | Descripción |
|---|---|
| 400 | Solicitud incorrecta o mensaje vacío. |
| 401 | Token inválido o vencido. |
| 503 | Servicio externo no disponible. |
| 500 | Error interno del servidor. |

## Ejecución con Docker

El servicio forma parte de la arquitectura general del proyecto y se inicia mediante Docker Compose desde la raíz del repositorio.

```bash
docker compose up -d --build ai-service
```

Para consultar los registros del servicio:

```bash
docker compose logs -f ai-service
```

Para detener los servicios:

```bash
docker compose down
```

## Ejecución local

Para ejecutar el servicio fuera de Docker:

1. Ingresar a la carpeta `ai-service`.
2. Instalar las dependencias.
3. Configurar las variables de entorno necesarias.
4. Iniciar el servidor.

```bash
npm install
npm start
```

El servicio utiliza el puerto 5000 por defecto.

## Dockerfile

El servicio utiliza una imagen basada en Node.js 20.

El Dockerfile instala las dependencias, copia los archivos de la aplicación, expone el puerto 5000 e inicia el servidor mediante `npm start`.

## Puerto utilizado

| Servicio | Puerto |
|---|---:|
| AI Service | 5000 |
| Ollama | 11434 (interno) |