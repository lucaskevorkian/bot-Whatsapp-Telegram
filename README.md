# Asistente Autónomo de Ventas con IA (n8n + Telegram + Ollama)

Sistema autónomo de atención, cotización de envíos y agendamiento de visitas a showroom implementado sobre **n8n**, interactuando a través de **Telegram**, ejecutando inferencia local mediante **Ollama** y persistiendo datos en **Google Sheets** y **Google Calendar**.

---

## Arquitectura y Seguridad
El flujo aplica un esquema de defensa en capas con pre-filtrado determinístico:
1. **Detección Determinística Previa:** Un nodo de código analiza los mensajes entrantes para bloquear intentos de prompt injection / jailbreak y desviar solicitudes de atención personalizada sin consumir tokens de inferencia.
2. **Capa Agéntica (AI Agent):** Ejecuta sobre Ollama con memoria contextual aislada por ID de chat (`chatId`).
3. **Herramientas (Tools):**
   - **Google Sheets:** Consulta de catálogo de productos en modo de solo lectura.
   - **Cálculo de Envíos:** Función de lógica fija para estimar costos de logística.
   - **Google Calendar:** Verificación y creación de reservas para visitas presenciales.

---

## Requisitos Previos
- [Docker Desktop](https://www.docker.com/) instalado y en ejecución.
- Token de bot de Telegram obtenido desde [@BotFather](https://t.me/botfather).
- Cuenta de Google Cloud con APIs de Google Sheets y Google Calendar habilitadas (o credenciales OAuth configuradas en n8n).
- Modelo LLM corriendo localmente en Ollama.

---

## Puesta en Marcha

### 1. Iniciar el túnel HTTPS (Cloudflare)
Telegram exige un webhook bajo protocolo HTTPS. En una terminal, ejecutá:
```powershell
docker run --rm cloudflare/cloudflared:latest tunnel --no-autoupdate --url http://host.docker.internal:5678
```
Copiá la URL generada (ejemplo: `https://xxxx-xxxx.trycloudflare.com`). **No cierres esta terminal**.

---

### 2. Levantar el contenedor de n8n
En otra terminal, ejecutá el contenedor asignando la URL del túnel en la variable `WEBHOOK_URL`:
```powershell
docker run -d --name n8n_telegram -p 5678:5678 -e WEBHOOK_URL="https://TU-URL-DE-CLOUDFLARE.trycloudflare.com/" -e N8N_DEFAULT_BINARY_DATA_MODE=filesystem -v n8n_data:/home/node/.n8n n8nio/n8n:latest
```

---

### 3. Importar y Configurar el Workflow
1. Abrí tu navegador e ingresá a la URL generada por Cloudflare (o en `http://localhost:5678`).
2. Creá un flujo nuevo y seleccioná **Import from File...** para cargar el archivo `workflow.json`.
3. Configurá las credenciales requeridas:
   - **Telegram API:** Pegá el token provisto por `@BotFather`.
   - **Google Sheets / Calendar:** Vinculá tus cuentas de servicio u OAuth.
   - **Ollama:** Apuntá a tu endpoint local (`http://host.docker.internal:11434`).
4. Activá el switch superior a **Published** para habilitar la atención automática continua.