# CLAUDE.md

Este arquivo fornece orientação ao Claude Code (claude.ai/code) ao trabalhar com código neste repositório.

## Visão Geral do Projeto

**capcut-mcp-server-extended** é um servidor Model Context Protocol (MCP) que conecta assistentes de IA
(Claude, etc.) à edição de vídeo do CapCut Pro através de um backend VectCutAPI. É um fork do
`atx-guy/capcut-mcp-server` estendido em direção a uma ferramenta profissional de automação de vídeo, com foco em
**talking head / conteúdo para reels**: vídeos verticais 1080×1920, legendas automáticas, texto animado e
presets composáveis que condensam fluxos de vários passos em uma única chamada de ferramenta.

O objetivo final é permitir que um assistente de IA produza um vídeo curto totalmente editado emitindo
algumas chamadas de ferramenta MCP — sem interação manual com o CapCut.

---

## Comandos

```bash
# Build (compila TypeScript → dist/, torna dist/index.js executável)
npm run build

# Desenvolvimento (modo watch, recompila a cada alteração)
npm run dev

# Executa o servidor compilado
npm start
```

Nenhum script de teste está configurado. O servidor inicia e se comunica via stdio (padrão) ou HTTP.
Sempre execute `npm run build` após cada alteração de código e confirme zero erros antes de commitar.

---

## Arquitetura

**Modos de transporte** (definidos pela variável de ambiente `TRANSPORT`):
- `stdio` (padrão) — para Claude Desktop / clientes MCP locais
- `http` — escuta em `PORT` (padrão 3000)

**Variáveis de ambiente principais**: `CAPCUT_API_URL` (padrão `http://localhost:9001`), `PORT`, `TRANSPORT`

**Backend VectCutAPI**: roda em `http://localhost:9001`. Endpoints úteis:
- `GET /get_font_types` — retorna a lista completa de nomes de fontes suportadas
- `POST /create_draft`, `/add_video`, `/add_text`, `/add_keyframe`, `/save_draft`, etc.

### Fluxo de dados

```
MCP Client
  → src/tools/index.ts          (registro de ferramentas + validação Zod)
  → src/tools/presets.ts        (ferramentas compostas de alto nível — Fase 3+)
  → src/services/api-client.ts  (singleton axios, timeout de 60 s)
  → Backend VectCutAPI          (http://localhost:9001)
```

### Layout do código-fonte

| Arquivo / Pasta | Função |
|---|---|
| `src/index.ts` | Ponto de entrada; configuração de transporte stdio/HTTP — **não modificar** |
| `src/tools/index.ts` | Todas as 13 definições de ferramentas MCP; helpers `formatResponse` / `handleError` |
| `src/tools/presets.ts` | Ferramentas compostas de alto nível (Fase 3+) |
| `src/schemas/index.ts` | Schemas Zod para toda entrada de ferramenta base |
| `src/services/api-client.ts` | `apiClient` singleton; mapeia chamadas de ferramenta → endpoints POST — **não modificar** |
| `src/types.ts` | Interfaces TypeScript (`DraftConfig`, `VideoTrack`, `ResponseFormat`, …) |
| `src/constants.ts` | `API_BASE_URL`, valores padrão, formatos suportados, efeitos, transições |
| `src/presets/typography.ts` | **[Fase 1 ✅]** Três estilos de texto nomeados; fonte = `Poppins_Bold` |
| `src/presets/animations.ts` | **[Fase 2 ✅]** Sequências de animação por keyframe (`popInUpper`) |
| `src/utils/validators.ts` | **[Fase 5]** Validação de caminhos, helpers Windows↔Unix |
| `utils_py/transcribe_audio.py` | Transcrição Whisper com timing por palavra; gera lista JSON de palavras |
| `utils_py/inspect_draft.py` | Lê `draft_content.json`; extrai caminho do áudio, duração e fps |
| `utils_py/validate_project.py` | Verifica se todos os arquivos de mídia referenciados num draft existem |

### 13 ferramentas MCP

```
capcut_create_draft
  → capcut_add_video / capcut_add_audio / capcut_add_text
  → capcut_add_image / capcut_add_subtitle / capcut_add_keyframe
  → capcut_add_effect / capcut_add_sticker
  → capcut_save_draft

capcut_get_duration          (somente leitura — consulta metadados de mídia)
capcut_add_animated_text     (add_text + animação por keyframe numa única chamada)
capcut_edit_draft_words      (pipeline completo: cria draft → adiciona vídeo → adiciona palavras → salva)
```

Todas as ferramentas aceitam `response_format: 'markdown' | 'json'`.
Markdown usa `formatResponse()` para saída legível por humanos; JSON retorna `structuredContent`.

---

## Convenções de Código

- **Modo estrito do TypeScript** — `strict: true`, `noUnusedLocals`, `noUnusedParameters`,
  `noImplicitReturns` estão todos habilitados em `tsconfig.json`. Evitar `any`; usar generics tipados ou
  `unknown` + type guards quando o formato for realmente dinâmico.
- **Módulos ESM** — `"type": "module"` no package.json. Todos os imports locais devem incluir a extensão
  `.js` (mesmo para arquivos de origem `.ts`). Exemplo: `import { foo } from './bar.js'`.
- **Nomenclatura de ferramentas** — todas as ferramentas MCP usam `snake_case` com o prefixo `capcut_`.
- **Schema primeiro** — toda entrada de ferramenta deve ter um schema Zod correspondente exportado de
  `src/schemas/index.ts` (ferramentas base) ou colocado junto do arquivo de preset (ferramentas de preset).
- **Exports de preset** — cada arquivo de preset exporta:
  1. Um objeto `const` com os valores do preset (ex.: `TYPOGRAPHY_STYLES`).
  2. Um schema Zod derivado desses valores (ex.: `TypographyStyleNameSchema`).
  3. O tipo TypeScript inferido (ex.: `type TypographyStyleName`).
- **Comentários em inglês** — todos os comentários inline, JSDoc e mensagens de commit em inglês.
- **Nenhuma modificação de arquivos estáveis** — `src/index.ts` e `src/services/api-client.ts` são
  estáveis; não tocar neles a menos que haja uma alteração de backend que quebre a compatibilidade.
- **Gate de build** — `npm run build` deve passar com zero erros após cada alteração.

### Adicionando uma nova ferramenta base

1. Adicionar tipos em `src/types.ts` se necessário.
2. Adicionar um schema Zod + tipo inferido em `src/schemas/index.ts`.
3. Adicionar o método de API em `src/services/api-client.ts`.
4. Registrar a ferramenta dentro de `registerTools()` em `src/tools/index.ts`.
5. Executar `npm run build`.

### Adicionando uma nova ferramenta de preset/composta

1. Definir os dados do preset no arquivo apropriado em `src/presets/*.ts`.
2. Adicionar também seu schema Zod e tipo TypeScript lá.
3. Registrar a ferramenta em `src/tools/presets.ts` (criar o arquivo se necessário).
4. Importar e chamar `registerPresetTools(server)` a partir de `src/index.ts`, se ainda não estiver feito.
5. Executar `npm run build`.

---

## Sistema de tipografia e animações

Quando o usuário pedir para adicionar texto a um clipe, SEMPRE usar
`capcut_add_animated_text` em vez de `capcut_add_text`.

### Estilos disponíveis (`typography_style`)

Definidos em `src/presets/typography.ts`. Os três usam `Poppins_Bold` como fonte.

| Nome | Cor | Stroke | Shadow | Quando usar |
|---|---|---|---|---|
| `defaultTypeWhite` | `#ecebeb` | preto, thickness=40 | sim | Uso geral — fundo escuro |
| `defaultTypeBlack` | `#000000` | não | não | Fundos claros |
| `defaultTypeRed` | `#aa1a1a` | não | não | Ênfase, alertas, labels |

Se o usuário não especificar um estilo, usar `defaultTypeWhite`.

### Fontes suportadas pela API

A lista completa é obtida com `GET http://localhost:9001/get_font_types`.
Fontes recomendadas para conteúdo em espanhol (latinas, bem legíveis em reels):

| Uso | Fonte |
|---|---|
| Título / palavra bold | `Poppins_Bold`, `Sora_Bold`, `Inter_Black`, `Kanit_Black` |
| Subtítulo / body | `Poppins_Regular`, `Sora_Regular`, `Nunito` |
| Display / impacto | `Thunder`, `Staatliches_Regular`, `Bungee_Regular` |

`Montserrat-Bold` **não é suportada** pela API — usar `Poppins_Bold` como equivalente.

### Animações disponíveis (`animation_in`)

Definidas em `src/presets/animations.ts`.

| Nome | Descrição | Duração |
|---|---|---|
| `popInUpper` | Cai de um pouco acima (offset +0.05) com fade in | 13 frames (≈433 ms a 30 fps) |

**Direção**: no espaço de keyframes do CapCut, o eixo Y positivo aponta para CIMA na
tela. O offset `+0.05` faz o elemento começar 0.05 unidades mais acima e "cair" até
sua posição final — efeito de entrada descendente suave.

Se o usuário não especificar animação, aplicar `popInUpper` por padrão para o texto principal.

### Parâmetros padrão para talking head

| Parâmetro | Valor | Motivo |
|---|---|---|
| `position_x` | `0.5` | Centralizado horizontalmente |
| `position_y` | `0.85` | Texto animado principal (próximo ao topo); legendas → usar `0.10` |
| `start` / `end` | conforme o timing do clipe indicado | — |

**Convenção do eixo Y**: `0 = fundo da tela`, `1 = topo da tela` (positivo = para cima).

---

## Utilitários Python (`utils_py/`)

Scripts auxiliares chamados a partir do Claude Code com o Python do sistema
(`/c/Users/Migue/AppData/Local/Programs/Python/Python311/python`).
Sempre executar com `PYTHONUTF8=1` para evitar erros de encoding no Windows.

### `transcribe_audio.py`

Transcreve um arquivo de áudio/vídeo com Whisper e retorna uma lista JSON de palavras com
timestamps. Aplica correção automática de sobreposições: se `word[i].start < word[i-1].end`,
ajusta `word[i].start = word[i-1].end + 0.01`.

```bash
python utils_py/transcribe_audio.py "path/to/video.mov" --lang es --model base
# Saída: [{ "word": "...", "start": 0.44, "end": 0.88 }, ...]
```

Modelos disponíveis: `tiny` | `base` | `small` | `medium` | `large`

### `inspect_draft.py`

Lê um `draft_content.json` do CapCut e extrai `audio_path`, `duration_sec` e `fps`.

```bash
python utils_py/inspect_draft.py "path/to/draft_content.json"
```

### `validate_project.py`

Verifica se todos os arquivos de mídia referenciados no draft existem em disco.

```bash
python utils_py/validate_project.py "path/to/draft_content.json"
# Saída: { "valid": true/false, "missing": [...] }
```

### `group_words.py`

Agrupa uma lista de palavras com timestamps em frases/legendas completas. Quebra a frase
quando o máximo de caracteres é excedido, há uma pausa longa, ou a palavra termina com `.?!…`.

```bash
python utils_py/group_words.py words.json --max-chars 35 --max-gap 0.5
# Entrada:  [{"word":"Hola","start":0.5,"end":0.8}, {"word":"mundo","start":0.8,"end":1.2}]
# Saída: [{"text":"Hola mundo","start":0.5,"end":1.2}]
```

Parâmetros:
- `--max-chars` (padrão: 35) — máximo de caracteres por frase
- `--max-gap` (padrão: 0.5s) — pausa máxima para manter palavras na mesma frase

### `calc_subtitle_y.py`

Agrupa palavras em frases e atribui um `position_y` específico a cada frase de acordo com o
número estimado de linhas visuais. Usa internamente o `group_words`.

**Convenção dos eixos**: `0 = fundo da tela`, `1 = topo`. Valores baixos = mais para baixo.

```bash
python utils_py/calc_subtitle_y.py words.json --base_y 0.10
# Saída: [{"text":"Compramos la Mazda","start":0.58,"end":2.3,"position_y":0.16}, ...]
```

Parâmetros:
- `--base_y` (padrão: 0.10) — Y do centro de uma frase de 1 linha
- `--line_height` (padrão: 0.06) — unidades Y por linha adicional
- `--chars_per_line` (padrão: 20) — caracteres estimados por linha visual (~3 palavras)
- `--max-chars` (padrão: 35) — máx. caracteres por frase
- `--max-gap` (padrão: 0.5s) — pausa máxima para manter palavras juntas

**Fórmula**: `position_y = base_y + (n_lines − 1) × line_height`
→ frases de 2 linhas sobem o centro para que a borda inferior fique em `base_y`.

Este módulo é **importado automaticamente** por `edit_draft_pipeline.py` quando se
usa o modo `--no-word-by-word --no-buildup` (modo frases). Não é necessário chamá-lo diretamente.

### `edit_draft_pipeline.py` ⭐ script principal

Pipeline unificado com chamadas de API **paralelas** (ThreadPoolExecutor, 8 workers).
Faz tudo numa única execução: prepara entradas → cria draft temporário →
adiciona elementos em paralelo → mescla no draft existente.

**Modos disponíveis**:
- `--word-by-word` (padrão) — uma palavra por elemento, centralizada, `position_y` fixo
- `--no-word-by-word --no-buildup` — uma frase completa por elemento, `position_y` ajustado por linhas
- `--no-word-by-word --buildup` — layout horizontal acumulativo por frase (legado)

```bash
python utils_py/edit_draft_pipeline.py \
  --draft "C:/path/to/draft_content.json" \
  --words words.json \
  --style defaultTypeWhite --animation popInUpper \
  --position_y 0.10
# Saída: {"entries_added":25,"source_words":25,"text_tracks_merged":25,"mode":"word_by_word",...}
```

Arquivos temporários: usar sempre `C:/smart_cut/tmp/` como diretório intermediário.

**`VECTCUT_DRAFT_DIR`**: o backend (VectCutAPI) salva os drafts temporários no seu próprio
diretório de trabalho. O padrão é `C:/capcut_project/capcut-mcp-back` (pasta do backend).
Se o backend for executado de outro caminho, passar `VECTCUT_DRAFT_DIR=<caminho>` como variável de ambiente.

**Desempenho**: para 25 palavras, faz ~75 chamadas de API em paralelo.
Tempo estimado: 3-6s.

### `add_words_to_draft.py`

Adiciona legendas de palavras ou frases diretamente a um `draft_content.json` EXISTENTE, sem
criar um novo projeto. Usa o VectCutAPI para gerar os elementos de texto e depois faz um
merge do JSON resultante no draft original.

**Vantagem**: preserva todas as faixas existentes (vídeo, B-roll, áudio, efeitos).

```bash
python utils_py/add_words_to_draft.py \
  --draft  "C:/path/to/draft_content.json" \
  --words  '[{"word":"Hola","start":0.5,"end":1.0}, ...]' \
  --style  defaultTypeWhite \
  --animation popInUpper \
  --position_x 0.5 \
  --position_y 0.85
# Saída: {"temp_draft_id":"...","entries_added":25,"text_tracks_merged":25,...}
```

Também aceita frases (chave `"text"` em vez de `"word"`):
```bash
python utils_py/group_words.py words.json | python utils_py/add_words_to_draft.py \
  --draft "C:/path/to/draft_content.json" --words -
```
*(passar `-` como `--words` não está implementado; salvar em arquivo intermediário primeiro)*

Cria backup automático em `draft_content.json.bak_words` (apenas se não existir).

---

## Fluxo típico: legendas sobre projeto EXISTENTE

Para adicionar legendas a um projeto CapCut já existente (preservando B-roll, efeitos, etc.):

```bash
# 1. Transcrever áudio
python utils_py/transcribe_audio.py "video.mov" --lang es --model medium

# 2. Agrupar palavras em frases (opcional, recomendado)
python utils_py/group_words.py words.json --max-chars 35 > phrases.json

# 3. Adicionar ao draft existente
python utils_py/add_words_to_draft.py \
  --draft "C:/Users/.../draft_content.json" \
  --words phrases.json \
  --style defaultTypeWhite \
  --animation popInUpper
```

## Fluxo alternativo: draft completo novo

A ferramenta `capcut_edit_draft_words` executa o pipeline completo numa única chamada
(cria um projeto NOVO, útil quando não existe draft anterior):

1. `POST /create_draft` — cria draft 1080×1920 no fps indicado
2. `POST /add_video` — adiciona o vídeo principal (duração completa)
3. `POST /add_text` × N + `POST /add_keyframe` × N — uma entrada por chamada com animação
4. `POST /save_draft` — salva; depois `publishDraftToCapcut()` copia para a pasta do CapCut

**Nota**: timestamps sobrepostos na transcrição do Whisper causam o erro `New segment overlaps`.
O script já os corrige automaticamente, mas se a lista for construída manualmente, garantir
que `word[i].start >= word[i-1].end`.

---

## Roadmap

### FASE 1 — Sistema de tipografia parametrizado ✅
`src/presets/typography.ts` — três estilos base (`defaultTypeWhite`, `defaultTypeBlack`,
`defaultTypeRed`), todos com `Poppins_Bold`, font_size 15, configuráveis em cor/stroke/shadow.

### FASE 2 — Biblioteca de animações ✅
`src/presets/animations.ts` — `popInUpper`: cai de um pouco acima (offset +0.05) com
fade in. Direção corrigida: no espaço de keyframe do CapCut, y positivo = mais alto na tela.
`resolveKeyframes()` converte a definição em chamadas para `apiClient.addKeyframe`.

### FASE 3 — Ferramenta de pipeline palavra por palavra ✅
`capcut_edit_draft_words` em `src/tools/index.ts` — pipeline completo: cria draft, adiciona
vídeo, adiciona cada entrada como texto animado, salva e publica no CapCut via `publishDraftToCapcut()`.
`utils_py/edit_draft_pipeline.py` — modo padrão `--word-by-word`: uma palavra por elemento,
centralizada, sem agrupamento. Modos alternativos: `--no-word-by-word --no-buildup` (frases),
`--no-word-by-word --buildup` (layout acumulativo legado).
`utils_py/add_words_to_draft.py` — adiciona legendas a um projeto existente (merge direto).

### FASE 4 — Ferramenta de legendas aprimorada
Estender `capcut_add_subtitle` para suportar:
- Estilos predefinidos: `"reels"`, `"youtube"`, `"minimal"`, `"bold"`
- `word_highlight` — destacar palavras-chave em outra cor
- `auto_position` — `"top"` | `"center"` | `"bottom"`

### FASE 5 — Validação e utilitários
`src/utils/validators.ts`:
- Validar que `video_path` exista antes de enviar à API
- Validar que `draft_folder` seja um caminho válido do CapCut
- Helper para converter caminhos Windows ↔ Unix

---

## Não tocar (por agora)

- `src/index.ts` — ponto de entrada estável
- `src/services/api-client.ts` — cliente de API funcional
- As 13 ferramentas existentes — estender apenas, nunca modificar o comportamento existente
