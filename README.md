# Documentación del Proyecto – Automatización de Registro de Experiencias

Antes de empezar le explicare porque subi un ZIP, y es que mi compañero tambien subio su programa (que es el mismo programa base) y por eso git me dio error al subirlo, intente de muchas formas incluido 
enviar por consola y eliminar el .git y nada, asi que de ultima opcion decidi subirlo como formato zip, disculpe si eso es un inconvenie :(

el proceso quedaria nada mas en descomprimir la carpeta y en teoria mi programa deberia de funcionar.

aca esta el link hacia el video de la fase 1 de mi proyecto, la cual cuenta con mi parte del video: https://drive.google.com/file/d/137fpFyJzhM4QwxmABIzXgdrgKAlfsdeY/view?usp=sharing

1. Descripción general del proyecto

Este proyecto tiene como objetivo automatizar el registro de experiencias desde una aplicación Android a través de un webhook en n8n, que procesa los datos y envía un correo con la información recopilada.
Aunque inicialmente se planeó integrar Google Calendar, se optó por enviar los eventos por correo electrónico usando Gmail, ya que Calendar presentaba problemas con el manejo de los datos.

2. Herramientas utilizadas

n8n: Plataforma de automatización de flujos.

Railway.app: Despliegue del flujo y conexión con n8n.

Gmail: Servicio utilizado para el envío automático de correos.

Google Calendar: Intentado inicialmente, luego descartado por incompatibilidad.

Android Studio: Desarrollo de la aplicación móvil.

Copilot (GitHub) y ChatGPT: Asistentes de código y resolución de errores.

3. Estructura del flujo en n8n

El flujo completo consta de los siguientes nodos:

Webhook

Recibe los datos desde la aplicación móvil mediante una solicitud POST.

URL: https://primary-production.../webhook/task-created

Code in JavaScript

Convierte y formatea los datos recibidos, especialmente las fechas.

Limpia los valores nulos (por ejemplo, title, description, location) y los convierte en "".

Edit Fields (Set)

Prepara el contenido final a utilizar (asignando texto como "Sin título" si está vacío).

Campos preparados: title, description, location, startDate, endDate.

Send a message (Gmail)

Envía un correo con la información estructurada al correo del usuario.

Respond to Webhook

Devuelve una respuesta al cliente confirmando la ejecución.

📸 (Agrega aquí una captura del flujo completo de nodos en n8n como evidencia del diseño final.)

4. Problemas encontrados
🧩 Problema 1: Google Calendar no aceptaba los datos

Al intentar usar el nodo "Create an Event (Google Calendar)", no se creaba ningún evento.

Las fechas se enviaban correctamente, pero el nodo requería campos estrictos en formato ISO.

Se intentó convertir los datos con new Date().toISOString(), pero no se resolvía el problema.

Solución:
Se decidió desechar temporalmente el uso de Calendar y usar Gmail como canal alternativo.

🧩 Problema 2: Los campos aparecían vacíos en el correo

A pesar de que la app enviaba campos (title, description, etc.), el correo mostraba contenido como:

Título: ``

Descripción: ``

Ubicación: ``

Causa:
La app enviaba los datos con valores forzados como "Sin título" o simplemente una comilla invertida ` para evitar errores, pero el flujo no los interpretaba correctamente.

Solución parcial:

Se revisó el nodo Code in JavaScript para que conservara los datos originales si existían, o los convirtiera en "".
Problema 3: Error de sintaxis en el nodo de código

Se intentó usar expresiones como {{ $json.title }} sin respetar la sintaxis de n8n.

Aparecían errores como:

ERROR: invalid syntax

Expression expected

Solución:
Se corrigieron los campos usando la sintaxis adecuada de n8n, validando que las variables existieran antes de usarlas para evitar errores de interpretación.

🧩 Problema 4: Error de incompatibilidad por usar una API de Android antigua

Durante las pruebas iniciales, la aplicación fue desarrollada sobre la API 24 (Android 7.0).
Al enviarse los datos desde la app al webhook, el entorno de n8n en Railway comenzó a devolver errores silenciosos o desconexiones sin respuesta clara.

Causa:
n8n requiere una comunicación moderna basada en estándares actuales de TLS y encabezados HTTPS, lo cual puede fallar en versiones antiguas de Android.
La API 24 no ofrecía total compatibilidad con las configuraciones de seguridad exigidas por Railway/n8n.

Síntomas observados:

n8n no recibía el payload aunque la app indicaba que se enviaba correctamente.

El webhook no se activaba.

Railway mostraba tráfico pero no ejecutaba el flujo.

Solución:
Se actualizó la versión objetivo de la app desde API 24 a API 31 (Android 12), lo que corrigió la comunicación y permitió:

El envío correcto de los datos.

La activación del webhook sin errores.

La ejecución completa del flujo.

📸 (Agrega aquí capturas del cambio de versión en Android Studio y la ejecución exitosa posterior.)
