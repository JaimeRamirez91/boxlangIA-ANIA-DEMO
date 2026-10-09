# Ejemplo de IA con BoxLang

Este proyecto envía el mensaje `Hola` al modelo `glm-5.3-flash` de OpenCode Go mediante `aiChat()` e imprime la respuesta. Utiliza el protocolo compatible con OpenAI y el endpoint `/chat/completions`.

## Requisitos

- [BoxLang](https://boxlang.ortusbooks.com/) instalado y disponible mediante el comando `boxlang`. En [box.json](box.json) se declara BoxLang 1.18.0.
- [CommandBox](https://www.ortussolutions.com/products/commandbox) para instalar las dependencias si todavía no están disponibles.
- Una clave válida de [OpenCode Go](https://opencode.ai/docs/go/) con acceso al modelo seleccionado.

No necesitas iniciar Ollama, LM Studio ni un servidor local: las solicitudes se envían a OpenCode Go.

## Instalar las dependencias

Desde la raíz del proyecto:

```sh
box install
```

[box.json](box.json) declara `bx-ai` versión `3.5.0+19` y su instalación en `lib/boxlang-modules/bx-ai`. Omite este paso si la dependencia ya está instalada.

## Configurar la clave

Define `OPENAI_API_KEY` en el entorno del proceso o en un archivo `.env` en la raíz del proyecto:

```dotenv
OPENAI_API_KEY=tu_clave_de_opencode_go
```

Aunque la variable se llama `OPENAI_API_KEY`, su valor debe ser una clave de **OpenCode Go**, no de OpenAI. El script la lee con `getSystemSetting( "OPENAI_API_KEY" )` y la pasa explícitamente en `options.apiKey`.

No publiques la clave ni añadas el archivo `.env` al repositorio. Si una clave queda expuesta, revócala y genera otra.

## Configuración del modelo

[boxlang.json](boxlang.json) define:

| Opción | Valor actual |
| --- | --- |
| `modules.bxai.settings.provider` | `openai-compatible` |
| `modules.bxai.settings.defaultParams.model` | `glm-5.3-flash` |
| `modules.bxai.settings.providers.OpenAICompatible.options.baseURL` | `https://opencode.ai/zen/go/v1` |
| `modulesDirectory` | `[ "${user-dir}/lib/boxlang-modules" ]` |

El adaptador añade `/chat/completions` a la URL base. La URL final es `https://opencode.ai/zen/go/v1/chat/completions`.

`openai-compatible` selecciona el proveedor; `OpenAICompatible` identifica la configuración de ese servicio. Para cambiar de modelo, edita `defaultParams.model` y comprueba que el nuevo modelo admita el mismo protocolo en los [endpoints de OpenCode Go](https://opencode.ai/docs/go/#endpoints).

## Funcionamiento del script

[test-ia.bxs](test-ia.bxs) realiza los siguientes pasos:

1. Obtiene la configuración cargada con `getModuleInfo( "bxai" ).settings`.
2. Llama a `aiChat()` con el mensaje `Hola` y `params = moduleSettings.defaultParams`. Pasar estos parámetros explícitamente evita que los valores predeterminados del adaptador sustituyan al modelo configurado.
3. Proporciona la clave mediante `options.apiKey` y las cabeceras `User-Agent: boxlang-ia/1.0` y `x-opencode-session`.
4. Imprime la respuesta con `println( answer )`.

El proveedor y la URL se toman de la configuración; el script no necesita crear un servicio con `aiService()` ni llamar a `setChatURL()`.

El identificador de sesión se genera con `createUUID()` para esta solicitud. En una conversación con varias solicitudes, reutiliza el mismo identificador. OpenCode Go está orientado a agentes de programación; consulta sus [requisitos para clientes externos](https://opencode.ai/docs/go/#where-can-i-use-it).

## Ejecutar

Desde la raíz del proyecto:

```sh
boxlang --bx-config ./boxlang.json test-ia.bxs
```

Una respuesta de ejemplo es:

```text
¡Hola! ¿Cómo estás? ¿En qué puedo ayudarte hoy?
```

La respuesta puede variar. Modifica `messages` en el script para cambiar la consulta.

## Solución de problemas

- **HTTP 404**: verifica el proveedor y la URL base. El adaptador `ollama` añade `/api/chat`, una ruta que no corresponde a esta API de OpenCode Go.
- **`Invalid API key`**: comprueba que `OPENAI_API_KEY` contenga una clave válida de OpenCode Go y que se pase mediante `options.apiKey`. La configuración actual no almacena la clave en el JSON.
- **`ModelProtocolUnsupported`**: confirma que el modelo admita `/chat/completions`. Los modelos que requieren `/messages` o `/responses` necesitan otro adaptador o implementación; cambiar solo el nombre del modelo no basta.
- **Modelo incorrecto o no disponible**: conserva `params = moduleSettings.defaultParams`, verifica el ID en `https://opencode.ai/zen/go/v1/models` y comprueba su disponibilidad en el servicio.
- **No se encuentra `aiChat`**: instala las dependencias con `box install` y ejecuta desde la raíz del proyecto usando `--bx-config ./boxlang.json`.