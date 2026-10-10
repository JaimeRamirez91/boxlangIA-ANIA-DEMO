# Central America Weather With BoxLang AI

A BoxLang web application that requests current weather from OpenWeather and asks an AI model to summarize it in Spanish. Select a country and submit the form to request conditions for its capital.

Supported countries: Guatemala, Belize, El Salvador, Honduras, Nicaragua, Costa Rica, and Panama. Results concern the capital's coordinates, not the whole country, and represent current conditions rather than a future forecast.

## Requirements

- [CommandBox](https://www.ortussolutions.com/products/commandbox), available as `box`.
- A Java runtime compatible with CommandBox and BoxLang. Java 21 was used for the local server.
- An [OpenCode Go](https://opencode.ai/docs/go/) API key with access to the configured model and tool calling.
- An [OpenWeather](https://openweathermap.org/api) API key with access to the Current Weather API.
- Internet access for dependency installation and the external APIs.

CommandBox installs the web engine specified in [server.json](server.json). A separate BoxLang CLI installation is not required for this workflow.

## Setup

Run all commands from the project root.

### Install Dependencies

```sh
box install
```

[box.json](box.json) declares `bx-ai` version `3.5.0+19`, installed under `lib/boxlang-modules/bx-ai`.

### Configure API Keys

Set these environment variables in the terminal that starts CommandBox:

```sh
export API_KEY="your-opencode-go-key"
export WEATHER_API_KEY="your-openweather-key"
```

Use your own credentials locally; do not commit them or share terminal history containing them. [env-example](env-example) lists example variable names, but copying that file alone does not ensure the server loads its values.

`API_KEY` is passed explicitly to the AI provider. Although the example also includes `OPENAI_API_KEY`, the current weather service reads `API_KEY`, not `OPENAI_API_KEY`.

### Start the Server

```sh
box server start
```

Open [http://127.0.0.1:50804/index.bxm](http://127.0.0.1:50804/index.bxm), select a country, and click **Consultar**.

Server management:

```sh
box server info
box server restart
box server stop
```

Restart after changing runtime configuration. If you change exported environment variables, stop the server and start it again from the terminal containing the new values.

## Configuration

| File | Responsibility |
| --- | --- |
| [box.json](box.json) | Package metadata and dependencies |
| [server.json](server.json) | CommandBox engine, web root, host, and port |
| [boxlang.json](boxlang.json) | BoxLang module discovery and AI provider settings |
| [index.bxm](index.bxm) | Country selection, form processing, and response rendering |
| [models/ElClimaService.bx](models/ElClimaService.bx) | Country coordinates, weather tool, and AI request |
| [styles.css](styles.css) | Page styling |
| [env-example](env-example) | Example API key variable names |

[server.json](server.json) requests `boxlang@1.18.6+62`, stores the engine under `.engine/boxlang`, and serves the project root on `127.0.0.1:50804`. Check startup logs for the actual runtime version installed by the engine package.

[boxlang.json](boxlang.json) selects:

| Setting | Value |
| --- | --- |
| Provider | `openai-compatible` |
| Model | `glm-5.3-flash` |
| Base URL | `https://opencode.ai/zen/go/v1` |
| Module directory | `${user-dir}/lib/boxlang-modules` |

The AI adapter sends requests to `/chat/completions`. When changing models, verify support for that protocol and tool calling. No local AI model server is needed; the web server is still required to execute the application.

## Request Flow

1. The page loads the country list from `getCountries()`.
2. A submitted country is validated against that list.
3. `getWeatherIAResponse()` registers the `get_weather` tool with `aiTool()` and supplies it through `params.tools` to `aiChat()`.
4. When the model calls the tool, `getWeatherData()` queries OpenWeather using the capital's latitude and longitude, metric units, and Spanish language settings.
5. The model is instructed to summarize city, temperature in Celsius, humidity, and wind speed in meters per second.
6. The page HTML-escapes the returned text before displaying it.

If the tool returns no weather `main` data, the service returns a no-data message. External API exceptions are not currently caught by the page and may produce an HTTP 500 response.

## Troubleshooting

- **Connection refused:** run `box server info` and start the server if stopped. Verify the configured port is available.
- **Not found or the wrong engine:** engine settings belong in [server.json](server.json), not [box.json](box.json). Confirm `box server info` reports BoxLang rather than Lucee, and use `/index.bxm`.
- **Class not found:** BoxLang classes use `.bx`. Keep the weather class at [models/ElClimaService.bx](models/ElClimaService.bx), matching the page's import; `.bxs` is for scripts.
- **`aiChat` or `bxai` not found:** run `box install` and confirm module discovery points to the project's `lib/boxlang-modules` directory. If `${user-dir}` resolves elsewhere in the servlet runtime, set `modulesDirectory` to the absolute project module path and restart.
- **Invalid AI key:** verify `API_KEY` contains an OpenCode Go credential and is available to the server process.
- **No current weather data:** verify `WEATHER_API_KEY`, Current Weather API access, and the model's tool-calling support. OpenWeather errors return an error structure instead of weather data.
- **Model unavailable or unsupported protocol:** check the configured model's availability and compatibility with `/chat/completions` in the OpenCode Go documentation.

This repository currently has no automated test suite. Loading the page checks rendering; submitting a country additionally requires both external APIs and valid credentials.