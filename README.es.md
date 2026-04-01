# Claude Code: Buenas Prácticas

[English](README.md) | [Русский](README.ru.md) | [Українська](README.uk.md) | **Español**

> **¿Para quién es esto?** Desarrolladores que ya usan Claude Code y quieren pasar de "funciona" a "esto es genuinamente 10 veces mejor." Tanto si llevas una semana usándolo como seis meses, aquí hay algo para ti.

La mayoría de la gente instala Claude Code, escribe algunos prompts y obtiene resultados decentes. Pero hay una diferencia enorme entre usarlo de manera casual y configurarlo para que Claude realmente *entienda* tus proyectos, recuerde tus decisiones y trabaje como tú quieres. Esta guía cierra esa brecha.

Empezaremos con los bloques fundamentales — plugins, servidores MCP y comandos — y luego veremos las prácticas que se acumulan con el tiempo: cómo estructurar tus instrucciones, cuándo usar cada herramienta y cómo hacer que las sesiones dejen de sentirse como empezar desde cero cada vez.

---

## Tabla de Contenidos

| # | Sección | Qué Aprenderás |
|---|---------|----------------|
| 1 | [El Ecosistema de Extensiones](#1-el-ecosistema-de-extensiones) | Plugins, servidores MCP y slash commands — las tres formas de extender Claude Code |
| 2 | [Archivos CLAUDE.md](#2-claudemd--tus-instrucciones-permanentes) | Global vs proyecto, reglas por ruta y `/init` — qué va dónde y por qué |
| 3 | [Skills](#3-skills--carga-conocimiento-cuando-lo-necesitas) | Convierte cualquier conocimiento de proyecto en contexto bajo demanda sin sobrecargar cada sesión |
| 4 | [Superpowers](#4-superpowers--el-plugin-que-lo-cambia-todo) | Flujos de trabajo estructurados que convierten a Claude de asistente en compañero de ingeniería |
| 5 | [Serena](#5-serena--navegación-de-código-que-entiende-tu-código) | Navega y edita código por *significado*, no solo por búsqueda de texto |
| 6 | [Ripgrep](#6-ripgrep--cuando-la-búsqueda-de-texto-es-la-herramienta-correcta) | Búsqueda de texto rápida para todo lo que Serena no puede ver |
| 7 | [Selección de Herramientas](#7-enseñar-a-claude-cuándo-usar-qué) | La tabla más impactante que puedes agregar a tu configuración |
| 8 | [Graphiti](#8-graphiti--memoria-que-persiste-entre-sesiones) | Un grafo de conocimiento para que Claude recuerde qué decidiste y por qué |
| 9 | [Sistemas de Memoria](#9-memoria--la-información-correcta-en-el-lugar-correcto) | Auto memory, MEMORY.md, Graphiti, CLAUDE.md — qué va dónde |
| 10 | [Context7](#10-context7--documentación-actualizada-de-verdad) | Documentación de librerías en tiempo real, porque tus datos de entrenamiento ya están desactualizados |
| 11 | [Hooks y Guardarraíles](#11-hooks-y-guardarraíles--automatiza-y-protege) | Automatización de sesiones, hooks de seguridad y git hooks trabajando juntos |
| 12 | [Permisos y Seguridad](#12-permisos-y-seguridad) | Deja que Claude vaya rápido sin que pueda hacer nada peligroso |
| 13 | [Subagentes y Equipos de Agentes](#13-subagentes-y-equipos-de-agentes) | Agentes personalizados, niveles de modelos y coordinación multi-agente |
| 14 | [Gestión de la Ventana de Contexto](#14-gestión-de-la-ventana-de-contexto) | `/compact`, `/clear`, `/btw` y mantener las sesiones enfocadas |
| 15 | [Git Worktrees](#15-git-worktrees--sesiones-en-paralelo) | Ejecuta múltiples sesiones de Claude en el mismo repo sin conflictos |
| 16 | [Git y Code Review](#16-git-y-code-review) | Commits convencionales, CodeRabbit y revisiones locales con subagentes |
| 17 | [Gestión de Sesiones](#17-gestión-de-sesiones) | Retoma, renombra y navega sesiones como un profesional |
| 18 | [Línea de Estado](#18-línea-de-estado--ve-qué-está-pasando) | Uso de contexto, coste y estadísticas de sesión en tiempo real de un vistazo |
| 19 | [Seguimiento de Tareas](#19-seguimiento-de-tareas--no-pierdas-el-hilo) | Por qué las tareas en sesión importan más de lo que crees |
| 20 | [Patrones de Fallo Comunes](#20-patrones-de-fallo-comunes--qué-evitar) | Los errores que todos cometen y cómo romper el ciclo |
| 21 | [Config Sync](#21-config-sync--la-misma-configuración-en-todas-partes) | Mantén dos máquinas perfectamente sincronizadas |

---

## 1. El Ecosistema de Extensiones

Claude Code de serie está bien. Claude Code con las extensiones adecuadas es una experiencia completamente diferente.

Hay tres formas de extenderlo, y funcionan juntas:

### Plugins — lo más importante

Los plugins agrupan todo: slash commands, agentes especializados, hooks y a veces herramientas MCP. Piensa en ellos como "paquetes de funcionalidades" para Claude Code.

```bash
# Explorar el marketplace de plugins
/plugins

# O instalar directamente
claude plugin install superpowers@claude-plugins-official
```

Estos son los plugins que vale la pena instalar de inmediato — profundizaremos en los más importantes más adelante:

| Plugin | Por Qué Lo Quieres |
|--------|-------------------|
| **superpowers** | Planificación, brainstorming, TDD, debugging, worktrees, code review — los flujos de trabajo estructurados que hacen a Claude disciplinado |
| **claude-md-management** | Escritura y auditoría de CLAUDE.md asistidas por IA |
| **skill-creator** | Convierte cualquier conocimiento de proyecto en un skill reutilizable |
| **serena** | Navegación de código semántica — encuentra símbolos, referencias, edita por significado |
| **context7** | Documentación de librerías en tiempo real para que Claude no alucine APIs desactualizadas |
| **code-review** | Flujos de trabajo de revisión de PRs con revisores subagentes en paralelo |
| **coderabbit** | Integración con la revisión de IA de [CodeRabbit](https://www.coderabbit.ai/) |
| **feature-dev** | Desarrollo de funcionalidades guiado con exploración del codebase |
| **playwright** | Automatización y testing en navegador |
| **plugin-dev** | Herramientas para construir tus propios plugins |

### Servidores MCP — herramientas externas

Los servidores MCP (Model Context Protocol) conectan a Claude con sistemas externos — tu IDE, gestor de issues, grafo de conocimiento, herramientas de monitorización. Proporcionan herramientas directas que Claude llama directamente.

```bash
claude mcp add <name> -- <command>
claude mcp list
```

Algunos de los más usados:

| Servidor | Qué Hace |
|----------|---------|
| **Graphiti** | Grafo de conocimiento para memoria persistente |
| **JetBrains** | Compilar, testear y refactorizar desde tu IDE |
| **Atlassian** | Jira + Confluence (normalmente preinstalado vía OAuth) |
| **Sentry** | Seguimiento de errores e investigación de issues |
| **Docker MCP** | Un catálogo de servidores MCP que puedes instalar bajo demanda |

### Slash Commands — la interfaz de usuario

Los slash commands son la forma de invocar skills desde plugins. Escribe `/` y verás lo que hay disponible:

```text
/commit              Crea un git commit
/code-review         Revisa un pull request
/revise-claude-md    Actualiza CLAUDE.md con los aprendizajes de la sesión
/skill-creator       Crea un nuevo skill a partir de conocimiento
/simplify            Revisa y simplifica el código modificado
```

Ese es el tour rápido. Ahora entremos en lo que realmente marca la diferencia.

---

## 2. CLAUDE.md — Tus Instrucciones Permanentes

Si solo haces una cosa de esta guía, que sea esta: **escribe un buen CLAUDE.md.**

Cada vez que inicias una sesión, Claude lee tus archivos CLAUDE.md. Son tus instrucciones persistentes — lo que de otra manera repetirías en cada conversación. Consigue la división correcta entre global y proyecto, y Claude empieza cada sesión sabiendo ya cómo trabajas.

### Dos archivos, dos funciones

| | Global (`~/.claude/CLAUDE.md`) | Proyecto (`<repo>/CLAUDE.md`) |
|-|-------------------------------|------------------------------|
| **Se carga** | En cada sesión, en cada proyecto | Solo en ese proyecto |
| **Piénsalo como** | Tu perfil de desarrollador | El manual del proyecto |
| **Contiene** | Cómo trabajas *tú* | Cómo funciona *este codebase* |

### Tu archivo global: el perfil del desarrollador

Esto le dice a Claude quién eres en todos los proyectos. Escríbelo una vez, refínalo con el tiempo:

- **Qué herramientas están disponibles** y cuándo activar cada una
- **Cómo te gustan los commits formateados** y tu estrategia de ramas
- **Tu estándar de calidad de código** — DRY, SOLID, tipado fuerte, lo que te importe
- **Qué tan exhaustivamente verificar el trabajo** antes de decir "listo"
- **Qué modelo de IA para qué tarea** — Haiku para consultas, Opus para arquitectura
- **Tus proyectos activos** — nombres, rutas, claves Jira para referencia rápida

Aquí está el esqueleto de un buen CLAUDE.md global:

```markdown
# Global Claude Code Instructions

## Tools & MCP Servers
| Tool | Purpose | When to Activate |
|------|---------|-----------------|
| Serena | Code navigation | activate_project("<name>") at session start |
| Graphiti | Persistent memory | search_memory_facts at session start |
| Context7 | Library docs | Before writing integration code |

## Git
Commits: <type>(<scope>): <description>
Never commit to main. Never force push. PRs require confirmation.

## Code Quality
Read before modifying. Comments explain *why*. No `any` types.

## Subagents
- haiku — exploration, quick lookups
- sonnet — implementation, tests, code review
- opus — architecture, security-critical decisions
```

> **Consejo pro:** Apunta a unas 100-200 líneas. Boris Cherny (creador de Claude Code) ha dicho que "unas 100 líneas funciona mejor que 800". Cada línea debería pasar la prueba: *¿el hecho de eliminarla haría que Claude cometiese errores?* Si Claude hace lo correcto sin una regla, elimina la regla.

### Tu archivo de proyecto: el manual del codebase

Aquí vive el conocimiento específico del proyecto. Piénsalo como el documento de onboarding que le darías a un nuevo miembro del equipo:

- **Arquitectura** — modelos de datos, archivos clave, cómo se conectan las cosas
- **Comandos de build/test/lint** — los comandos exactos, sin suposiciones
- **Estándares de código específicos del proyecto** — "usa `Logger`, no `print()`"
- **Trampas comunes** — las cosas que te dan problemas en *este* codebase
- **Punteros de integración** — clave Jira, `group_id` de Graphiti, nombre del proyecto en Serena

> **Qué no pertenece aquí:** Consejos genéricos de programación (eso es global), instrucciones de activación de herramientas (también global), cualquier cosa temporal (eso son tareas o memoria).

### ¿Empezando desde cero? Usa `/init`

Si estás configurando CLAUDE.md para un proyecto existente, no empieces desde un archivo en blanco. El comando `/init` de Claude Code explora tu codebase — archivos de paquetes, configuración, estructura de directorios, documentación existente — y genera un CLAUDE.md inicial automáticamente:

```bash
# Flujo multi-fase interactivo que explora antes de escribir
CLAUDE_CODE_NEW_INIT=1 claude /init
```

Es un punto de partida sólido. Luego refínalo con el tiempo usando `/revise-claude-md` (ver más abajo).

### Reglas por ruta: carga el contexto correcto para los archivos correctos

> *Todavía no lo he usado yo mismo, pero es demasiado útil para no recomendarlo.*

En lugar de cargar todas tus reglas todo el tiempo, puedes aplicar reglas a patrones de archivos específicos usando `.claude/rules/`:

```yaml
# .claude/rules/api-conventions.md
---
paths:
  - "src/api/**/*.ts"
---

## API Design Rules
- All endpoints return { data, error, meta }
- Use Zod for request validation
- Rate limit: 100 req/min per user
```

Este archivo solo se carga cuando Claude trabaja con archivos que coinciden con `src/api/**/*.ts`. Tus convenciones de tests solo se cargan cuando editas tests. Tus patrones de frontend solo se cargan cuando tocas componentes. La misma idea que los skills (Sección 3), pero activado por ruta en lugar de por tarea.

### No lo escribas solo

El plugin **claude-md-management** tiene dos skills que hacen el trabajo pesado:

```bash
claude plugin install claude-md-management@claude-plugins-official
```

| Comando | Qué Hace |
|---------|---------|
| `/claude-md-improver` | Audita tu CLAUDE.md existente en busca de lagunas, inconsistencias y buenas prácticas que se están perdiendo |
| `/revise-claude-md` | Después de una sesión de trabajo, captura lo que aprendiste — nuevos patrones, trampas, correcciones |

**El hábito que vale la pena:** Ejecuta `/revise-claude-md` al final de cualquier sesión en la que hayas descubierto algo sobre el proyecto. Con el tiempo, tu CLAUDE.md evoluciona de un esbozo a un manual completo y probado en batalla — sin que tengas que escribirlo desde cero.

### Herencia de CLAUDE.md (monorepos y directorios anidados)

Claude Code sube el árbol de directorios y carga cada CLAUDE.md que encuentra:
- `~/.claude/CLAUDE.md` — tu perfil global
- `./CLAUDE.md` o `./.claude/CLAUDE.md` — raíz del proyecto (súbelo a git para el equipo)
- Archivos CLAUDE.md en directorios hijos — se cargan de forma perezosa cuando Claude trabaja en esos directorios

En monorepos, el CLAUDE.md raíz cubre las convenciones compartidas, y cada directorio de paquete puede tener sus propias reglas. Usa `claudeMdExcludes` en settings si los archivos de otros equipos generan ruido.

---

## 3. Skills — Carga Conocimiento Cuando lo Necesitas

Aquí hay un problema que encontrarás rápidamente: tu CLAUDE.md se hace largo. Sigues añadiendo reglas — convenciones de API, patrones de testing, estándares de base de datos, checklists de despliegue — y eventualmente, cada sesión carga 2.000 tokens de contexto tanto si los necesita como si no.

Los skills resuelven esto. Son como secciones de CLAUDE.md que se cargan **bajo demanda** en lugar de siempre.

### La idea en 30 segundos

Un skill es un archivo markdown con una descripción. Cuando Claude empieza una tarea, comprueba: "¿Algún skill coincide con lo que estoy a punto de hacer?" Si es así, carga el contenido de ese skill. Si no, se mantiene al margen.

```text
Tú: Arregla el test inestable en UserService

Claude: [ve que es una tarea de testing]
        [carga el skill test-driven-development]
        [carga el skill de convenciones de testing del proyecto]
        [trabaja con contexto completo de testing — nada más cargado]
```

### El antes y el después

**Antes** — todo metido en CLAUDE.md:

```markdown
## API Design Rules
- All endpoints return { data, error, meta }
- Use Zod for request validation
- Rate limit: 100 req/min per user
- Error codes follow RFC 7807
... (50 líneas más)

## Database Conventions
- All tables have created_at, updated_at
- Use soft deletes
- Indexes on all foreign keys
... (40 líneas más)
```

Cada sesión carga todo esto. ¿Arreglando un bug de CSS? Sigues cargando reglas de API y base de datos en contexto.

**Después** — CLAUDE.md ligero, conocimiento bajo demanda:

```markdown
## Available Skills
- `/api-design` — API conventions, response format, error codes
- `/db-conventions` — Database patterns, migrations, indexes
```

Ahora el contexto de API solo se carga cuando realmente estás trabajando en endpoints. Mismo conocimiento, menos desperdicio, mejor enfoque.

### Cómo construir un skill

No necesitas escribir archivos de skill a mano. El plugin **skill-creator** lo hace:

```bash
claude plugin install skill-creator@claude-plugins-official
```

Luego simplemente cuéntale lo que sabes:

```text
Tú: /skill-creator

Claude: ¿Qué conocimiento quieres convertir en un skill?

Tú: Nuestra API siempre devuelve respuestas en este formato: { data, error, meta }.
    Usamos Zod para toda la validación de requests. El rate limit es 100 req/min.
    Los errores siguen RFC 7807. Todos los endpoints están versionados bajo /api/v1/...

Claude: [Crea un archivo de skill bien estructurado con frontmatter apropiado,
         condiciones de activación e instrucciones organizadas]
```

Tú aportas el conocimiento, el plugin se encarga del formato y las descripciones de activación.

### Qué hace funcionar un archivo de skill

```markdown
---
name: api-design
description: Use when creating or modifying API endpoints — covers response
  format, validation, error handling, and versioning conventions
---

## Response Format
All endpoints return:
{ "data": ..., "error": null | { ... }, "meta": { ... } }

## Validation
Use Zod schemas for all request bodies...
```

El campo `description` hace la mayor parte del trabajo. Le dice a Claude *cuándo* cargar el skill, así que escríbelo como una condición de activación: "Usar cuando [hago X] — cubre [Y y Z]."

> **Regla general:** Si le has explicado algo a Claude más de dos veces en sesiones distintas, debería ser un skill.

---

## 4. Superpowers — El Plugin Que Lo Cambia Todo

Si instalas un solo plugin de esta guía, que sea este.

La cosa con Claude es que *es capaz* de excelentes prácticas de ingeniería — planificar antes de programar, escribir tests primero, depurar metódicamente. Pero abandonado a sus propios dispositivos, tiende a ir directo a escribir código. Todos lo hacemos. La diferencia es que Superpowers hace que estas prácticas sean **automáticas y obligatorias**, no aspiracionales.

```bash
claude plugin install superpowers@claude-plugins-official
```

### Qué cambia después de instalarlo

Superpowers añade un conjunto de **skills de proceso** que se activan en función de lo que estás haciendo:

| Cuando estás... | Superpowers activa... | Lo que significa que Claude... |
|----------------|----------------------|-------------------------------|
| Empezando algo nuevo | **Brainstorming** | Presenta 2-3 enfoques con sus trade-offs antes de escribir código |
| Listo para construir | **Writing Plans** | Crea un plan de implementación paso a paso para tu aprobación |
| Construyendo | **Executing Plans** | Sigue el plan paso a paso, con seguimiento de progreso |
| Escribiendo cualquier función o fix | **Test-Driven Development** | Escribe un test que falle *primero*, luego la implementación |
| Encontrando un bug | **Systematic Debugging** | Formula hipótesis, las prueba metódicamente, acota la causa |
| Terminando un bloque de trabajo | **Code Review** | Revisa contra el plan y los estándares del proyecto |
| A punto de decir "listo" | **Verification** | Ejecuta comprobaciones finales antes de declarar la tarea completa |
| Con tareas independientes | **Parallel Agents** | Despacha múltiples agentes simultáneamente |
| Empezando trabajo de feature | **Git Worktrees** | Aísla el trabajo en una rama de worktree dedicada |
| Completando una rama | **Finishing a Branch** | Finalización limpia de rama y preparación del PR |

### Por qué esto es enormemente útil

**Sin Superpowers**, una sesión típica se parece a esto:

```text
Tú: Implementa autenticación de usuario con OAuth

Claude: ¡Claro! Voy a crear el middleware de auth...
        [escribe 200 líneas de código]
        [se da cuenta de que necesita un enfoque diferente]
        [reescribe la mitad]
        ¡Listo! He implementado la autenticación OAuth.
        [sin tests, sin plan, sin verificación]
```

**Con Superpowers:**

```text
Tú: Implementa autenticación de usuario con OAuth

Claude: [Brainstorming se activa]
        Veo tres enfoques aquí:
        1. Passport.js — maduro, muchas estrategias, pero pesado
        2. Auth.js (NextAuth) — construido para Next.js, más simple
        3. Custom con jose — ligero, control total
        Aquí están los trade-offs...

Tú: Opción 2, vamos con Auth.js

Claude: [Planning se activa]
        Aquí está el plan de implementación:
        Paso 1: Instalar dependencias y configurar providers
        Paso 2: Crear rutas API de auth
        Paso 3: Añadir middleware de sesión
        Paso 4: Proteger rutas con guards de auth
        ...

Tú: Bien, adelante

Claude: [Execution se activa]
        Empezando el Paso 1...
        [TDD se activa] Escribiendo test para la configuración del OAuth provider...
        [El test falla — bien]
        [Escribe la implementación para pasar el test]
        [El test pasa]
        Paso 1 completo. Pasando al Paso 2...
        ...
        [Todos los pasos completados]
        [Verification se activa]
        Ejecutando la suite completa de tests... Todos pasando.
        Comprobando contra el plan... Todos los pasos cubiertos.
        Listo.
```

La misma petición, resultado completamente diferente. El plan detecta los giros equivocados temprano. Los tests detectan las regresiones. La verificación previene sorpresas del tipo "funciona en mi máquina".

### El flujo de planificación previene el mayor desperdicio de tiempo

El mayor desperdicio de tiempo en el desarrollo asistido por IA no son los modelos lentos ni el código malo — es **construir lo que no toca**. Dices "añade autenticación", Claude lo interpreta de una manera, tú querías algo diferente, y has quemado 20 minutos en código que vas a tirar.

El flujo plan-luego-ejecuta de Superpowers elimina esto. Claude te muestra el plan *antes* de escribir código. Lo revisas, lo ajustas, y solo entonces dices "adelante". Los 2 minutos que tardas en revisar un plan te ahorran los 20 minutos que perderías deshaciendo el enfoque equivocado.

### Activadores de pensamiento profundo para decisiones críticas

Cuando necesitas que Claude piense especialmente duro — decisiones de arquitectura, refactorizaciones complejas, depuración complicada — puedes llevar el razonamiento al máximo:

- **`ultrathink`** — incluye esta palabra en cualquier prompt para el máximo esfuerzo de razonamiento en esa petición específica
- **`/effort high`** — establece un nivel de esfuerzo persistente para toda la sesión
- **EXPLORE -> PLAN -> CODE -> COMMIT** — haz que Claude lea archivos primero sin escribir, luego `ultrathink` sobre el plan, critícalo por casos edge, *y entonces* ejecuta

Son especialmente potentes combinados con el flujo de planificación de Superpowers. Deja que Claude haga brainstorming con esfuerzo normal, luego `ultrathink` sobre el plan final antes de la ejecución.

---

## 5. Serena — Navegación de Código Que Entiende Tu Código

La mayor parte del tiempo, cuando Claude necesita entender tu código, hace una de dos cosas: lee el archivo entero, o ejecuta una búsqueda de texto. Ambas funcionan, pero ambas son un desperdicio. Leer un archivo de 500 líneas cuando solo necesitas una función quema contexto. Buscar con grep el nombre de una clase encuentra la definición pero no lo que realmente la *usa*.

Serena cambia esto. Es un servidor MCP que entiende tu código como **símbolos** — clases, funciones, métodos, variables — no solo texto. Sabe qué hace una función, dónde está definida, y todo lo que la referencia.

### Cómo se siente la diferencia

| La forma antigua | Con Serena |
|----------------|------------|
| `grep -r "class UserService"` por todo el proyecto | `find_symbol("UserService")` — va directo allí |
| Leer el archivo entero para entender su estructura | `get_symbols_overview("user-service.ts")` — solo el esquema |
| Buscar y reemplazar para renombrar algo | `rename_symbol("oldName", "newName")` — seguro y semántico |
| Buscar manualmente "¿quién llama esto?" | `find_referencing_symbols("UserService")` — respuesta completa |

El ahorro de tokens por sí solo es significativo. Pero la verdadera ganancia es la precisión — Serena encuentra la *definición*, no solo coincidencias de texto.

### Configurarlo para un proyecto

Dos pasos. Primero, crea una configuración de Serena en tu proyecto:

```yaml
# .serena/settings.yml
project_name: "my-project"
language_server:
  language: typescript  # o python, swift, go, etc.
  root_path: "src"
```

Segundo, dile a Claude que lo active. En el CLAUDE.md de tu proyecto:

```markdown
## Serena (Semantic Code Analysis)
At session start, call `activate_project("my-project")`.
```

(Si configuras un hook de inicio de sesión — cubierto en la [Sección 11](#11-hooks-y-guardarraíles--automatiza-y-protege) — esto ocurre automáticamente.)

### El flujo de trabajo con Serena

La forma más eficiente de explorar código con Serena sigue un patrón de **zoom-in**:

```text
list_dir                → ¿Qué hay en este directorio?
get_symbols_overview    → ¿Qué símbolos hay en este archivo? (sin cuerpos — solo nombres y firmas)
find_symbol             → Muéstrame el cuerpo completo de esta función
find_referencing_symbols → ¿Quién usa esto? (antes de cambiarlo)
```

Y para editar:

```text
replace_symbol_body   → Reescribe el cuerpo de una función con precisión
insert_before_symbol  → Añade algo encima de un símbolo
insert_after_symbol   → Añade algo debajo de un símbolo
```

### Dónde Serena no ayuda

Serena solo indexa **símbolos de código**. No puede ver:
- Archivos de configuración (JSON, YAML, TOML)
- Markdown y documentación
- Literales de cadena, mensajes de error, comentarios
- Archivos que no sean código de ningún tipo

Para esos, necesitas búsqueda de texto — lo que nos lleva a...

---

## 6. Ripgrep — Cuando la Búsqueda de Texto Es la Herramienta Correcta

Ripgrep (`rg`) es una herramienta de búsqueda de texto rápida, y Claude Code la usa internamente para su herramienta `Grep`. Por defecto, Claude usa una versión integrada, pero puedes apuntarla a tu instalación del sistema para mejor rendimiento.

### Configuración rápida

```json
// ~/.claude/settings.json
{
  "env": {
    "USE_BUILTIN_RIPGREP": "0"
  }
}
```

```bash
# Instalar si no lo tienes
brew install ripgrep        # macOS
sudo apt install ripgrep    # Ubuntu/Debian
```

### Serena vs Ripgrep — la división del trabajo

Estas dos herramientas se complementan perfectamente, y enseñarle a Claude cuándo usar cada una es una de las cosas de mayor valor que puedes hacer (más sobre esto en la [Sección 7](#7-enseñar-a-claude-cuándo-usar-qué)):

| Usa Ripgrep cuando necesitas | Usa Serena cuando necesitas |
|-----------------------------|----------------------------|
| Literales de cadena, mensajes de error | Definiciones de funciones/clases |
| Valores de configuración en muchos archivos | ¿Quién referencia este símbolo? |
| Archivos no-código (docs, JSON, YAML) | El cuerpo de un método específico |
| Patrones regex en cualquier lugar | Renombrado seguro de símbolos |
| "Encuentra cada archivo que mencione X" | "¿Cómo es la API de este archivo?" |

Ninguna herramienta sustituye a la otra. Juntas, cubren todo.

---

## 7. Enseñar a Claude Cuándo Usar Qué

Esto es posiblemente lo de mayor apalancamiento en toda esta guía, y es simplemente una tabla.

Claude tiene muchas herramientas disponibles — Serena, Ripgrep, Glob, Context7, Graphiti, la herramienta Edit, la herramienta Read. Sin orientación, recurre a sus valores de entrenamiento por defecto: leer archivos completos y buscar con grep. Eso está bien, pero es como usar un martillo para todo cuando también tienes un destornillador y un taladro.

### La tabla

Añade esto a tu CLAUDE.md global. En serio, simplemente pégala:

| Tarea | Mejor Herramienta | Por Qué |
|-------|------------------|---------|
| Encontrar un símbolo/clase/función | Serena `find_symbol` | Semántica, consciente del lenguaje |
| Entender la estructura de un archivo | Serena `get_symbols_overview` | Eficiente en tokens — solo el esquema |
| Comprobar referencias antes de cambiar algo | Serena `find_referencing_symbols` | Completa — detecta cada uso |
| Buscar literales de cadena, valores de config, mensajes de error | `Grep` (ripgrep) | Patrones de texto que no son símbolos de código |
| Buscar archivos no-código (markdown, JSON, YAML) | `Grep` (ripgrep) | Serena solo indexa código |
| Encontrar archivos por nombre o patrón | `Glob` | Más rápido que cualquier búsqueda para descubrir archivos |
| Editar el cuerpo de una función/método | Serena `replace_symbol_body` | Reemplazo preciso a nivel de símbolo |
| Editar configuración o archivos no-código | `Edit` | Edición de texto estándar |
| Consultar la API actual de una librería | Context7 | Documentación en tiempo real — los datos de entrenamiento pueden estar desactualizados |
| Recordar una decisión o patrón pasado | Graphiti `search_memory_facts` | Grafo de conocimiento multi-sesión |

**La regla general:** Serena para símbolos de código. Grep para texto y no-código. Glob para archivos. Context7 para documentación. Graphiti para memoria.

### Por qué esto importa más de lo que crees

Sin esta tabla, una sesión típica de Claude podría:
1. Leer un archivo de 400 líneas entero para encontrar una función (40 tokens/línea = 16.000 tokens desperdiciados)
2. Buscar con grep el nombre de una clase y obtener coincidencias de cadenas en comentarios, tests y docs
3. Intentar recordar si la API de Zustand cambió desde su fecha de corte de entrenamiento

Con la tabla:
1. `get_symbols_overview` -> 200 tokens para todo el esquema del archivo, luego leer el cuerpo de una función
2. `find_symbol` -> va directamente a la definición
3. Context7 -> obtiene la documentación actual

La misma tarea, una fracción de los tokens, mayor precisión. A lo largo de una sesión, esto se acumula.

---

## 8. Graphiti — Memoria Que Persiste Entre Sesiones

Aquí hay un patrón frustrante: pasas 30 minutos explicando tu arquitectura a Claude. Avanzas bien. En la siguiente sesión, lo explicas todo de nuevo. CLAUDE.md ayuda — pero es estático, y tienes que acordarte de actualizarlo.

Graphiti es diferente. Es un **grafo de conocimiento** en el que Claude escribe durante las sesiones y del que lee al inicio. Con el tiempo, construye una red de entidades, relaciones y hechos sobre tu proyecto — con marcas de tiempo para que sepas cuándo se decidió algo y si sigue vigente.

### Qué va en Graphiti (y qué no)

Piensa en Graphiti como el lugar para **decisiones con contexto** — las cosas que son demasiado detalladas o demasiado fluidas para CLAUDE.md:

**Sí:**
- "Elegimos event sourcing para orders porque necesitamos un audit trail. El trade-off son lecturas más complejas vía projections."
- "La reescritura del middleware de auth está motivada por cumplimiento legal, no deuda técnica — las decisiones de alcance deben favorecer el cumplimiento sobre la ergonomía."
- "React Query v5 requiere envolver las mutations en startTransition — descubierto durante la actualización."

**No:**
- Cambios de código rutinarios (para eso está el git log)
- Secretos o credenciales (nunca)
- Volcados verbosos de código (guárdalos en el código, no en la memoria)
- Cualquier cosa que ya esté en CLAUDE.md (no dupliques)

### Cómo funciona en la práctica

Cada proyecto obtiene un `group_id` para delimitar su conocimiento. Añade esto al CLAUDE.md de tu proyecto:

```markdown
## Memory (Graphiti Knowledge Graph)
When using Graphiti tools, always use `group_id="my-project"`.
```

**Inicio de sesión** — Claude carga contexto relevante:

```text
search_memory_facts(
  query="architecture decisions patterns",
  group_ids=["my-project"]
)
```

**Fin de sesión** — Claude guarda lo que aprendió:

```text
add_memory(
  group_id="my-project",
  content="Decided to use event sourcing for order processing
           because we need audit trail and temporal queries."
)
```

¿Lo mejor? Puedes automatizar ambos con hooks (ver [Sección 11](#11-hooks-y-guardarraíles--automatiza-y-protege)), para que simplemente *ocurra* sin que tengas que pensar en ello.

---

## 9. Memoria — La Información Correcta en el Lugar Correcto

Claude Code tiene múltiples sistemas de memoria, y cada uno hace algo diferente. Usar el equivocado es como archivar un recibo en tu diario — está guardado, pero nunca lo encontrarás cuando lo necesites.

### La hoja de referencia rápida

| Qué Estás Almacenando | Dónde Va | Por Qué Ahí |
|----------------------|---------|-------------|
| Instrucciones y reglas persistentes | **CLAUDE.md** | Siempre cargado, siempre seguido — tú lo escribes |
| Datos de referencia rápida (IDs, nombres, flags de estado) | **MEMORY.md** | Se carga automáticamente por proyecto, escaneable |
| Patrones que Claude descubre mientras trabaja | **Auto Memory** | Claude los escribe él mismo mientras trabaja |
| Decisiones con contexto rico (el *por qué*) | **Graphiti** | Buscable, con marcas de tiempo, relacional |
| Progreso de tarea actual | **Tasks** | Sobrevive a la compactación de contexto |
| Conocimiento profundo por tema | **Skills** | Se carga bajo demanda, no siempre |

### Auto Memory — Claude enseñándose a sí mismo

Esto es distinto del MEMORY.md que tú escribes. Claude Code escribe automáticamente notas para sí mismo mientras trabaja — comandos de build que funcionaron, insights de depuración, arquitectura que descubrió, correcciones de estilo de código que le diste. Estos se acumulan en `~/.claude/projects/<project>/memory/`.

La distinción clave:
- **CLAUDE.md** = reglas que *tú* escribes, se cargan en cada sesión
- **Auto Memory** = patrones que *Claude* aprende, también se cargan en cada sesión
- No los dupliques — si Claude ya aprendió algo vía auto memory, no necesitas añadirlo a CLAUDE.md

### MEMORY.md en la práctica

Ubicado en `~/.claude/projects/<slug>/memory/MEMORY.md`, este archivo se carga automáticamente en cada sesión para ese proyecto.

**Buen contenido para MEMORY.md:**
- "ID del proyecto Supabase: `abc123`"
- "Migración a v2 auth: completada el 2026-01-15"
- "Nombres de edge functions: `process-payment`, `send-notification`"

**Mal contenido para MEMORY.md:**
- Patrones de código (lee el codebase en su lugar)
- Resúmenes del historial git (ejecuta `git log`)
- Recetas de fix (el fix está en el código)
- Cualquier cosa que ya esté en CLAUDE.md

**Mantenlo por debajo de 200 líneas.** Para temas más profundos, enlaza a archivos separados:

```markdown
<!-- MEMORY.md -->
- [Supabase setup](supabase.md) — project ID, edge functions, RLS policies
- [Localization](localization.md) — supported languages, string key format
```

---

## 10. Context7 — Documentación Actualizada de Verdad

Los datos de entrenamiento de Claude tienen una fecha de corte. Las librerías publican actualizaciones después de esa fecha. Esto significa que Claude podría escribir código con confianza usando una API que cambió hace dos meses. Context7 soluciona esto obteniendo **documentación en tiempo real** en el momento en que Claude la necesita.

```bash
claude plugin install context7@claude-plugins-official
```

### Cuándo importa

- Escribir código de integración para cualquier librería (React, Express, Prisma, Tailwind — todas)
- Actualizar o migrar entre versiones de librerías
- Depurar comportamiento específico de una librería
- Usar una librería sobre la que Claude parece inseguro

### Enseña a Claude a usarlo

Añade a tu CLAUDE.md global:

```markdown
## Context7
Call Context7 before writing integration code for any library.
Training data may be outdated — always verify current API.
```

### Cuándo saltárselo

Context7 es para documentación de librerías. No lo uses para conceptos generales de programación, refactorizar tu propio código, lógica de negocio o code review.

---

## 11. Hooks y Guardarraíles — Automatiza y Protege

Los hooks sirven para dos propósitos: **automatización** (para que no olvides hacer cosas) y **protección** (para que Claude no haga cosas que no debería). La combinación de hooks de Claude Code y git hooks crea una sólida red de seguridad.

### Inicio de sesión: carga todo automáticamente

Configura un hook que se ejecute al inicio de cada sesión:

```json
// ~/.claude/settings.json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/session-start.sh",
            "statusMessage": "Loading project context"
          }
        ]
      }
    ]
  }
}
```

El script del hook lee el CLAUDE.md de tu proyecto, extrae el `group_id` de Graphiti y el nombre del proyecto en Serena, e inyecta recordatorios:

```bash
#!/bin/bash
INPUT=$(cat)
CWD=$(echo "$INPUT" | python3 -c \
  "import sys,json; print(json.load(sys.stdin).get('cwd',''))" 2>/dev/null)

GROUP_ID=""
SERENA_NAME=""

if [ -f "$CWD/CLAUDE.md" ]; then
  GROUP_ID=$(sed -n 's/.*group_id="\([^"]*\)".*/\1/p' "$CWD/CLAUDE.md" | head -1)
  SERENA_NAME=$(sed -n 's/.*activate_project("\([^"]*\)".*/\1/p' "$CWD/CLAUDE.md" | head -1)
fi

MSGS="Action required: Read ~/.claude/CLAUDE.md for global instructions.\n"
[ -n "$SERENA_NAME" ] && MSGS="${MSGS}Action required: activate_project(\"$SERENA_NAME\")\n"
[ -n "$GROUP_ID" ] && MSGS="${MSGS}Action required: search Graphiti with group_id=$GROUP_ID\n"
echo -e "$MSGS"
```

Ahora cada sesión empieza completamente cargada — sin recordatorios manuales.

### Fin de sesión: guarda lo que aprendiste

Un stop hook se ejecuta cuando Claude está a punto de terminar. Úsalo para capturar conocimiento no guardado:

```json
{
  "stopHooks": [
    {
      "type": "prompt",
      "prompt": "Before stopping, check if any architectural decisions or bug root causes were discovered. If so, save to Graphiti. Check if MEMORY.md needs new identifiers or status updates.",
      "model": "haiku"
    }
  ]
}
```

> **¿Por qué Haiku?** El stop hook es una auditoría rápida — no necesita razonamiento pesado. Usar Haiku lo mantiene rápido y barato.

### Tipos de hooks: más que solo shell scripts

Los hooks vienen en tres sabores, cada uno para un nivel diferente de verificación:

| Tipo | Qué Hace | Ideal Para |
|------|---------|-----------|
| `command` | Ejecuta un shell script, comprueba el código de salida | Linting, formatting, notificaciones |
| `prompt` | Envía un prompt a un modelo Claude para evaluación de un solo turno | Revisión ligera, comprobaciones de seguridad |
| `agent` | Lanza un subagente con acceso completo a herramientas (hasta 50 turnos) | Verificación profunda, comprobaciones en múltiples archivos |

Un hook `prompt` es perfecto para comprobaciones de "¿es seguro este código?" sin la sobrecarga de un subagente completo. Un hook `agent` puede leer archivos, ejecutar búsquedas y verificar condiciones en todo el codebase antes de permitir que una llamada de herramienta pase.

### Git hooks: la otra capa de seguridad

No te apoyes solo en los hooks de Claude Code. Los git hooks detectan problemas en el momento del commit independientemente de cómo se escribió el código:

- **pre-commit** — ejecuta linters, formatters, comprobaciones de tipos
- **commit-msg** — aplica el formato de commit convencional
- **pre-push** — ejecuta tests antes de hacer push

Los dos sistemas de hooks se complementan entre sí:
- **Los hooks de Claude Code** detectan problemas *durante* la sesión (antes de que Claude escriba un archivo)
- **Los git hooks** detectan problemas *en el momento del commit* (la compuerta final antes de que el código entre en el historial)

Juntos, crean defensa en profundidad. Los hooks de Claude Code evitan que se escriban patrones incorrectos. Los git hooks evitan que cualquier cosa que se cuele llegue a ser cometida.

### Patrones de hooks comunes

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review this file change for security issues (injection, XSS, exposed secrets). Return {\"ok\": true} if safe, {\"ok\": false, \"reason\": \"...\"} if not.",
            "model": "haiku"
          }
        ]
      }
    ]
  }
}
```

### El resultado

Con ambos sistemas de hooks en su lugar, tu flujo de trabajo tiene múltiples puntos de control:
- **Inicio de sesión:** herramientas activadas, memoria cargada, contexto listo
- **Durante el trabajo:** comprobaciones de seguridad en ediciones, formatting al guardar
- **En el commit:** linting, tests, formato del mensaje
- **Fin de sesión:** decisiones guardadas, memoria actualizada, nada perdido

---

## 12. Permisos y Seguridad

El sistema de permisos de Claude Code controla lo que Claude puede hacer sin pedirte permiso. La configuración correcta permite que Claude vaya rápido en operaciones seguras mientras mantiene los guardarraíles en todo lo peligroso.

### Los tres niveles

```json
{
  "permissions": {
    "allow": [
      "Read(**)",                          // Leer cualquier cosa
      "Edit(**)",                          // Editar cualquier cosa
      "Bash(git:*)",                       // Todos los comandos git
      "Bash(npm:*)",                       // Todos los comandos npm
      "mcp__plugin_serena_serena__*"       // Todas las herramientas de Serena
    ],
    "deny": [
      "Bash(sudo *)",                      // Nunca sudo
      "Bash(git push --force *)",          // Nunca force push
      "Read(.env)",                        // Nunca leer secretos
      "Read(**/*.key)",                    // Nunca leer archivos de clave
      "Read(~/.ssh/**)",                   // Nunca leer claves SSH
      "Read(~/.aws/**)",                   // Nunca leer credenciales AWS
      "Read(~/.kube/**)",                  // Nunca leer config de kube
      "Read(~/.npmrc)",                    // Nunca leer auth de npm
      "Read(~/.git-credentials)",          // Nunca leer credenciales de git
      "Edit(~/.bashrc)",                   // Nunca modificar config del shell
      "Edit(~/.zshrc)"                     // Nunca modificar config del shell
    ],
    "ask": [
      "Bash(curl * | bash*)"              // Preguntar antes de pipe-to-bash
    ]
  }
}
```

### Los principios

1. **Sé generoso con las lecturas** — Claude trabaja mejor cuando puede explorar libremente
2. **Sé estricto con las operaciones destructivas** — force push, borrado de esquemas, `rm -rf` en rutas del sistema
3. **Niega siempre el acceso a secretos** — `.env`, credenciales, claves, archivos PEM, SSH, AWS, configs de kube
4. **Wildcard para herramientas MCP** — `mcp__plugin_serena_serena__*` concede todas las herramientas de Serena de una vez
5. **Usa `ask` para zonas grises** — cosas que a veces están bien pero a veces son peligrosas

### Prevenir servidores MCP maliciosos en repos clonados

Los repositorios pueden incluir archivos `.mcp.json` que activan automáticamente servidores MCP cuando los abres. Esto es un riesgo de cadena de suministro — un repo malicioso podría inyectar herramientas. Desactiva la activación automática:

```json
{
  "enableAllProjectMcpServers": false
}
```

Con esto configurado, los servidores MCP a nivel de proyecto requieren opt-in explícito.

### Modo bypass (para usuarios avanzados)

Si confías en tu lista de deny y no quieres prompts de permiso para todo lo demás:

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions"
  },
  "skipDangerousModePermissionPrompt": true
}
```

Esto significa que Claude solo se detendrá a preguntar cuando encuentre algo explícitamente denegado. Rápido, pero requiere una lista de reglas deny bien configurada. No habilites esto hasta que tus reglas deny estén sólidas.

---

## 13. Subagentes y Equipos de Agentes

Claude Code puede lanzar instancias más pequeñas de Claude para manejar subtareas. Esta es una de las funcionalidades más potentes una vez que vas más allá de los valores por defecto.

### Selección de modelo: el cerebro adecuado para el trabajo

| Modelo | Mejor Para | El Trade-off |
|--------|-----------|-------------|
| **Haiku** | Búsqueda de archivos, consultas rápidas, exploración | Rápido y barato — pero puede perder matices |
| **Sonnet** | Implementación, tests, code review, QA estándar | Mejor equilibrio — tu herramienta diaria |
| **Opus** | Decisiones de arquitectura, código crítico de seguridad, refactorizaciones complejas | Razonamiento más profundo — vale la pena para trabajo de alto riesgo |

Añade esto a tu CLAUDE.md global:

```markdown
## Subagents
- haiku — exploration, quick lookups
- sonnet — implementation, tests, code review, standard QA
- opus — architecture, security-critical, complex refactors
```

### Definiciones de subagentes personalizados

Más allá de los agentes integrados, puedes definir tus propios agentes especializados en `.claude/agents/`. Son reutilizables entre sesiones y pueden acumular conocimiento con el tiempo.

Ejemplo — un revisor de código con conocimiento del proyecto:

```markdown
<!-- .claude/agents/code-reviewer.md -->
---
name: code-reviewer
description: Reviews code changes for bugs, security issues, and adherence to project conventions
model: sonnet
memory: project
tools:
  - Read
  - Grep
  - Glob
---

You are a code reviewer for this project. Review changes for:
- Logic errors and edge cases
- Security vulnerabilities
- Adherence to project conventions in CLAUDE.md
- Test coverage gaps

Be specific. Cite line numbers. Suggest fixes, don't just flag problems.
```

La configuración `memory: project` significa que este agente acumula conocimiento del codebase en `~/.claude/agent-memory/code-reviewer/` entre sesiones. Cuanto más lo uses, mejor entiende tu codebase.

Definir agentes que se adapten a tus necesidades específicas es un cambio radical — un revisor de seguridad, un escritor de tests, un asesor de arquitectura. Cada uno puede tener su propio modelo, herramientas y memoria.

### Equipos de Agentes

> *Todavía no he usado Agent Teams, pero vale la pena conocerlos para proyectos complejos.*

Los Agent Teams van más allá de los simples subagentes. En lugar de delegación unidireccional de padre a hijo, los equipos tienen:
- **Listas de tareas compartidas** con seguimiento de dependencias
- **Mensajería entre pares** entre agentes
- **Ejecución paralela** — cada miembro del equipo es una instancia completa de Claude Code

Piénsalo como pasar de "un desarrollador con asistentes" a "un pequeño equipo trabajando juntos."

Actívalo:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Mejores casos de uso: revisión de código en paralelo (revisores de seguridad + rendimiento + tests), desarrollo de módulos independientes, cambios entre capas donde frontend y backend necesitan moverse juntos.

**Guía de tamaño:** 3-5 miembros del equipo, 5-6 tareas por miembro. Cada miembro consume sus propios tokens, así que esto es una herramienta de poder — úsala cuando el paralelismo justifique el coste.

---

## 14. Gestión de la Ventana de Contexto

Incluso con una ventana de contexto de 1M — y especialmente al usar subagentes que mantienen limpia la sesión principal — vale la pena ser intencional sobre qué está en contexto. Una sesión enfocada con 200K de contexto relevante supera a una sesión saturada con 800K de ruido acumulado.

Dicho esto, la ventana de 1M más la delegación a subagentes significa que la gestión del contexto es menos una crisis de lo que solía ser. Estos son buenos hábitos, no técnicas de supervivencia.

### Los comandos clave

| Comando | Qué Hace | Cuándo Usarlo |
|---------|---------|--------------|
| `/compact` | Comprime el historial de conversación preservando contexto clave | Cuando la sesión se siente lenta o estás cambiando de subtarea |
| `/compact Focus on the API changes` | Compresión guiada — le dice a Claude qué preservar | Cuando quieres que contexto específico sobreviva |
| `/clear` | Borra completamente el historial de conversación | Entre tareas no relacionadas — el comando más infrautilizado |
| `/btw` | Haz una pregunta lateral que no entra en el historial | Consultas rápidas que no deberían crecer el contexto |
| `/context` | Muestra el desglose de tokens por categoría | Diagnosticar qué está consumiendo tu contexto |
| `Ctrl+B` | Pone en segundo plano un subagente en ejecución | Cuando un subagente tarda mucho y quieres hacer otra cosa |

### Hábitos prácticos

**Usa `/clear` agresivamente.** Mezclar tareas no relacionadas en una sesión es el mayor desperdicio de contexto. ¿Terminaste de depurar ese endpoint de API? `/clear` antes de empezar el trabajo de frontend. El coste de releer unos pocos archivos no es nada comparado con cargar 100K de contexto de depuración irrelevante.

**Las instrucciones personalizadas de `/compact` funcionan.** En lugar de solo `/compact`, prueba:

```text
/compact Keep the architectural decisions and test commands,
         summarize the debugging session
```

Esto guía qué sobrevive a la compresión. Útil cuando estás a punto de cambiar de enfoque pero quieres mantener decisiones específicas en contexto.

**`/btw` para consultas rápidas.** ¿Necesitas comprobar la firma de una función o un valor de configuración pero no quieres que ensucie tu conversación? `/btw what's the return type of getUserById?` — la respuesta aparece en un overlay descartable, nunca entra en el historial.

**Los subagentes son tu mejor defensa de contexto.** Delega trabajo exploratorio a subagentes. Se ejecutan en su propio contexto, devuelven solo la respuesta y mantienen limpia tu sesión principal. "Usa un subagente para investigar los fallos de test en el módulo de auth" es mejor que investigar directamente y llenar tu sesión con 50 lecturas de archivos.

### Qué sobrevive a la compactación

- CLAUDE.md: se relee desde disco en cada compactación — siempre sobrevive
- Auto Memory / MEMORY.md: siempre se carga — siempre sobrevive
- Tasks: sobreviven a la compactación — por eso importan
- Instrucciones de conversación: **NO sobreviven** a menos que se escriban en CLAUDE.md

Si una instrucción que le diste a Claude desapareció después de la compactación, solo estaba en la conversación. Escríbela en CLAUDE.md o en un skill si debe ser permanente.

---

## 15. Git Worktrees — Sesiones en Paralelo

Esta es una de las técnicas de productividad más potentes: ejecutar múltiples sesiones de Claude en el mismo repositorio sin que se pisen mutuamente los archivos.

### La idea

Un git worktree es un checkout separado del mismo repo en un directorio diferente, en su propia rama. Cada sesión de Claude trabaja en su propio worktree — archivos diferentes, rama diferente, sin conflictos.

### Soporte integrado

Claude Code tiene soporte de primera clase para worktrees, y el plugin Superpowers incluye un skill **Using Git Worktrees** que gestiona el flujo de trabajo:

```bash
# Iniciar una sesión en un nuevo worktree
claude --worktree feature-auth
```

Esto crea `.claude/worktrees/feature-auth/` con una rama dedicada.

### El patrón del usuario avanzado

Los equipos de alto rendimiento ejecutan 5-15 sesiones de Claude simultáneamente — algunas en pestañas de terminal, otras en la app web — cada una en su propio worktree. Frontend en una, backend en otra, tests en una tercera. Todas trabajan en paralelo sin conflictos, y cada sesión abre su propio PR cuando termina.

### Consejos de configuración

**Añade el directorio de worktrees al gitignore:**

```gitignore
.claude/worktrees/
```

**Copia archivos env en nuevos worktrees** con `.worktreeinclude`:

```text
# .worktreeinclude — archivos en gitignore que se copian en cada nuevo worktree
.env.local
.env.development
```

Solo se copian los archivos en gitignore — los archivos tracked vienen del checkout de git automáticamente.

### Worktrees de subagentes

También puedes dar a los subagentes sus propios worktrees para trabajo paralelo verdaderamente aislado:

```markdown
<!-- .claude/agents/my-agent.md -->
---
isolation: worktree
---
```

El worktree se limpia automáticamente si el agente no hace cambios.

---

## 16. Git y Code Review

### Commits convencionales

Define tu formato una vez en el CLAUDE.md global y nunca más vuelvas a pensar en ello:

```markdown
## Git
Commits: <type>(<scope>): <description> (50 char max subject)
Body: blank line + bullet list, 72 char wrap
Types: feat fix docs style refactor perf test build ci chore revert
Rules: Never commit to main. Never force push. Never skip hooks.
git commit is auto-approved. git push requires confirmation.
```

### Code review: usa ambos enfoques

La mejor configuración de code review usa **dos enfoques complementarios**:

**1. [CodeRabbit](https://www.coderabbit.ai/) — revisión de IA en la nube sobre PRs**

CodeRabbit revisa tus pull requests automáticamente en GitHub. Detecta issues desde una perspectiva fresca — no ha visto el contexto de tu sesión, así que encuentra cosas que tú y Claude habéis pasado por alto.

```bash
claude plugin install coderabbit@claude-plugins-official
```

```text
/coderabbit:review    — Ejecutar revisión CodeRabbit en los cambios actuales
```

**Flujo de trabajo para las revisiones de CodeRabbit:**
1. Aborda todos los comentarios — incluyendo las sugerencias "nice to have"
2. Resuelve cada hilo de revisión después de arreglarlo
3. Publica `@coderabbitai resolve` para limpiar los comentarios pendientes
4. Publica `@coderabbitai full review` para una nueva pasada
5. CodeRabbit cambia a APPROVED cuando todos los hilos están limpios

**2. Revisiones locales con subagentes — feedback inmediato y contextual**

Claude Code puede lanzar subagentes de revisión que comprueban tu trabajo contra el plan y los estándares del proyecto dentro de tu sesión. El plugin Superpowers hace esto automáticamente después de completar pasos importantes de implementación.

También puedes activarlo manualmente:

```text
/code-review
```

O define un agente de revisión personalizado (ver [Sección 13](#13-subagentes-y-equipos-de-agentes)) que acumule conocimiento sobre tu codebase con el tiempo.

**¿Por qué ambos?** Las revisiones locales son rápidas y contextuales — conocen el plan y detectan desviaciones inmediatamente. Las revisiones de CodeRabbit son frescas e independientes — detectan issues que el sesgo de sesión pasaría por alto. Juntos, dan defensa en profundidad.

---

## 17. Gestión de Sesiones

Las sesiones en Claude Code no son desechables — puedes reanudarlas, nombrarlas y navegar entre ellas. Esto importa para trabajo de varios días y cuando necesitas cambiar de contexto.

### Comandos clave

| Comando | Qué Hace |
|---------|---------|
| `claude --continue` | Reanuda la sesión más reciente |
| `claude --resume` | Abre un selector de sesiones para elegir cuál reanudar |
| `claude -n auth-refactor` | Inicia una sesión con nombre (más fácil de encontrar después) |
| `/rename auth-refactor` | Renombra la sesión actual |
| `claude --from-pr 123` | Reanuda una sesión vinculada a un PR específico |

### Atajos del selector de sesiones

En el selector `/resume`:
- **`P`** — previsualizar una sesión
- **`R`** — renombrar
- **`B`** — filtrar a la rama git actual
- **`A`** — alternar entre el directorio actual y todos los proyectos

### El prefijo `!`

¿Necesitas ejecutar un comando de shell rápido sin gastar un round-trip de llamada a herramienta? Prefixa con `!`:

```text
! git status
! npm test
! ls -la src/
```

La salida llega directamente al contexto de la conversación. Más rápido que pedirle a Claude que lo ejecute, y a veces es todo lo que necesitas.

---

## 18. Línea de Estado — Ve Qué Está Pasando

Claude Code soporta una línea de estado personalizada en la parte inferior del terminal que muestra información en tiempo real sobre tu sesión. Configúrala con un shell script en `~/.claude/statusline.sh`.

Una buena línea de estado muestra:
- **Uso de contexto** — qué tan llena está la ventana de contexto (verde < 50%, amarillo 50-79%, rojo 80%+)
- **Coste de la sesión** — cuánto has gastado en esta sesión
- **Tiempo transcurrido** — cuánto tiempo llevas trabajando
- **Tasa de aciertos de caché de prompts** — qué tan eficientemente se está reutilizando el contexto

Esto te da consciencia inmediata de cuándo hacer `/compact` o `/clear`, y te ayuda a desarrollar intuición sobre qué es caro y qué es barato.

---

## 19. Seguimiento de Tareas — No Pierdas el Hilo

Cuando tu conversación se alarga, Claude Code comprime los mensajes más antiguos para mantenerse dentro de los límites de contexto. Normalmente está bien — pero significa que Claude puede olvidar en qué punto estaba de un plan con varios pasos.

Las tareas sobreviven a esta compresión. Siempre son visibles, incluso después de la compactación.

```text
TaskCreate   — Crear una tarea
TaskUpdate   — Marcar in_progress, completed o blocked
TaskList     — Ver todas las tareas actuales
```

**La práctica:** Para cualquier trabajo con más de 2-3 pasos, crea las tareas por adelantado. Marca cada una como `in_progress` cuando la empieces, `completed` cuando termines — inmediatamente, no en lote al final.

> Las tareas son para la sesión actual. Para información que debe persistir entre sesiones, usa MEMORY.md o Graphiti.

---

## 20. Patrones de Fallo Comunes — Qué Evitar

Estos son los errores que más tiempo hacen perder con Claude Code. Cada uno es fácil de caer en él y fácil de corregir una vez que sabes qué buscar.

### La sesión todo en uno

**El patrón:** Depuras un endpoint de API, luego arreglas un bug de CSS, luego escribes una migración, luego refactorizas un test — todo en la misma sesión. Al final, Claude lleva 500K de contexto mezclado y sus respuestas se están volviendo vagas.

**La solución:** `/clear` entre tareas no relacionadas. Se siente un desperdicio ("¡acababa de tener todo ese contexto!") pero un inicio fresco con el contexto adecuado supera a una sesión saturada en todo momento.

### El bucle de corrección

**El patrón:** Claude hace algo mal. Lo corriges. Lo vuelve a hacer mal, de forma ligeramente diferente. Lo corriges de nuevo. Tres rondas después, estás frustrado y Claude está confundido — cada corrección añadió contexto contradictorio.

**La solución:** Después de dos correcciones fallidas sobre el mismo problema, para de corregir. `/clear` y escribe un prompt inicial mejor. El problema no es la comprensión de Claude — es que la instrucción original no era suficientemente clara, y cada corrección está haciendo el contexto más ruidoso.

### El CLAUDE.md sobrespecificado

**El patrón:** Tu CLAUDE.md tiene 800 líneas de reglas detalladas. Claude ignora la mitad porque las reglas importantes están enterradas en el ruido. Añades más reglas para compensar. Empeora.

**La solución:** Cada línea en CLAUDE.md debería pasar la prueba: *¿el hecho de eliminarla haría que Claude cometiese un error específico?* Si Claude ya hace lo correcto sin una regla, elimina la regla. Apunta a ~100-200 líneas. Mueve el conocimiento profundo a skills que se carguen bajo demanda.

### La brecha confianza-sin-verificación

**El patrón:** Claude escribe código que parece plausible. Lo miras por encima, dices "se ve bien" y sigues. Más tarde descubres que no maneja casos edge, o que sutilmente rompió algo más.

**La solución:** Siempre proporciona criterios de verificación — tests que ejecutar, scripts que correr, comportamientos específicos que comprobar. El skill de verificación de Superpowers hace esto automáticamente, pero también puedes simplemente decir: "Antes de terminar, ejecuta la suite de tests y muéstrame la salida."

### La exploración infinita

**El patrón:** Le pides a Claude que "investigue los problemas de rendimiento." Claude lee 80 archivos, ejecuta 30 búsquedas y llena tu contexto con un análisis exhaustivo que no necesitabas.

**La solución:** Acota tus peticiones. "Comprueba si la consulta N+1 en getUserOrders está causando la lentitud" en lugar de "investiga el rendimiento." O delega a un subagente: "Usa un subagente para investigar los problemas de rendimiento en el módulo de pedidos" — la investigación ocurre en un contexto separado y devuelve solo los hallazgos.

### La mentira "añadiré los tests después"

**El patrón:** Claude implementa una función sin tests. Planeas añadirlos después. Nunca lo haces. El siguiente cambio rompe la función silenciosamente.

**La solución:** El flujo de trabajo TDD de Superpowers maneja esto automáticamente — los tests van primero, no al final. Si no estás usando Superpowers, conviértelo en una regla en tu CLAUDE.md: "Escribe tests que fallen antes de la implementación. Sin excepciones."

---

## 21. Config Sync — La Misma Configuración en Todas Partes

Si desarrollas en más de una máquina — digamos, un portátil del trabajo y uno personal — quieres que tu configuración de Claude Code sea la misma en ambas. El mismo CLAUDE.md, los mismos hooks, los mismos permisos, los mismos plugins.

### cc-config-sync

[cc-config-sync](https://www.npmjs.com/package/cc-config-sync) es una herramienta CLI que almacena tu configuración de Claude Code en un repositorio git y empuja/tira entre máquinas:

```bash
npm install -g cc-config-sync

cc-config-sync pull        # Captura la config de la máquina actual en el repo
cc-config-sync push --yes  # Aplica la config del repo a la máquina actual
cc-config-sync status      # Comprueba qué es diferente
cc-config-sync push --dry-run  # Previsualiza sin escribir
```

### Qué se sincroniza

- `~/.claude/CLAUDE.md` (instrucciones globales)
- `~/.claude/settings.json` (permisos, hooks, plugins)
- `~/.claude/hooks/*.sh` (scripts de hooks)
- `~/.claude/plugins/installed_plugins.json`
- Archivos CLAUDE.md por proyecto, settings y archivos de memoria

### Una cosa que vigilar

`settings.local.json` contiene sobreescrituras específicas de la máquina — como rutas que difieren entre máquinas. Revísalas antes de hacer push entre entornos. Lo que funciona en el layout del sistema de archivos de una máquina puede no aplicar a otra.

---

## Por Dónde Empezar

Si estás configurando desde cero, este es el orden que te da más valor más rápido:

1. **Instala los plugins principales** — superpowers, claude-md-management, skill-creator, context7, serena, code-review, coderabbit
2. **Genera tu primer CLAUDE.md** — usa `/init` para el esqueleto, luego refina con `/claude-md-improver`
3. **Añade la tabla de selección de herramientas** a tu CLAUDE.md global — esta sola tabla cambia cómo trabaja Claude
4. **Configura los permisos** — permite lecturas libremente, niega lo peligroso y los secretos
5. **Configura hooks** — inicio de sesión para activación automática de herramientas, fin de sesión para guardar memoria
6. **Escribe tu primer CLAUDE.md de proyecto** — arquitectura, convenciones, archivos clave, comandos de build
7. **Configura Serena** para tu proyecto principal — el ahorro de tokens empieza inmediatamente
8. **Configura Graphiti** para memoria persistente — las decisiones dejan de perderse entre sesiones
9. **Crea tu primer skill** — toma algo que sigues explicando y conviértelo en bajo demanda
10. **Confía en Superpowers** — deja que guíe los flujos de trabajo de planificación, TDD y verificación

Cada paso se construye sobre los anteriores. No tienes que hacerlos todos de una vez — incluso los pasos 1-5 harán una diferencia notable.

---

## Recursos de la Comunidad

- [Documentación de Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Plugins de Claude Code](https://github.com/anthropics/claude-code-plugins)
- [CodeRabbit — AI Code Review](https://www.coderabbit.ai/)
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — lista curada de skills, hooks, agentes, plugins
- [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) — 135+ agentes, 400K+ skills
- [Trail of Bits Claude Code Config](https://github.com/trailofbits/claude-code-config) — configuración con enfoque en seguridad
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
- [Graphiti Knowledge Graph](https://github.com/getzep/graphiti)
- [cc-config-sync](https://www.npmjs.com/package/cc-config-sync)
