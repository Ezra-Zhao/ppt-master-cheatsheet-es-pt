# 🎞️ PPT Master — Guía rápida (Español)

> Comandos verificados contra el README y la guía Getting Started de [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master) (2026-10-12).
> Qué es: le das un PDF, un DOCX, una página web o un texto pegado, y un agente de IA lo convierte en una **presentación de PowerPoint nativa**: formas, cuadros de texto, gráficos y tablas reales, que puedes pulsar y seguir editando. Nada de diapositivas como imagen.

---

## 0. Requisito

Solo **Python 3.10+**. El resto llega con `pip install -r requirements.txt`.

Pandoc es un caso aparte: solo hace falta para formatos antiguos (`.doc`, `.odt`, `.rtf`, `.tex`, `.rst`, `.org`, `.typ`). `.docx`, `.html`, `.epub` y `.ipynb` se procesan de forma nativa.

## 1. Instalación (elige una)

**A — Clon de Git (recomendado):**

```bash
git clone https://github.com/hugohe3/ppt-master.git
cd ppt-master
pip install -r requirements.txt
```

**B — ZIP:** página de GitHub → **Code → Download ZIP** → descomprime → `pip install -r requirements.txt`. Sin historial de Git no hay actualizaciones con `git pull`. En Releases hay además un paquete reducido, solo con la skill (~56 MB, sin las presentaciones de ejemplo).

**C — Marketplace de skills:**

```bash
# CLI para varios agentes (Claude Code, Cursor, Codex, …)
npx skills add hugohe3/ppt-master

# O dentro de Claude Code
/plugin marketplace add hugohe3/ppt-master
/plugin install ppt-master@ppt-master
```

Por el marketplace solo bajan los archivos de la skill: igual tienes que ejecutar `pip install -r requirements.txt` en la carpeta instalada.

## 2. Ábrelo en un agente

PPT Master es un flujo de trabajo («skill») que corre dentro de cualquier herramienta de IA con capacidad de agente. Solo usas su panel de chat; no programas nada.

| Tipo | Ejemplos |
|---|---|
| Agente integrado en el IDE | VS Code, Cursor, Trae, Windsurf, Zed |
| Plugin / extensión de IDE | Claude Code (VS Code / JetBrains), GitHub Copilot, Cline |
| Agente de terminal | Claude Code CLI, Codex CLI, Gemini CLI |

La recomendación del autor: **Claude Code**. Apunta la herramienta a la carpeta del proyecto: **File → Open Folder** en un IDE, o `cd ppt-master` en un agente de terminal. Para mejores resultados, el proyecto recomienda un modelo de contexto amplio (Kimi K3 o Claude, ~1M de tokens) con generación de imágenes por IA.

## 3. Crear una presentación — el ciclo de 3 pasos

1. **Deja tu material** en `projects/` (PDF, DOCX, imágenes, Markdown, una URL) o pega el texto directamente en el chat.
2. **Pídele a la IA lo que quieres:**

```
Please create a PPT from projects/q3-report/sources/report.pdf
```

```
Please turn the following into a PPT: [pega tu contenido]
```

3. **Recoge el archivo** en `exports/<nombre>_<timestamp>.pptx` — DrawingML nativo, editable en PowerPoint, Keynote, WPS o LibreOffice.

Por defecto, la IA confirma antes una especificación de diseño (plantilla, formato — PPT 16:9 —, número de páginas). Para saltarte esa vuelta, dilo:

```
Quickly generate a 5-page deck from projects/q3-report/sources/report.pdf — no need to confirm with me
```

Gráficos y tablas salen como formas editables por defecto; con `--native-charts-and-tables` obtienes objetos Chart/Table de PowerPoint con datos reales y botón **Edit Data** (se guarda como un `_native_charts_tables.pptx` aparte).

## 4. Plantillas — dos rutas distintas

Por defecto manda el **diseño libre**; no necesitas plantilla. Úsala solo cuando haya que reutilizar un juego fijo de layouts o una identidad de marca.

| Quieres… | Ruta | Qué pasa |
|---|---|---|
| Esta misma presentación, con contenido nuevo | **Template fill** (relleno directo) | Elige las páginas que encajan (una página puede servir para varias diapositivas), y escribe texto, tablas y datos de gráficos directamente en el archivo original. Diseño, layouts, imágenes y animaciones se conservan. No crea páginas, no cambia layouts, no sustituye imágenes. |
| Una presentación nueva con el estilo de esta | **create-template** | Convierte el `.pptx` en un paquete reutilizable y genera una presentación nueva por la tubería SVG: estructura nueva, las páginas que quieras. |

**Template fill** — entrégale a la IA la presentación y tu material, y pídelo sin rodeos:

```
Fill this deck with the new content: [deck.pptx + tu material]
```

**create-template** — hay que activarlo de forma explícita; la IA no cambia sola de modo:

```
You: Replicate this as a template via /create-template: projects/brand/our_deck.pptx
```

Eso genera un paquete (SVG de layouts + `design_spec.md` + colores de tema, fuentes e imágenes). La plantilla vive en la biblioteca de la skill, en `skills/ppt-master/templates/layouts/<id>/` (regístrala con `register_template.py`), o dentro de un proyecto, en `projects/<proyecto>/templates/`. **Se invoca por la ruta de la carpeta** en el chat; el nombre solo nunca la activa:

```
You: Make a deck from sources/report.pdf with template skills/ppt-master/templates/layouts/academic_defense/
```

El relleno directo también tiene su ruta por comandos:

```bash
python3 scripts/template_fill_pptx.py analyze <proyecto>/sources/<origen.pptx> -o <proyecto>/analysis/slide_library.json
python3 scripts/template_fill_pptx.py scaffold <proyecto>/analysis/slide_library.json -o <proyecto>/analysis/fill_plan.json --slides "1,3,4"
python3 scripts/template_fill_pptx.py check-plan <proyecto>/analysis/slide_library.json <proyecto>/analysis/fill_plan.json -o <proyecto>/analysis/check_report.json
python3 scripts/template_fill_pptx.py apply <proyecto>/sources/<origen.pptx> <proyecto>/analysis/fill_plan.json -o <proyecto>/exports/filled.pptx
```

## 5. Imágenes (opcional)

Dos vías, combinables por imagen: **generación con IA** (la herramienta de imágenes del agente anfitrión, o `image_gen.py` con una clave de proveedor) y **búsqueda web** (`image_search.py`). La búsqueda funciona sin configurar nada (Openverse / Wikimedia Commons); las claves gratuitas `PEXELS_API_KEY` / `PIXABAY_API_KEY` en `.env` añaden bancos de fotos de más calidad. Las licencias se gestionan solas, con un crédito en la diapositiva cuando la licencia lo exige.

## 6. Actualizar

```bash
# Instalaciones por clon de Git
python3 skills/ppt-master/scripts/update_repo.py
```

Instalaciones por ZIP: descarga el ZIP nuevo, descomprímelo en una carpeta nueva, copia tu `.env` y tu carpeta `projects/`, y repite `pip install -r requirements.txt`.

## 7. Coste

Gratis y de código abierto (MIT). El proyecto lo dice claro: el único coste es el uso de tu propio modelo de IA; no hay ninguna suscripción de PPT encima. La tubería corre en tu máquina: salvo la comunicación con tu modelo de IA, tus datos no salen de ahí.

## 8. ¿Atascado?

- La IA pierde el contexto → pídele que lea `skills/ppt-master/SKILL.md`.
- Problemas de layout, exportación o modelo → el [FAQ](https://github.com/hugohe3/ppt-master/blob/main/docs/faq.md) del proyecto.

---

*Guía completa: [Getting Started](https://github.com/hugohe3/ppt-master/blob/main/docs/getting-started.md) · Repositorio: https://github.com/hugohe3/ppt-master*
