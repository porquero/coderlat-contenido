---
title: 'TTS Sidecar: voz clonada en español latino, 100% local'
date: '19-10-2026 18:34'
author: CristianRojas-SoftwareEngineer
ia: asistido
taxonomy:
    tool: [claude]
    resource: [herramienta]
    country: [chile]
metadata:
    description: 'Síntesis de voz 100% local con clonación de voz en español latinoamericano, usable por CLI desde cualquier lenguaje de programación.'
faq:
    -
        q: '¿Cómo clono mi voz en español con TTS Sidecar?'
        a: 'Con unos 10 segundos de audio en español y dos archivos de referencia (timbre y condicionamiento) ejecutas `tts-sidecar voice clone --name mi_voz --reference timbre.wav --speech condicion.wav`. Después sintetizas con `tts-sidecar speak --text "Hola mundo" -v mi_voz`.'
    -
        q: '¿Se puede generar voz con IA sin conexión a internet ni APIs externas?'
        a: 'Sí. TTS Sidecar funciona de forma local y sin APIs externas. Solo necesitas descargar el modelo una vez con `tts-sidecar setup`, que lo guarda en la caché de HuggingFace de tu usuario.'
    -
        q: '¿En qué sistemas operativos funciona TTS Sidecar?'
        a: 'En Windows x64, Linux x64 y ARM64, y macOS con Apple Silicon (ARM64). Los Mac con procesador Intel no están soportados.'
    -
        q: '¿Puedo clonar la voz de otra persona con TTS Sidecar?'
        a: 'Solo con el permiso explícito de la persona titular de la voz. El audio generado no lleva marca de agua, así que el proyecto pide indicar que es sintetizado cuando lo publiques o compartas.'
sources:
    -
        title: 'TTS Sidecar (repositorio en GitHub)'
        url: 'https://github.com/CristianRojas-SoftwareEngineer/TTS-Sidecar'
---

TTS Sidecar convierte texto en voz, incluso con tu propia voz clonada, en español latinoamericano. Todo ocurre en tu equipo y sin APIs externas. Se maneja desde la línea de comandos, por lo que cualquier lenguaje de programación puede invocarlo.

Es útil para quienes desarrollan software que necesita voz sintetizada y para quienes quieren generar audio con su propia voz.

## Qué ofrece

- **Motor**: Chatterbox Multilingual V3, con el modelo `es-mx-latam` (licencia MIT) y más de 23 idiomas.
- **Clonación de voz**: usa unos 10 segundos de audio de referencia.
- **100% offline**: sin APIs externas ni conexiones a internet.
- **CLI universal**: se invoca desde cualquier lenguaje.
- **Audio nativo**: usa las APIs de audio del sistema operativo.
- **Instalador por plataforma**: despliega un bundle de PyInstaller `--onedir` (carpeta de la aplicación).

## Plataformas y canales de instalación

| Plataforma | Formato del binario | Nota |
|---|---|---|
| Windows x64 | Instalador `.exe` | Instala en tu perfil, sin permisos de administrador |
| Linux x64 / ARM64 | AppImage | Requiere glibc 2.35 o superior (Ubuntu 22.04+, Debian 12+, Fedora 36+) |
| macOS ARM64 (Apple Silicon) | `.dmg` | Mac Intel no está soportado |

El proyecto se distribuye por dos canales:

| Canal | Audiencia | Requisito |
|---|---|---|
| Binario precompilado | Público general | Ninguno, no necesitas Python |
| Paquete PyPI | Audiencia técnica | Python 3.13 o superior |


## Instalación en una línea

Los scripts descargan el último Release, verifican el checksum contra `SHA256SUMS.txt` y ejecutan `setup`. Si el checksum no coincide, abortan la instalación.

**Linux**

```
curl -fsSL https://raw.githubusercontent.com/CristianRojas-SoftwareEngineer/TTS-Sidecar/main/install-linux.sh | sh
```

**macOS (Apple Silicon)**

```
curl -fsSL https://raw.githubusercontent.com/CristianRojas-SoftwareEngineer/TTS-Sidecar/main/install-macos.sh | sh
```

**Windows (PowerShell)**

```
irm https://raw.githubusercontent.com/CristianRojas-SoftwareEngineer/TTS-Sidecar/main/install-windows.ps1 | iex
```

Si usas Homebrew en macOS, existe un Cask propio:

```
brew tap CristianRojas-SoftwareEngineer/tts-sidecar
brew install --cask tts-sidecar
```

El Cask exige Homebrew y no descarga el modelo, así que después debes correr `tts-sidecar setup`.

## Otras formas de instalar

**Desde PyPI** (requiere Python 3.13+):

```
uv tool install tts-sidecar
# o: pipx install tts-sidecar

tts-sidecar setup
tts-sidecar speak --text "Hola mundo"
```

En Linux, `sounddevice` necesita la librería `libportaudio2` para reproducir audio. Si solo guardas a archivo con `speak --output`, no la necesitas.

**Desde el código fuente:**

```
pip install -r requirements.txt

python scripts/build_windows.py    # Windows
python scripts/build_linux.py      # Linux
python scripts/build_macos.py      # macOS
```

## Descarga del modelo con `setup`

El modelo `es-mx-latam` ocupa varios cientos de MB y viene aparte del ejecutable. Se descarga una sola vez a la caché de HuggingFace de tu usuario (`~/.cache/huggingface/hub`).

- **Windows**: el instalador ofrece una casilla final que ejecuta `setup` por ti.
- **Linux y macOS**: ejecuta `tts-sidecar setup` después de instalar.

`setup` corre los chequeos de entorno y descarga el modelo solo si falta. Mientras el modelo no esté disponible, `speak` y `daemon start` fallan de inmediato y te remiten a `tts-sidecar setup`.

## Primer arranque: avisos de SmartScreen y Gatekeeper

Si descargas el instalador desde el navegador, el sistema puede mostrar una advertencia de «editor desconocido». Es esperable porque los binarios aún no están firmados con certificado de código. El proyecto prevé firmarlos con SignPath Foundation en una versión futura.

- **Windows (SmartScreen)**: pulsa **Más información** y luego **Ejecutar de todas formas**.
- **macOS (Gatekeeper)**: haz clic derecho sobre el `.app` o `.dmg`, elige **Abrir** y confirma.

Para verificar el archivo de forma objetiva, compara su SHA-256 con el `SHA256SUMS.txt` del Release. Los instaladores de una línea y el canal PyPI no disparan estos avisos.

## Uso rápido

**Clonar tu voz.** Necesitas dos archivos: uno de referencia (timbre) y otro de condicionamiento (speech).

```
tts-sidecar voice clone --name mi_voz --reference timbre.wav --speech condicion.wav
tts-sidecar speak --text "Hola mundo" -v mi_voz
tts-sidecar speak --text "Hola mundo" -v mi_voz --output audio.wav
```

**Síntesis básica.** Sin indicar voz, `speak` usa la voz de fábrica `default`, que viene empaquetada y es de solo lectura.

```
tts-sidecar speak --text "Hola mundo"
tts-sidecar speak --text "Hola mundo" --voice mi_voz
tts-sidecar speak --text "Hola mundo" --output audio.wav
```

### Cómo se resuelven las voces

Hay dos niveles, con prioridad para el usuario:

| Nivel | Dónde vive | Permisos |
|---|---|---|
| Usuario | Directorio de datos del usuario, por sistema operativo | Escritura (se crean con `voice clone`) |
| Fábrica | Empaquetadas en el ejecutable, incluida `default` | Solo lectura |

Si registras una voz con el mismo nombre que una de fábrica, la tuya la sobrescribe.

### Comandos disponibles

| Comando | Función |
|---|---|
| `speak --text "..."` | Sintetiza y reproduce (`--output` guarda a WAV) |
| `voice clone` | Clona una voz |
| `voice remove --name X` | Elimina una voz |
| `voice list` | Lista las voces (admite `--json`) |
| `devices` | Muestra los dispositivos de audio (admite `--json`) |
| `doctor` | Diagnóstico (admite `--json`) |
| `setup` | Chequeos y descarga del modelo si falta |
| `version` | Muestra la versión (admite `--json`) |

Para quitar todo, `tts-sidecar setup --uninstall` desinstala en un comando en los tres sistemas. Añade `--yes` para omitir la confirmación.

## Úsalo desde tu lenguaje

Como es una CLI, basta con lanzar un subproceso.

```
# Python
subprocess.run(["./tts-sidecar", "speak", "--text", "Hola mundo"])

# Node.js
child_process.spawn("./tts-sidecar", ["speak", "--text", "Hola mundo"])
```

El README incluye también ejemplos para Bash, Rust, Go y Java. El proyecto cuenta además con un modo daemon (servidor persistente), documentado en `docs/DAEMON-MODE.md` del repositorio.

## Uso ético y responsable

TTS Sidecar clona voces arbitrarias y el audio generado no lleva marca de agua (la de PerthNet está desactivada). Por eso no se distingue técnicamente de una grabación real. El proyecto pide lo siguiente:

- **Consentimiento**: clona solo voces con permiso explícito de su titular.
- **Honestidad de identidad**: no la uses para suplantar a otra persona, cometer fraude, difamar ni producir contenido engañoso.
- **Divulgación**: indica que el audio es sintetizado al publicarlo o compartirlo.
- **Reporte**: si detectas un uso indebido, abre un Issue en el repositorio.

## Licencia

El código de `tts-sidecar` se distribuye bajo **GPL-3.0-or-later**, con uso comercial permitido. Los trabajos derivados que se distribuyan deben liberarse bajo la misma licencia. El modelo Chatterbox Multilingual V3 usa licencia **MIT**. Las dependencias empaquetadas conservan sus propias licencias, detalladas en `THIRD-PARTY-LICENSES.md`.

## Mi experiencia

🎯 Arquitecto de Soluciones | 🚀 Ingeniero en IA | 🤖 ConsultorIA | 👨🏻‍💻 Backend Tech Lead | 🕵🏻‍♂️ Especialista en Ingeniería/Arquitectura | 🏅 Speaker del 1er Congreso IA LATAM | ✨AI Voice-InterConnector

## Más información

- Repositorio: [TTS Sidecar en GitHub](https://github.com/CristianRojas-SoftwareEngineer/TTS-Sidecar)
- Documentación técnica en la carpeta `docs/` del repositorio (diseño, arquitectura, compilación, distribución y paridad entre sistemas).
