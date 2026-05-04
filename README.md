# Omnistack

<p align="center">
  <strong>Context Token Optimization Pipeline for multi-repository AI workflows</strong>
</p>

<p align="center">
  <a href="#overview">Overview</a> - 
  <a href="#why-omnistack">Why Omnistack</a> - 
  <a href="#architecture">Architecture</a> - 
  <a href="#tooling">Tooling</a> - 
  <a href="#installation">Installation</a> - 
  <a href="#master-script">Master Script</a> - 
  <a href="#gemini-project-rules-omnistack-senior-edition">Project Rules</a> - 
  <a href="#upload-to-github">Upload to GitHub</a> - 
  <a href="#license">License</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License: MIT">
  <img src="https://img.shields.io/badge/status-stable-brightgreen.svg" alt="Status: Stable">
  <img src="https://img.shields.io/badge/context-multi--repo-blue.svg" alt="Multi-repo context">
</p>

***

## Overview

Omnistack is a high-performance context optimization architecture for Senior and Staff engineers working across large multi-repo ecosystems in AI coding environments such as Gemini or Google Antigravity.

It addresses **context obesity** through a discovery hierarchy: first understand the global architecture map, then extract only the exact code blocks needed for the task. This reduces token consumption dramatically while keeping debugging, refactoring, and cross-repo analysis precise.

## Why Omnistack

Omnistack is designed for teams and individual engineers who need to operate across 10+ repositories without flooding the model context with tests, logs, backups, or unrelated implementation files.

### Core use cases

- **Cross-repo navigation:** trace dependencies, contracts, and data flows between services.
- **High-density debugging:** follow a bug from trigger to persistence across multiple repositories.
- **Safe refactoring:** change method signatures or interfaces and identify affected consumers precisely.
- **Cost efficiency:** stay under large-context thresholds and avoid wasting tokens on irrelevant files.

## Architecture

Omnistack follows a layered workflow:

1. **Skeleton Map** — build a global architectural map with `repomix`.
2. **Discovery** — use `ag` for zero-token local search.
3. **Surgical Extraction** — use `sg` to extract only relevant AST-level blocks.
4. **Incremental Memory** — maintain `.cce_context.json` to skip unchanged files.

### Context hierarchy

- `global_skeleton.md` is the architecture map and source of truth.
- `.cce_context.json` is the lightweight memory index for changed files.
- Full file reads are the last resort, not the default behavior.

## Tooling

| Tool | Role | Install (macOS) |
|---|---|---|
| `repomix` | Global skeleton map generation | `npm install -g repomix` |
| `ag` | Local zero-token discovery | `brew install the_silver_searcher` |
| `sg` | AST-based surgical extraction | `brew install ast-grep` |
| `python3` | Memory indexing for CCE-Lite | Native |

## Installation

1. Install dependencies:
   ```bash
   brew install the_silver_searcher ast-grep
   npm install -g repomix
   ```
2. Save the master script as `/usr/local/bin/omnistack`.
3. Make it executable:
   ```bash
   chmod +x /usr/local/bin/omnistack
   ```
4. Place all repositories under one root workspace.
5. Create `repomix.config.json` in that root.
6. Run:
   ```bash
   omnistack
   ```

## Master Script

The script below generates the skeleton map, validates token budget, and refreshes the incremental memory index.

```bash
#!/bin/bash

# Configuración de Colores
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m' 

echo -e "${GREEN}🚀 Iniciando Sincronización Omnistack (v2.1 - Stable)${NC}"

# VALIDACIÓN DE DIRECTORIO RAÍZ
if [ ! -f "repomix.config.json" ]; then
    echo -e "${RED}❌ Error: No se detectó repomix.config.json en este directorio.${NC}"
    echo "Asegúrate de estar en la raíz de tus repositorios."
    exit 1
fi

# Validación de Dependencias
check_dep() {
    if ! command -v $1 &> /dev/null; then
        echo -e "${RED}❌ Error: $1 no está instalado.${NC} Ejecuta: $2"
        exit 1
    fi
}
check_dep "repomix" "npm install -g repomix"
check_dep "ag" "brew install the_silver_searcher"
check_dep "sg" "brew install ast-grep"

# EJECUCIÓN REPOMIX
echo -e "${YELLOW}📦 Generando Skeleton Map...${NC}"
repomix

if [ $? -ne 0 ]; then
    echo -e "${RED}❌ Error en Repomix. Abortando pipeline.${NC}"
    exit 1
fi

# TOKEN GUARD (Estimación 1 token ≈ 4 chars)
if [ -f "global_skeleton.md" ]; then
    CHARS=$(wc -c < "global_skeleton.md")
    EST_TOKENS=$((CHARS / 4))
    if [ $EST_TOKENS -gt 200000 ]; then
        echo -e "${RED}⚠️  ALERTA: ~$EST_TOKENS tokens detectados. Mapa pesado.${NC}"
    else
        echo -e "📊 Contexto saludable: ${GREEN}~$EST_TOKENS tokens${NC}"
    fi
fi

# CCE-LITE: INDEXADOR PYTHON ROBUSTO (CHUNKED READING)
echo -e "${YELLOW}🧠 Sincronizando Memoria CCE...${NC}"
python3 << 'EOF'
import os, hashlib, json

index = {}
exts = ('.py', '.js', '.ts', '.sql', '.yml', '.yaml')
ignored_dirs = {'.git', 'node_modules', 'dist', 'venv', '__pycache__', 'infraestructura'}

def get_file_hash(path):
    hasher = hashlib.md5()
    try:
        with open(path, 'rb') as f:
            while chunk := f.read(4096):
                hasher.update(chunk)
        return hasher.hexdigest()
    except Exception:
        return None

for root, dirs, files in os.walk('.'):
    dirs[:] = [d for d in dirs if d not in ignored_dirs]
    for f in files:
        if f.endswith(exts):
            full_path = os.path.join(root, f)
            clean_path = os.path.relpath(full_path, '.')
            f_hash = get_file_hash(full_path)
            if f_hash:
                index[clean_path] = f_hash

with open('.cce_context.json', 'w') as f:
    json.dump(index, f, indent=2)
EOF

if [ $? -eq 0 ]; then
    echo -e "${GREEN}✅ Memoria .cce_context.json actualizada.${NC}"
else
    echo -e "${RED}❌ Error en el indexador de memoria.${NC}"
    exit 1
fi

echo -e "${GREEN}🏁 Pipeline Omnistack listo para Antigravity.${NC}"
```

## Gemini Project Rules: Omnistack Senior Edition

This section can be copied into global IDE rules or into a `.gemini_rules.md` file.

### Workflow Orchestration

#### 1. Plan Mode & Verification
- **Default Plan:** Entra en modo planificación para cualquier tarea no trivial (3+ pasos o decisiones arquitectónicas).
- **Fail-Fast:** Si el plan falla, DETENTE y re-planifica inmediatamente. No intentes "forzar" una solución errónea.
- **Verification:** Nunca marques una tarea como completa sin probar que funciona. Ejecuta tests, revisa logs y demuestra la corrección técnica.

#### 2. Subagent Strategy
- **Isolation:** Usa subagentes para investigación o análisis paralelo para mantener limpio el contexto principal.
- **Subagent Hygiene:** Al invocar subagentes, pásales el contexto del `global_skeleton.md` para evitar exploraciones ciegas y costosas.

#### 3. Self-Improvement Loop
- **Lessons Learned:** Tras cada corrección del usuario, actualiza `tasks/lessons.md`.
- **Iteration:** Revisa las lecciones al inicio de cada sesión para evitar repetir errores técnicos o de flujo.

#### 4. Demand Elegance
- **Senior Standards:** Antes de proponer un cambio, pregunta: "¿Es esta la forma más elegante?".
- **No Hacks:** Si una solución se siente "sucia", busca la implementación técnica superior basándote en patrones de diseño modernos.

### Task Management & Principles

1. **Plan First**: Escribe el plan en `tasks/todo.md`.
2. **Track Progress**: Marca ítems completados conforme avances.
3. **Simplicity First**: Cambios mínimos, impacto máximo. Evita el over-engineering.
4. **No Laziness**: Encuentra la causa raíz (Root Cause Analysis). No apliques "parches" temporales.

### Tech Stack & Commands

- **Backend:** Python / Node.js.
- **Testing:** Ejecuta siempre `pytest` (Python) o los tests correspondientes.
- **Linter/Format:** Ejecuta `python -m black .` e `isort .` antes de dar un archivo por terminado.
- **Token Discipline:** Si el archivo `global_skeleton.md` supera los **200k tokens**, notifica al usuario para realizar una poda en `repomix.config.json`.
- **Memory Limit:** Mantén `tasks/lessons.md` bajo 60 líneas; consolida lecciones antiguas en principios generales si excede este límite.

### OMNISTACK UNIFIED PROTOCOL (Surgical & Incremental)

#### 1. Context Hierarchy & CCE
- **Memory First:** Consulta `.cce_context.json` antes de procesar. **NO** re-leas archivos cuyo hash MD5 no haya cambiado desde la última sincronización.
- **Documento Maestro:** `global_skeleton.md` es la única fuente de verdad para la arquitectura multi-repo. Trátalo como un mapa de firmas (clases/métodos), no como código completo.

#### 2. Exploration Strategy (Zero-Waste)
- **Step A (Localizar):** Usa `ag` (Silver Searcher) para encontrar la línea y archivo exactos. Coste: **0 tokens**.
- **Step B (Extraer):** Si el archivo es extenso (>200 líneas), usa `sg` (ast-grep) para extraer solo el bloque lógico relevante.
  - *Ejemplo:* `sg -p 'class Name { $$$ }' path/file.py`
- **Step C (Leer):** Solo haz `cat` del archivo completo si la edición afectará a múltiples secciones dispersas de forma simultánea.

#### 3. Multi-Repo Isolation
- **Context Tagging:** Especifica siempre el repositorio en cada propuesta: `[Repo-Name] path/to/file`.
- **Cross-Repo Dependencies:** Si un cambio en el Repo A afecta al Repo B, documenta la dependencia en `tasks/todo.md` antes de proceder.

#### 4. Execution & Output (Caveman + RTK)
- **Validation:** Ejecuta siempre `pytest | rtk`. El alias `rtk` filtra el ruido y entrega solo los fallos críticos.
- **Caveman Mode:** Respuestas directas, técnicas y sin cortesías. Entrega solo diffs de código o bloques de lógica densa. No incluyas prosa introductoria ni conclusiones innecesarias.

## Suggested Workspace Layout

```text
workspace-root/
├── repomix.config.json
├── global_skeleton.md
├── .cce_context.json
├── tasks/
│   ├── todo.md
│   └── lessons.md
├── repo-a/
├── repo-b/
└── repo-c/
```


## Operating Model

1. Run `omnistack` to refresh architecture and memory.
2. Search with `ag` before reading large files.
3. Extract with `sg` before opening full files.
4. Edit only when the correct scope is confirmed.
5. Validate with tests and formatter before declaring completion.

## Contributing

Contributions should preserve the core principles of Omnistack: minimal context waste, precise extraction, and disciplined multi-repo reasoning.

When proposing changes, prefer improvements that strengthen reproducibility, observability, and token efficiency.

## License

This project is licensed under the MIT License. See the `LICENSE` file for the full text.
