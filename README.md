# Ejemplo de IA con BoxLang

Un ejemplo mínimo que envía `Say hello!` a un modelo local mediante el módulo `bx-ai` e imprime su respuesta.

## Boxlang IA docs: `https://ai.ortusbooks.com/` `https://ai.boxlang.io/explorer/`

## Requisitos previos

- [BoxLang](https://boxlang.ortusbooks.com/) instalado, con el comando `boxlang` disponible en la terminal. Este proyecto está configurado para BoxLang 1.18.0.
- [CommandBox](https://www.ortussolutions.com/products/commandbox) instalado si necesitas instalar las dependencias del proyecto.
- [LM Studio Bionic](https://lmstudio.ai/) instalado con un modelo de chat descargado. Puedes descargar la aplicación desde el sitio oficial. Este ejemplo está configurado para `google/gemma4-12b`.

## 1. Instalar las dependencias

Abre una terminal en la raíz del proyecto, donde se encuentra [box.json](box.json):

```sh
box install
```

Este comando instala la dependencia declarada `bx-ai` en `lib/boxlang-modules/bx-ai`. Omite este paso si el módulo ya está instalado.

## 2. Iniciar Bionic

Referencias oficiales: [documentación de Bionic](https://lmstudio.ai/docs/bionic) y [configuración del servidor de API local](https://lmstudio.ai/docs/developer/core/server).

En Bionic:

1. Descarga y carga el modelo que quieras utilizar.
2. Abre la sección Developer y configura el servidor de API para usar el puerto `1235`.
3. Inicia el servidor.


Verifica que el servidor responda:

```sh
http://localhost:1235/v1/models
```

La respuesta debe ser un JSON con los modelos disponibles. Comprueba que el `id` del modelo coincida con el valor de la configuración indicada a continuación.

## 3. Revisar la configuración

El archivo [boxlang.json](boxlang.json) contiene la siguiente configuración:

- `modules.bxai.settings.provider`: `openai-compatible`.
- `modules.bxai.settings.defaultParams.model`: `google/gemma4-12b`. Sustitúyelo por el ID exacto de tu modelo si es necesario.
- `modules.bxai.settings.providers.OpenAICompatible.options.baseURL`: `http://localhost:1235/v1`. Cámbialo si tu servidor usa otro puerto.
- `modulesDirectory`: el directorio `lib/boxlang-modules` del proyecto.

Los nombres `openai-compatible` y `OpenAICompatible` son diferentes intencionalmente: el primero selecciona el proveedor y el segundo es el nombre del servicio que se utiliza para buscar su configuración. No sustituyas ninguno por `local`. Aunque el modelo se ejecute localmente en Bionic, utiliza el formato de API compatible con OpenAI.

El valor vacío de `apiKey` es adecuado para un servidor local con la autenticación desactivada. No necesitas una cuenta de OpenAI.

## 4. Ejecutar el ejemplo

Desde la raíz del proyecto:

```sh
boxlang --bx-config ./boxlang.json test-ia.bxs
```

[test-ia.bxs](test-ia.bxs) llama a `aiChat( "Say hello!" )` e imprime la respuesta. Una ejecución correcta produce un texto similar a:

```text
Hello! How can I help you today?
```

La respuesta puede variar entre ejecuciones. Edita el mensaje del script para probar otra instrucción.

## Solución de problemas

- **`Failed to parse JSON Connection Failure`**: comprueba que el servidor de API de Bionic esté iniciado, que el puerto coincida con la configuración y que la consulta a `/v1/models` responda correctamente. Verifica también que la clave de opciones del proveedor sea `OpenAICompatible`; de lo contrario, no se aplica la URL configurada.
- **`Provider [local] is not supported`**: establece `settings.provider` en `openai-compatible`.
- **Modelo no encontrado**: utiliza el ID exacto devuelto por `/v1/models` y asegúrate de que el modelo esté disponible en Bionic.
- **No se encuentra `aiChat`**: ejecuta `box install` y después ejecuta el script desde la raíz del proyecto con `--bx-config ./boxlang.json`.