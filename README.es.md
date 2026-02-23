# Agentic Repo

Una colección de GitHub Actions workflows reutilizables que utilizan **GitHub Copilot CLI** para automatizar tareas habituales de mantenimiento de repositorios: triaje de issues, etiquetado, sincronización de documentación y más.

## Workflows

| Workflow | Disparador | Descripción |
|---|---|---|
| **Issue Quality Enhancer** | Issue abierto | Normaliza títulos y cuerpos de issues: traduce al inglés, añade estructura, inserta referencias de código relevantes y asigna etiquetas |
| **Smart Labeler** | Issue / PR abierto o editado | Analiza el contenido y aplica las etiquetas existentes más apropiadas |
| **Copilot Suggester** | Manual | Escanea el código y abre ideas en GitHub Discussions sobre seguridad, rendimiento, UX y otras mejoras |
| **Label Beautifier** | Manual | Renombra las etiquetas para seguir un esquema consistente de emojis y colores; admite modo dry-run |
| **Continuous Documentation** | PR abierto | Detecta desviación entre cambios de código y README / docs de API; aplica correcciones directamente en la rama del PR |
| **Workflow Installer** | Manual | Despliega los caller workflows de este repositorio a uno o más repositorios destino |
| **Lint Workflows** | Push / PR | Valida la sintaxis YAML de todos los archivos de workflow |

## Cómo funciona

Cada workflow está implementado como un **workflow reutilizable** (definido aquí) con un **caller** correspondiente (un archivo ligero en `caller-workflows/`). Los repositorios destino solo necesitan el caller; toda la lógica vive en un único lugar y las actualizaciones se aplican automáticamente.

Los archivos caller contienen un placeholder `__OWNER__` en la referencia `uses:`. Cuando se instalan a través del **Workflow Installer**, el placeholder se reemplaza automáticamente por tu usuario u organización de GitHub. Si instalas los callers manualmente, reemplaza `__OWNER__` por el propietario de tu fork de este repositorio.

Los workflows utilizan **GitHub Copilot CLI** con el GitHub MCP Server para leer el contexto del repositorio, interactuar con la API de GitHub y producir resultados significativos y adaptados al proyecto.

## Requisitos previos

- Cuenta de GitHub con acceso a **GitHub Copilot**
- Un **Personal Access Token (PAT)** con permisos de Copilot, almacenado como secreto del repositorio con el nombre `COPILOT_PAT`

## Instalación

### Opción A — Caller workflows (recomendado)

Copia los archivos de `caller-workflows/` al directorio `.github/workflows/` del repositorio destino:

```bash
git clone https://github.com/<tu-org>/agentic-repo.git
# Reemplaza __OWNER__ por tu organización/usuario antes de copiar
sed -i 's/__OWNER__/<tu-org>/g' agentic-repo/caller-workflows/*.yml
cp agentic-repo/caller-workflows/*.yml <repo-destino>/.github/workflows/
```

A continuación, añade el secreto `COPILOT_PAT` al repositorio destino.

### Opción B — Workflow Installer

1. Ve a **Actions** → **Workflow Installer**
2. Introduce los repositorios destino (separados por coma): `owner/repo1, owner/repo2`
3. Selecciona qué workflows instalar
4. Elige si crear un PR o hacer push directamente a la rama principal
5. Ejecuta el workflow

El installer reemplaza automáticamente `__OWNER__` por el valor correcto.

### Opción C — Copia completa

Copia los workflows de `.github/workflows/` directamente si prefieres tener la implementación completa bajo tu control. En este modo cada workflow se ejecuta de forma autónoma, sin necesidad de caller.

## Configuración

### Filtro de autor

Por defecto, los workflows **Issue Quality Enhancer**, **Smart Labeler** y **Continuous Documentation** solo se ejecutan cuando el autor del issue/PR es el propietario del repositorio. Puedes cambiar este comportamiento con el input `author_filter`:

| Valor | Comportamiento |
|---|---|
| `owner_only` | (por defecto) Solo el propietario del repositorio activa el workflow |
| `collaborators` | Cualquier usuario con acceso write/maintain/admin puede activarlo |
| `all` | Cualquier autor activa el workflow — usar con precaución |

Ejemplo de caller con filtro personalizado:

```yaml
jobs:
  enhance:
    uses: tu-org/agentic-repo/.github/workflows/issue-quality-enhancer.yml@main
    with:
      issue_number: ${{ github.event.issue.number }}
      issue_title: ${{ github.event.issue.title }}
      issue_body: ${{ github.event.issue.body }}
      issue_author: ${{ github.event.issue.user.login }}
      author_filter: collaborators
    secrets: inherit
```

### Análisis de UI (Copilot Suggester)

El Suggester puede compilar, ejecutar y analizar visualmente tu aplicación mediante Playwright. Está habilitado por defecto, pero se puede desactivar con `analyze_ui: false` si tu proyecto no tiene interfaz web o quieres un escaneo más rápido solo de código.

## Uso

### Issue Quality Enhancer

Se activa automáticamente cuando se abre un issue. Mejora el título y el cuerpo, añade etiquetas y deja un comentario de resumen.

### Smart Labeler

Se activa en eventos de apertura y edición de issues y PRs. Compara las etiquetas existentes con la lista de etiquetas del repositorio y las ajusta según el contenido. Cuando este workflow y el Issue Quality Enhancer se ejecutan sobre el mismo issue, el Smart Labeler detecta que el Enhancer ya lo procesó y se salta la ejecución para evitar conflictos.

### Copilot Suggester

Ejecución manual desde la pestaña Actions. Selecciona la categoría de sugerencias (`all`, `security`, `performance`, etc.) y si analizar la UI. Copilot escanea el código, comprueba si ya existen issues y discussions relacionados, y abre entradas en Discussions para hallazgos relevantes, omitiendo sugerencias que ya están registradas.

### Label Beautifier

Ejecución manual. Establece `dry_run: true` para previsualizar los cambios propuestos antes de aplicarlos. Renombra, cambia el color y añade descripciones a todas las etiquetas sin romper las asociaciones existentes con issues y PRs.

### Continuous Documentation

Se activa automáticamente en pull requests que modifican archivos de código. También disponible como workflow autónomo (sin necesidad de caller). Compara el diff con las secciones del README y los docs de API, edita los archivos directamente en la rama del PR para corregir la desviación y publica un comentario de resumen.

### Workflow Installer

Ejecución manual. Instala o convierte workflows en uno o más repositorios destino. Si ya existe un workflow standalone completo, se convierte en caller automáticamente. Acepta URLs completas de GitHub, `owner/repo` o simplemente `repo` (usa el owner actual por defecto).

## Estructura del proyecto

```
agentic-repo/
├── .github/
│   └── workflows/
│       ├── issue-quality-enhancer.yml
│       ├── smart-labeler.yml
│       ├── the-suggester-discussion-mode.yml
│       ├── label-beautifier.yml
│       ├── continuous-docs.yml
│       ├── workflow-installer.yml
│       └── lint.yml
├── caller-workflows/
│   ├── issue-quality-enhancer.yml
│   ├── smart-labeler.yml
│   ├── the-suggester-discussion-mode.yml
│   ├── label-beautifier.yml
│   └── continuous-docs.yml
├── README.md
└── README.es.md
```

## Secreto requerido

| Secreto | Ámbito | Descripción |
|---|---|---|
| `COPILOT_PAT` | Repositorio | PAT de GitHub con acceso a Copilot. Necesario en cada repositorio que invoque estos workflows. |

## Consideraciones de seguridad

Todas las invocaciones de Copilot CLI usan el flag `--allow-all-tools`, que otorga al agente de IA acceso sin restricciones a todas las herramientas disponibles en su entorno de ejecución (GitHub MCP Server, Playwright, sistema de archivos, etc.). Combinado con el secreto `COPILOT_PAT`, el agente puede leer y escribir contenido del repositorio, issues, etiquetas, discussions y ramas de PR.

**Recomendaciones:**

- Usa un PAT con los **scopes mínimos necesarios** (típicamente `repo`, `copilot`).
- Empieza con `author_filter: owner_only` (el valor por defecto) y amplía solo después de evaluar el riesgo.
- Revisa la salida de Copilot CLI en los logs de Actions después de cada ejecución.
- Para repositorios sensibles, ejecuta el Suggester y el Label Beautifier en modo `dry_run` primero.

## Tecnologías

- [GitHub Actions](https://docs.github.com/es/actions) — orquestación de workflows
- [GitHub Copilot CLI](https://docs.github.com/es/copilot/using-github-copilot/using-github-copilot-in-the-command-line) — ejecución de tareas con IA
- [GitHub MCP Server](https://github.com/github/github-mcp-server) — acceso estructurado a la API de GitHub para Copilot
- [Playwright MCP Server](https://github.com/microsoft/playwright-mcp) — automatización de navegador para análisis de UI (Copilot Suggester)

---

**[Read in English](README.md)**
