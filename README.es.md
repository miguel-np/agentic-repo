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

## Cómo funciona

Cada workflow está implementado como un **workflow reutilizable** (definido aquí) con un **caller** correspondiente (un archivo de ~15 líneas en `caller-workflows/`). Los repositorios destino solo necesitan el caller ligero; toda la lógica vive en un único lugar y las actualizaciones se aplican automáticamente.

Los workflows utilizan **GitHub Copilot CLI** con el GitHub MCP Server para leer el contexto del repositorio, interactuar con la API de GitHub y producir resultados significativos y adaptados al proyecto.

## Requisitos previos

- Cuenta de GitHub con acceso a **GitHub Copilot**
- Un **Personal Access Token (PAT)** con permisos de Copilot, almacenado como secreto del repositorio con el nombre `COPILOT_PAT`

## Instalación

### Opción A — Caller workflows (recomendado)

Copia los archivos de `caller-workflows/` al directorio `.github/workflows/` del repositorio destino:

```bash
git clone https://github.com/<tu-org>/agentic-repo.git
cp agentic-repo/caller-workflows/*.yml <repo-destino>/.github/workflows/
```

A continuación, añade el secreto `COPILOT_PAT` al repositorio destino.

### Opción B — Workflow Installer

1. Ve a **Actions** → **Workflow Installer**
2. Introduce los repositorios destino (separados por coma): `owner/repo1, owner/repo2`
3. Selecciona qué workflows instalar
4. Elige si crear un PR o hacer push directamente a la rama principal
5. Ejecuta el workflow

### Opción C — Copia completa

Copia los workflows de `.github/workflows/` directamente si prefieres tener la implementación completa bajo tu control.

## Uso

### Issue Quality Enhancer

Se activa automáticamente cuando se abre un issue. Solo se ejecuta para issues creados por el propietario del repositorio. Mejora el título y el cuerpo, añade etiquetas y deja un comentario de resumen.

### Smart Labeler

Se activa en eventos de apertura y edición de issues y PRs. Compara las etiquetas existentes con la lista de etiquetas del repositorio y las ajusta según el contenido.

### Copilot Suggester

Ejecución manual desde la pestaña Actions. Selecciona la categoría de sugerencias (`all`, `security`, `performance`, etc.). Copilot escanea el código, comprueba si ya existen issues relacionados y abre entradas en Discussions para hallazgos relevantes, omitiendo sugerencias que ya están registradas.

### Label Beautifier

Ejecución manual. Establece `dry_run: true` para previsualizar los cambios propuestos antes de aplicarlos. Renombra, cambia el color y añade descripciones a todas las etiquetas sin romper las asociaciones existentes con issues y PRs.

### Continuous Documentation

Se activa automáticamente en pull requests que modifican archivos de código. Compara el diff con las secciones del README y los docs de API, edita los archivos directamente en la rama del PR para corregir la desviación y publica un comentario de resumen.

### Workflow Installer

Ejecución manual. Instala o convierte workflows en uno o más repositorios destino. Si ya existe un workflow standalone completo, se convierte en caller automáticamente.

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
│       └── workflow-installer.yml
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

## Tecnologías

- [GitHub Actions](https://docs.github.com/es/actions) — orquestación de workflows
- [GitHub Copilot CLI](https://docs.github.com/es/copilot/using-github-copilot/using-github-copilot-in-the-command-line) — ejecución de tareas con IA
- [GitHub MCP Server](https://github.com/github/github-mcp-server) — acceso estructurado a la API de GitHub para Copilot
- [Playwright MCP Server](https://github.com/microsoft/playwright-mcp) — automatización de navegador para análisis de UI (Copilot Suggester)

---

**[Read in English](README.md)**
