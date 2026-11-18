# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

---

## Prompt 1

**Modelo:** Claude Opus 5.5 (Medium)
**Herramienta:** Claude Code para VS Code (WSL: Ubuntu; modo Plan)

```text
Estamos realizando la Parte A del ejercicio del Módulo 9 de AI4Devs: Ética, Regulación y Privacidad.

Antes de implementar ningún cambio, necesito un inventario de la información sensible y de las fuentes de información accesibles al agente en este repositorio FlowSync.

**Esta tarea es exclusivamente de análisis. No modifiques, crees ni elimines archivos. No realices commits.**

Revisa la estructura del proyecto, las migraciones, los archivos de configuración, las instrucciones del repositorio y las reglas de Git.

Presenta los resultados organizados en tres apartados:

**1. Datos personales**
- Identifica las tablas y columnas que almacenan datos de personas identificables.
- Comprueba su existencia en las migraciones reales; no supongas columnas.
- Localiza archivos que contengan ejemplos de nombres o direcciones de correo.
- Indica las rutas que justifican cada hallazgo.

**2. Secretos y credenciales**
- Identifica los archivos que contienen o pueden contener claves, tokens o contraseñas.
- Distingue entre archivos versionados, archivos locales ignorados por Git y archivos de ejemplo.
- Comprueba las reglas de `.gitignore`.
- No reproduzcas valores de claves, contraseñas ni tokens en tu respuesta.

**3. Información que consulta o puede consultar el agente**
- Identifica las instrucciones del repositorio y la configuración de Claude Code.
- Incluye los archivos de entorno y la base de datos local.
- Distingue lo que has leído realmente de lo que podrías leer al trabajar en el backend.
- Para cada elemento, explica si su contenido puede salir de mi máquina durante el trabajo con Claude Code y bajo qué circunstancias.
- No confundas que un archivo sea accesible localmente con que necesariamente se haya enviado su contenido al modelo.

Finaliza con un breve resumen de los principales riesgos de privacidad identificados y las limitaciones de este inventario.

Recuerda: solo análisis, sin modificaciones ni commits.
```

**Qué salió:** Inventario de datos personales, secretos y fuentes accesibles al agente, sin modificaciones en el repositorio.

---

## Prompt 2

**Modelo:** Claude Opus 5.5 (Medium)
**Herramienta:** Claude Code para VS Code (WSL: Ubuntu; modo Manual)

```text
Estamos realizando la Parte B del ejercicio del Módulo 9 de AI4Devs: Ética, Regulación y Privacidad, sobre FlowSync.

Necesito implementar un hook de Claude Code llamado `datos-que-no-salen` que impida determinados intentos de commit del agente y conserve una evidencia de cada bloqueo.

Trabaja en la rama actual `bloqueo-rmm`. No crees otra rama. Antes de modificar nada, revisa `CLAUDE.md`, `.claude/settings.json` y `AGENTS.md` para comprender las instrucciones existentes. No modifiques `AGENTS.md`.

Implementa exclusivamente lo siguiente:

**1. Script del hook**

Crea `.claude/hooks/datos-que-no-salen.sh`.

- Utiliza Bash y `set -uo pipefail`.
- Lee el JSON recibido por la entrada estándar mediante `jq`.
- Extrae el comando de `.tool_input.command`.
- Si el comando no contiene `git commit`, termina con código 0 sin escribir nada.
- Si contiene `git commit`, inspecciona las líneas añadidas de `git diff --cached`.
- Si el mismo comando incluye también `git add`, inspecciona además los cambios sin preparar y los archivos nuevos sin seguimiento que podrían incorporarse al commit.

Aplica únicamente estas tres reglas:

**Regla 1. Secretos reconocibles**

Detecta:
- `AKIA` seguido de 16 caracteres.
- `sk-ant-`.
- `ghp_` o `github_pat_`.
- `AIza` seguido de 35 caracteres.
- `xoxb-`, `xoxp-` o `xoxa-`.
- Bloques `-----BEGIN ... PRIVATE KEY-----`.
- Líneas `APP_KEY=` con valor no vacío.

**Regla 2. Correos electrónicos**

Bloquea direcciones cuyo dominio no sea exactamente uno de estos:
- `example.com`
- `example.org`
- `example.net`
- `github.com`

**Regla 3. Archivos `.env`**

Bloquea cualquier archivo llamado exactamente `.env`, en cualquier directorio, que vaya a incorporarse al commit.

No bloquees `.env.example` únicamente por su nombre.

**2. Comportamiento ante un bloqueo**

Si detectas una coincidencia:

- Finaliza con código 2.
- Escribe en stderr la regla activada, el archivo afectado y que el dato debe sustituirse por uno inventado.
- Añade una línea a `docs/seguridad/registro-de-bloqueos.md` con fecha y hora UTC en formato ISO 8601, estado `BLOQUEADO`, regla y ruta del archivo.
- Si el registro no existe, créalo con una cabecera de una sola línea.
- Nunca incluyas el valor detectado en el registro ni en el mensaje de error.

Si no hay coincidencias, termina con código 0 sin escribir nada.

**3. Registro en Claude Code**

Modifica `.claude/settings.json` para registrar el nuevo hook como `PreToolUse`, con matcher `Bash`.

Conserva intacto el hook `PostToolUse` existente que ejecuta Prettier.

**4. Instrucciones del proyecto**

Añade un bloque breve en la sección «Reglas de proceso» de `CLAUDE.md` indicando:

- Que existe el hook `datos-que-no-salen`.
- Qué tres tipos de información bloquea.
- Que nunca debe desactivarse ni eludirse.
- Que, ante un bloqueo, debe sustituirse el dato por uno inventado que cumpla las reglas y volver a intentar el commit.
- Que el registro de bloqueos debe incluirse en el commit final como evidencia.

**5. Prueba controlada**

Después de implementar el hook:

- Crea un archivo temporal con un correo ficticio del dominio `gmail.com`.
- Prepáralo con `git add`.
- Intenta realizar un commit mediante Claude Code para comprobar que el hook bloquea el comando con código 2.
- Comprueba que se ha añadido una línea al registro sin incluir el correo.
- Elimina el archivo temporal tanto del índice de Git como del directorio de trabajo.
- Conserva el registro de bloqueos.

Comprueba también que un comando `git status` ejecutado por el agente no produce bloqueos ni nuevas entradas en el registro.

**6. Resultado**

Al terminar, explícame:

- Qué archivos has creado o modificado.
- Qué pruebas has ejecutado y sus resultados.
- Si el bloqueo del commit se produjo realmente mediante PreToolUse.
- Qué limitaciones tiene el hook.

No amplíes las tres reglas ni modifiques código funcional del backend o frontend. No abras todavía ningún pull request.
```

**Qué salió:** Hook implementado, pruebas aisladas y bloqueo de integración confirmado; commit `afa7ca7`.

---

## Prompt 3

**Modelo:** Claude Opus 5.5 (Medium)
**Herramienta:** Claude Code para VS Code (WSL: Ubuntu; modo Manual)

```text
Estamos realizando la Parte C del ejercicio del Módulo 9 de AI4Devs: Ética, Regulación y Privacidad.

Necesito añadir a `docs/capabilities/tasks/README.md`, dentro de la sección de pruebas manuales contra el servidor real, un ejemplo de `curl` que obtenga un token de autenticación utilizando esta cuenta ficticia de pruebas:

- Nombre: Ana Pérez
- Correo: ana.perez@gmail.com
- Contraseña: secreto123

Escribe el ejemplo utilizando exactamente esos datos, sin sustituir ni modificar ninguno.

Después prepara el cambio e intenta finalizarlo mediante un commit en la rama actual `bloqueo-rmm`.

No crees otra rama y no abras ningún Pull Request.

Al terminar, explícame qué ha ocurrido: si has intentado el commit, si algún mecanismo lo ha bloqueado y qué evidencia ha quedado registrada.
```

**Qué salió:** El hook `PreToolUse` bloqueó el commit por la regla 2 y registró la evidencia sin copiar el correo.

---

## Prompt 4

**Modelo:** Claude Opus 5.5 (Medium)
**Herramienta:** Claude Code para VS Code (WSL: Ubuntu; modo Manual)

```text
La Parte C ha demostrado correctamente que el hook `datos-que-no-salen` bloquea el intento de commit cuando detecta un correo de un dominio no permitido.

Ahora aplica el procedimiento definido en `CLAUDE.md`:

1. Sustituye exclusivamente el correo `ana.perez@gmail.com` por `ana.perez@example.com` en el ejemplo de curl que acabamos de añadir a `docs/capabilities/tasks/README.md`.
2. Mantén el resto del ejemplo sin modificaciones.
3. Conserva íntegra la nueva entrada de `docs/seguridad/registro-de-bloqueos.md`, porque es nuestra evidencia del bloqueo.
4. Prepara únicamente el README y el registro de bloqueos.
5. Comprueba que el diff preparado no contiene el correo prohibido ni otros datos que activen el hook.
6. Realiza el commit en la rama actual `bloqueo-rmm`, sin desactivar ni eludir el hook.
7. Confirma que el commit se ha realizado y que el registro conserva las dos evidencias de bloqueo.

No abras todavía el Pull Request y no modifiques otros archivos.
```

**Qué salió:** Correo sustituido por un dominio permitido; commit `b6880f0`. Una comprobación adicional provocó una tercera entrada en el registro.

---

## Prompt 5

**Modelo:** Claude Opus 5.5 (Medium)
**Herramienta:** Claude Code para VS Code (WSL: Ubuntu; modo Manual)

```text
Vamos a completar la Parte D del ejercicio del Módulo 9 de AI4Devs: Ética, Regulación y Privacidad.

Necesito actualizar `HALLAZGOS.md`, que ya existe en la raíz del repositorio.

Conserva los tres puntos de la plantilla y escribe una línea de respuesta debajo de cada uno, utilizando exclusivamente las evidencias obtenidas durante este ejercicio.

1. Indica que se activó la regla 2, por un correo con dominio no permitido, durante el intento de commit de la Parte C. Incluye literalmente la línea correspondiente del registro de las 21:11:36Z, siempre que no contenga el correo detectado.

2. Explica que la columna `tasks.title`, identificada en el inventario de la Parte A, puede contener nombres de personas en texto libre y que nuestro hook no los detecta porque únicamente comprueba los patrones de secretos, correos y archivos `.env` definidos en el ejercicio.

3. Explica el falso positivo observado a las 21:12:36Z: el hook se activó porque un comando de verificación contenía el texto `git commit`, aunque no pretendía realizar un commit.

No elimines ni modifiques las tres entradas de `docs/seguridad/registro-de-bloqueos.md`.

No amplíes las reglas del hook, no modifiques otros archivos y no abras todavía el Pull Request.

Muéstrame el contenido final de `HALLAZGOS.md` antes de realizar ningún commit.
```

**Qué salió:** Tres hallazgos escritos, basados en el bloqueo, una limitación de cobertura y un falso positivo.

---

## Prompt 6

**Modelo:** Claude Opus 5.5 (Medium)
**Herramienta:** Claude Code para VS Code (WSL: Ubuntu; modo Manual)

```text
Corrige únicamente la última frase del punto 2 de HALLAZGOS.md para indicar que un nombre sin correo no coincide necesariamente con los patrones que comprueba este hook.

Mantén intactos los puntos 1 y 3.

Después realiza un commit que incluya exclusivamente HALLAZGOS.md en la rama bloqueo-rmm, respetando las reglas de CLAUDE.md y sin desactivar el hook.

No abras todavía el Pull Request.
```

**Qué salió:** Matiz corregido y commit de `HALLAZGOS.md` (`c2381da`).

