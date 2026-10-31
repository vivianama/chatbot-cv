Chatbot CV · Preguntale al asistente de Vivi
Un chatbot gratuito, embebido en una página web, para que reclutadores y equipos de RRHH le hagan preguntas al perfil profesional de Viviana Acevedo en lugar de leer un CV estático.
Demo: https://vivianama-cv.netlify.app
Problema que resuelve
Un CV en PDF se lee en 30 segundos y no responde dudas. Este proyecto permite que una persona de RRHH pregunte lo que le interesa (trayectoria, formación, certificaciones, herramientas) y obtenga respuestas basadas únicamente en un documento de perfil controlado por su autora.
También es un proyecto de portfolio: muestra cómo llevar un caso de uso de IA generativa de la idea a un despliegue público, con costo cero.
Cómo funciona
```
Navegador (index.html)
        │  POST /api/chat  { messages: [...] }
        ▼
Netlify Function (netlify/functions/chat.mjs)
        │  valida el historial + instrucciones del sistema + perfil.mjs
        ▼
Gemini API (generateContent)
        │
        ▼
{ reply } → Navegador
```
Frontend (`index.html`): una página estática con la interfaz de chat. No contiene credenciales y muestra las respuestas como texto plano (`textContent`), no como HTML.
Backend (`netlify/functions/chat.mjs`): recibe el historial de la conversación, lo valida, lo combina con las instrucciones del sistema y el perfil, y llama a la API de Gemini. Expone la ruta `/api/chat`.
Base de conocimiento (`perfil.mjs`): un módulo que exporta el texto del perfil (trayectoria, formación, hobbies, contacto público). Es lo único que el asistente puede usar para responder.
Hosting: Netlify (plan gratuito). Modelo: Gemini API, configurable por variable de entorno.
Controles incluidos en la Function
Solo acepta `POST`.
Cada mensaje se recorta a 500 caracteres y solo se envían los últimos 10 mensajes del historial.
Se aceptan únicamente los roles `user` y `model`; la conversación debe terminar en un mensaje del usuario.
Temperatura baja (0.3) y máximo de 600 tokens de salida.
Reintento único ante `503` y mensaje amable ante `429` (cuota agotada).
Instrucciones del sistema que limitan el tema al perfil, prohíben inventar datos, protegen datos sensibles (sueldo, DNI, dirección) e ignoran pedidos de cambiar las reglas.
Cómo correrlo localmente
Requisitos: Node.js, Netlify CLI y una API key de Gemini (Google AI Studio).
```bash
git clone https://github.com/vivianama/chatbot-cv.git
cd chatbot-cv
npm install -g netlify-cli
export GEMINI_API_KEY="tu_clave"
export GEMINI_MODEL="nombre-del-modelo"   # el modelo de Gemini que quieras usar
netlify dev
```
Abrí la URL local que muestra `netlify dev`.
Para desplegar tu propia copia:
Conectá el repo a Netlify.
Cargá `GEMINI_API_KEY` y `GEMINI_MODEL` en Site configuration → Environment variables.
Reemplazá el contenido de `perfil.mjs` por tu propia información.
Estructura del repo
```
chatbot-cv/
├── index.html                  # interfaz de chat
├── perfil.mjs                  # base de conocimiento (editá con tu información)
├── netlify.toml                # configuración de Netlify
└── netlify/
    └── functions/
        └── chat.mjs            # Function: valida, arma el prompt y llama a Gemini
```
Decisiones de diseño
Todo el perfil en el prompt, sin RAG: el documento es corto, así que es más simple, barato y fácil de mantener.
Function como intermediario: la API key vive solo del lado del servidor y permite agregar validaciones sin tocar el frontend.
Stack gratuito de punta a punta, para que cualquiera pueda replicarlo.
Seguridad y privacidad
La API key y el modelo se leen de variables de entorno; no están en el repo ni en el navegador.
`perfil.mjs` es público (está en este repo): solo incluye información que su autora decidió publicar. No incluye datos internos de ningún empleador ni datos de contacto privados.
El asistente rechaza pedidos fuera de tema y los intentos de modificar sus instrucciones, pero ninguna defensa contra prompt injection es absoluta.
Limitaciones
Responde solo con lo que está en `perfil.mjs`; si falta algo, debería decir que no lo sabe, pero un modelo de lenguaje puede equivocarse. La propia interfaz lo advierte.
El nivel gratuito de Gemini tiene límites de uso: con mucho tráfico el asistente puede dejar de responder temporalmente.
No hay límite de consultas por IP.
Responde en el idioma en que le escriben (español o inglés).
Próximas mejoras
Límite de consultas por IP.
Conjunto de preguntas de prueba (incluidas adversariales) para validar el comportamiento antes de cada cambio.
Servir solo los archivos públicos desde una carpeta dedicada, en lugar de publicar la raíz del repo.
Respuestas en streaming.
Medir qué preguntas hacen los visitantes (sin datos personales) para mejorar el perfil.
Autora
Viviana Acevedo · Product Manager | Project Manager de IT | Data & AI | Automatización
LinkedIn: https://www.linkedin.com/in/vivianaacevedo/
