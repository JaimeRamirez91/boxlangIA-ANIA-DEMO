# El clima en Centroamérica con BoxLang AI

Aplicación web en BoxLang que consulta el clima actual en OpenWeather y utiliza un modelo de inteligencia artificial para resumirlo en español. Selecciona un país y envía el formulario para consultar las condiciones de su capital.

Países disponibles: Guatemala, Belice, El Salvador, Honduras, Nicaragua, Costa Rica y Panamá. Los resultados corresponden a las coordenadas de la capital, no a todo el país, y describen condiciones actuales, no un pronóstico futuro.

## Requisitos

- [CommandBox](https://www.ortussolutions.com/products/commandbox), disponible mediante el comando `box`.
- Un entorno de ejecución de Java compatible con CommandBox y BoxLang. Para el servidor local se utilizó Java 21.
- Una clave de API de [OpenCode Go](https://opencode.ai/docs/go/) con acceso al modelo configurado y llamadas a herramientas.
- Una clave de API de [OpenWeather](https://openweathermap.org/api) con acceso a la API de clima actual (Current Weather API).
- Conexión a Internet para instalar las dependencias y consultar las API externas.

CommandBox instala el motor web especificado en [server.json](server.json). Para este flujo no es necesario instalar por separado la CLI de BoxLang.

## Instalación

Ejecuta todos los comandos desde la raíz del proyecto.

### Instalar las dependencias

```sh
box install
```

[box.json](box.json) declara `bx-ai` versión `3.5.0+19`, que se instala en `lib/boxlang-modules/bx-ai`.

### Configurar las claves de API

Define estas variables de entorno en la terminal desde la que iniciarás CommandBox:

```sh
export API_KEY="tu-clave-de-opencode-go"
export WEATHER_API_KEY="tu-clave-de-openweather"
```

Utiliza tus propias credenciales localmente; no las incluyas en el repositorio ni compartas un historial de terminal que las contenga. [env-example](env-example) muestra nombres de variables de ejemplo, pero copiar ese archivo no garantiza que el servidor cargue sus valores.

`API_KEY` se pasa explícitamente al proveedor de IA. Aunque el ejemplo también incluye `OPENAI_API_KEY`, el servicio de clima actual lee `API_KEY`, no `OPENAI_API_KEY`.

### Iniciar el servidor

```sh
box server start
```

Abre [http://127.0.0.1:50804/index.bxm](http://127.0.0.1:50804/index.bxm), selecciona un país y pulsa **Consultar**.

Comandos para administrar el servidor:

```sh
box server info
box server restart
box server stop
```

Reinicia el servidor después de cambiar la configuración del entorno de ejecución. Si modificas las variables de entorno exportadas, detén el servidor y vuelve a iniciarlo desde la terminal que contiene los nuevos valores.

## Configuración

| Archivo | Función |
| --- | --- |
| [box.json](box.json) | Metadatos del paquete y dependencias |
| [server.json](server.json) | Motor de CommandBox, raíz web, dirección y puerto |
| [boxlang.json](boxlang.json) | Localización de módulos de BoxLang y configuración del proveedor de IA |
| [index.bxm](index.bxm) | Selección del país, procesamiento del formulario y presentación de la respuesta |
| [models/ElClimaService.bx](models/ElClimaService.bx) | Coordenadas de los países, herramienta de clima y solicitud a la IA |
| [styles.css](styles.css) | Estilos de la página |
| [env-example](env-example) | Nombres de ejemplo para las variables de claves de API |

[server.json](server.json) solicita `boxlang@1.18.6+62`, almacena el motor en `.engine/boxlang` y sirve la raíz del proyecto en `127.0.0.1:50804`. Consulta los registros de inicio para conocer la versión real del entorno de ejecución instalada por el paquete del motor.

[boxlang.json](boxlang.json) define:

| Opción | Valor |
| --- | --- |
| Proveedor | `openai-compatible` |
| Modelo | `glm-5.3-flash` |
| Base URL | `https://opencode.ai/zen/go/v1` |
| Directorio de módulos | `${user-dir}/lib/boxlang-modules` |

El adaptador de IA envía las solicitudes a `/chat/completions`. Al cambiar de modelo, comprueba que admita ese protocolo y llamadas a herramientas. No se necesita un servidor local de modelos de IA; sí es necesario el servidor web para ejecutar la aplicación.

## Flujo de la consulta

1. La página carga la lista de países mediante `getCountries()`.
2. El país enviado se valida contra esa lista.
3. `getWeatherIAResponse()` registra la herramienta `get_weather` con `aiTool()` y la pasa a `aiChat()` mediante `params.tools`.
4. Cuando el modelo llama a la herramienta, `getWeatherData()` consulta OpenWeather con la latitud y longitud de la capital, unidades métricas e idioma español.
5. Se indica al modelo que resuma la ciudad, la temperatura en grados Celsius, la humedad y la velocidad del viento en metros por segundo.
6. La página escapa los caracteres HTML del texto recibido antes de mostrarlo.

Si la herramienta no devuelve datos de clima en `main`, el servicio devuelve un mensaje de ausencia de datos. Actualmente, la página no captura las excepciones de las API externas, por lo que pueden producir una respuesta HTTP 500.

## Solución de problemas

- **Conexión rechazada:** ejecuta `box server info` e inicia el servidor si está detenido. Comprueba que el puerto configurado esté disponible.
- **Página no encontrada o motor incorrecto:** la configuración del motor debe estar en [server.json](server.json), no en [box.json](box.json). Confirma que `box server info` indique BoxLang en lugar de Lucee y utiliza `/index.bxm`.
- **Clase no encontrada:** las clases de BoxLang utilizan `.bx`. Mantén la clase de clima en [models/ElClimaService.bx](models/ElClimaService.bx), de acuerdo con la importación de la página; `.bxs` corresponde a scripts.
- **No se encuentra `aiChat` o `bxai`:** ejecuta `box install` y confirma que la búsqueda de módulos apunte al directorio `lib/boxlang-modules` del proyecto. Si `${user-dir}` se resuelve en otra ubicación durante la ejecución del servlet, configura `modulesDirectory` con la ruta absoluta del directorio de módulos del proyecto y reinicia.
- **Clave de IA no válida:** comprueba que `API_KEY` contenga una credencial de OpenCode Go y esté disponible para el proceso del servidor.
- **Sin datos actuales del clima:** comprueba `WEATHER_API_KEY`, el acceso a la API de clima actual y la compatibilidad del modelo con llamadas a herramientas. Los errores de OpenWeather devuelven una estructura de error en lugar de datos meteorológicos.
- **Modelo no disponible o protocolo no compatible:** consulta en la documentación de OpenCode Go la disponibilidad del modelo configurado y su compatibilidad con `/chat/completions`.

Este repositorio no dispone actualmente de una suite de pruebas automatizadas. Cargar la página permite comprobar su presentación; enviar un país requiere además ambas API externas y credenciales válidas.