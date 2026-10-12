# 🎞️ PPT Master — Guia rápido (Português)

> Comandos verificados no README e no guia Getting Started de [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master) (2026-10-12).
> O que é: você entrega um PDF, um DOCX, uma página da web ou um texto colado, e um agente de IA transforma isso numa **apresentação PowerPoint nativa**: formas, caixas de texto, gráficos e tabelas de verdade, que dá para clicar e continuar editando. Nada de slide virado imagem.

---

## 0. Requisito

Só **Python 3.10+**. O resto vem com `pip install -r requirements.txt`.

O Pandoc é um caso à parte: só é preciso para formatos antigos (`.doc`, `.odt`, `.rtf`, `.tex`, `.rst`, `.org`, `.typ`). `.docx`, `.html`, `.epub` e `.ipynb` são tratados nativamente.

## 1. Instalação (escolha uma)

**A — Clone do Git (recomendado):**

```bash
git clone https://github.com/hugohe3/ppt-master.git
cd ppt-master
pip install -r requirements.txt
```

**B — ZIP:** página do GitHub → **Code → Download ZIP** → descompacte → `pip install -r requirements.txt`. Sem histórico do Git, não há atualização via `git pull`. Na página de Releases existe ainda um pacote menor, só com a skill (~56 MB, sem as apresentações de exemplo).

**C — Marketplace de skills:**

```bash
# CLI multiagente (Claude Code, Cursor, Codex, …)
npx skills add hugohe3/ppt-master

# Ou dentro do Claude Code
/plugin marketplace add hugohe3/ppt-master
/plugin install ppt-master@ppt-master
```

Pelo marketplace descem só os ficheiros da skill: continua a ser preciso correr `pip install -r requirements.txt` na pasta instalada.

## 2. Abra num agente

O PPT Master é um fluxo de trabalho («skill») que corre dentro de qualquer ferramenta de IA com capacidade de agente. Você só usa o painel de chat; não programa nada.

| Tipo | Exemplos |
|---|---|
| Agente integrado no IDE | VS Code, Cursor, Trae, Windsurf, Zed |
| Plugin / extensão de IDE | Claude Code (VS Code / JetBrains), GitHub Copilot, Cline |
| Agente de terminal | Claude Code CLI, Codex CLI, Gemini CLI |

A escolha do autor: **Claude Code**. Aponte a ferramenta para a pasta do projeto: **File → Open Folder** num IDE, ou `cd ppt-master` num agente de terminal. Para o melhor resultado, o projeto recomenda um modelo de contexto grande (Kimi K3 ou Claude, ~1M de tokens) com geração de imagens por IA.

## 3. Criar uma apresentação — o ciclo de 3 passos

1. **Largue o material** em `projects/` (PDF, DOCX, imagens, Markdown, um URL) ou cole o texto direto no chat.
2. **Peça à IA o que quer:**

```
Please create a PPT from projects/q3-report/sources/report.pdf
```

```
Please turn the following into a PPT: [cole o seu conteúdo]
```

3. **Pegue o arquivo** em `exports/<nome>_<timestamp>.pptx` — DrawingML nativo, editável no PowerPoint, Keynote, WPS ou LibreOffice.

Por padrão, a IA confirma antes uma especificação de design (template, formato — PPT 16:9 —, número de páginas). Para pular essa etapa, diga:

```
Quickly generate a 5-page deck from projects/q3-report/sources/report.pdf — no need to confirm with me
```

Gráficos e tabelas saem como formas editáveis por padrão; com `--native-charts-and-tables` você recebe objetos Chart/Table do PowerPoint com dados de verdade e o botão **Edit Data** (guardado num `_native_charts_tables.pptx` separado).

## 4. Templates — dois caminhos diferentes

O padrão é **design livre**; template não é obrigatório. Use um só quando for preciso reaproveitar um conjunto fixo de layouts ou uma identidade de marca.

| Você quer… | Caminho | O que acontece |
|---|---|---|
| Esta apresentação exata, com conteúdo novo | **Template fill** (preenchimento direto) | Escolhe as páginas que servem (uma página pode render vários slides), e escreve texto, tabelas e dados de gráficos direto no arquivo original. Design, layouts, imagens e animações ficam. Não cria páginas, não muda layouts, não troca imagens. |
| Uma apresentação nova no estilo desta | **create-template** | Converte o `.pptx` num pacote reutilizável e gera uma apresentação nova pelo pipeline SVG: estrutura nova, quantas páginas quiser. |

**Template fill** — entregue à IA a apresentação e o seu material, e peça sem rodeios:

```
Fill this deck with the new content: [deck.pptx + o seu material]
```

**create-template** — precisa de ser acionado de forma explícita; a IA não muda de modo sozinha:

```
You: Replicate this as a template via /create-template: projects/brand/our_deck.pptx
```

Isso gera um pacote (SVGs de layout + `design_spec.md` + cores do tema, fontes e imagens). O template mora na biblioteca da skill, em `skills/ppt-master/templates/layouts/<id>/` (registe-o com `register_template.py`), ou dentro de um projeto, em `projects/<projeto>/templates/`. **Invoca-se pelo caminho da pasta** no chat; só o nome nunca o aciona:

```
You: Make a deck from sources/report.pdf with template skills/ppt-master/templates/layouts/academic_defense/
```

O preenchimento direto também tem o seu caminho por comandos:

```bash
python3 scripts/template_fill_pptx.py analyze <projeto>/sources/<origem.pptx> -o <projeto>/analysis/slide_library.json
python3 scripts/template_fill_pptx.py scaffold <projeto>/analysis/slide_library.json -o <projeto>/analysis/fill_plan.json --slides "1,3,4"
python3 scripts/template_fill_pptx.py check-plan <projeto>/analysis/slide_library.json <projeto>/analysis/fill_plan.json -o <projeto>/analysis/check_report.json
python3 scripts/template_fill_pptx.py apply <projeto>/sources/<origem.pptx> <projeto>/analysis/fill_plan.json -o <projeto>/exports/filled.pptx
```

## 5. Imagens (opcional)

Dois caminhos, combináveis por imagem: **geração com IA** (a ferramenta de imagens do agente anfitrião, ou `image_gen.py` com uma chave de fornecedor) e **busca na web** (`image_search.py`). A busca funciona sem configurar nada (Openverse / Wikimedia Commons); as chaves gratuitas `PEXELS_API_KEY` / `PIXABAY_API_KEY` no `.env` somam bancos de fotos de melhor qualidade. As licenças são tratadas automaticamente, com um crédito no slide quando a licença o exige.

## 6. Atualizar

```bash
# Instalações por clone do Git
python3 skills/ppt-master/scripts/update_repo.py
```

Instalações por ZIP: baixe o ZIP novo, descompacte numa pasta nova, copie o seu `.env` e a pasta `projects/`, e repita `pip install -r requirements.txt`.

## 7. Custo

Grátis e de código aberto (MIT). O projeto é direto: o único custo é o uso do seu próprio modelo de IA; não há nenhuma assinatura de PPT por cima. O pipeline corre na sua máquina: tirando a conversa com o seu modelo de IA, os seus dados não saem dela.

## 8. Travou?

- A IA perdeu o contexto → peça para ela ler `skills/ppt-master/SKILL.md`.
- Problemas de layout, exportação ou modelo → o [FAQ](https://github.com/hugohe3/ppt-master/blob/main/docs/faq.md) do projeto.

---

*Guia completo: [Getting Started](https://github.com/hugohe3/ppt-master/blob/main/docs/getting-started.md) · Repositório: https://github.com/hugohe3/ppt-master*
