# CapCut MCP Server — Extended

Um servidor MCP (Model Context Protocol) estendido para automatizar
a edição de vídeo no CapCut através do Claude Code. Suporta tipografia
parametrizada, presets de animação e fluxos completos de produção talking-head.

## Baseado em

Este projeto é um fork e extensão do
[capcut-mcp-server](https://github.com/atx-guy/capcut-mcp-server)
de [@atx-guy](https://github.com/atx-guy), licenciado sob MIT.

Todas as ferramentas originais foram preservadas e estendidas com:
- Sistema de tipografia parametrizado (fonte, tamanho, posição, cor)
- Biblioteca de presets de animação
- Preset de produção talking head
- Preset de fluxo Reels/Shorts
- Sistema de estilos de legenda

## Ferramentas

| Ferramenta | Descrição |
|------|-------------|
| `capcut_create_draft` | Cria um novo projeto com dimensões e fps customizados |
| `capcut_add_video` | Adiciona clipe de vídeo com timing, transições e velocidade |
| `capcut_add_audio` | Adiciona áudio com volume e efeitos de fade |
| `capcut_add_text` | Adiciona texto estilizado com fontes, cores e animações |
| `capcut_add_subtitle` | Importa legendas SRT com estilo customizado |
| `capcut_add_effect` | Aplica efeitos visuais |
| `capcut_save_draft` | Salva o projeto na pasta de drafts do CapCut |
| `capcut_talking_head` | Preset completo de talking head (um único comando) |

## Requisitos

- Node.js 18+
- Claude Code
- CapCut (Windows ou Mac)

## Instalação
```bash
git clone https://github.com/TU_USUARIO/capcut-mcp-server-pro
cd capcut-mcp-server-pro
npm install
npm run build
claude mcp add capcut -- node /ruta/dist/index.js
```

## Uso
```bash
claude
> Crea un proyecto talking head con el video en C:/grabacion.mp4, 
  subtítulos en blanco, fuente Montserrat 72px, y guárdalo en CapCut
```

## Licença

MIT — veja [LICENSE](LICENSE)

# CapCut MCP Server

Um servidor Model Context Protocol (MCP) profissional para automação de edição de vídeo no **CapCut Pro**. Este servidor permite que assistentes de IA e aplicações criem e editem vídeos programaticamente usando os recursos de edição do CapCut.

## 🎬 Funcionalidades

- **Suíte Completa de Edição de Vídeo**: Cria drafts, adiciona vídeos, áudios, textos, imagens, efeitos e mais
- **Ferramentas Profissionais**: 11 ferramentas especializadas para fluxos de produção de vídeo
- **Type-Safe**: Construído com TypeScript para confiabilidade e excelente suporte de IDE
- **Transporte Flexível**: Suporta conexões tanto stdio (local) quanto HTTP (remota)
- **Validação de Entrada**: Schemas Zod completos com mensagens de erro úteis
- **Formatos de Saída Duplos**: JSON para máquinas, Markdown para humanos

## 📋 Pré-requisitos

Antes de usar este servidor MCP, é necessário ter o **VectCutAPI** (servidor de API do CapCut) em execução:

1. **Instalar o VectCutAPI**:
   ```bash
   git clone https://github.com/sun-guannan/VectCutAPI.git
   cd VectCutAPI
   pip install -r requirements.txt
   ```

2. **Iniciar o Servidor de API**:
   ```bash
   python capcut_server.py
   ```
   O servidor iniciará em `http://localhost:9001` por padrão.

## 🚀 Instalação

### Opção 1: Instalar via npm (quando publicado)
```bash
npm install -g capcut-mcp-server
```

### Opção 2: Compilar a partir do código-fonte
```bash
# Clonar este repositório
git clone <your-repo-url>
cd capcut-mcp-server

# Instalar dependências
npm install

# Compilar o projeto
npm run build

# Testar o servidor
npm start
```

## 🔧 Configuração

### Para o Claude Desktop

Adicione ao arquivo de configuração do Claude Desktop:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "capcut": {
      "command": "node",
      "args": ["/absolute/path/to/capcut-mcp-server/dist/index.js"],
      "env": {
        "CAPCUT_API_URL": "http://localhost:9001"
      }
    }
  }
}
```

### Para outros clientes MCP

O servidor suporta dois modos de transporte:

#### Modo Stdio (Padrão - Integração Local)
```bash
node dist/index.js
```

#### Modo HTTP (Acesso Remoto)
```bash
TRANSPORT=http PORT=3000 node dist/index.js
```

## 🛠️ Ferramentas Disponíveis

### 1. `capcut_create_draft`
Cria um novo projeto de edição de vídeo com dimensões e taxa de quadros customizadas.

**Exemplo**:
```typescript
{
  "width": 1920,
  "height": 1080,
  "fps": 30
}
```

### 2. `capcut_add_video`
Adiciona clipes de vídeo com transições, controle de velocidade e ajustes de volume.

**Exemplo**:
```typescript
{
  "draft_id": "abc123",
  "video_url": "https://example.com/video.mp4",
  "start": 0,
  "end": 10,
  "volume": 0.8,
  "transition": "fade_in",
  "speed": 1.0
}
```

### 3. `capcut_add_audio`
Adiciona música de fundo ou efeitos sonoros com fade in/out.

**Exemplo**:
```typescript
{
  "draft_id": "abc123",
  "audio_url": "https://example.com/music.mp3",
  "start": 0,
  "end": 30,
  "volume": 0.5,
  "fade_in": 2,
  "fade_out": 2
}
```

### 4. `capcut_add_text`
Adiciona sobreposições de texto estilizado com animações, sombras e fundos.

**Exemplo**:
```typescript
{
  "draft_id": "abc123",
  "text": "Welcome!",
  "start": 0,
  "end": 3,
  "font_size": 72,
  "font_color": "#FFFFFF",
  "background_color": "#000000",
  "shadow_enabled": true,
  "animation": "fade_in"
}
```

### 5. `capcut_add_image`
Adiciona sobreposições de imagem com posicionamento, escala e rotação.

**Exemplo**:
```typescript
{
  "draft_id": "abc123",
  "image_url": "https://example.com/logo.png",
  "start": 0,
  "end": 5,
  "position_x": 0.9,
  "position_y": 0.1,
  "scale": 0.3
}
```

### 6. `capcut_add_subtitle`
Importa legendas em formato SRT com estilo customizado.

**Exemplo**:
```typescript
{
  "draft_id": "abc123",
  "srt_content": "1\n00:00:01,000 --> 00:00:03,000\nWelcome to my video",
  "font_size": 36,
  "font_color": "#FFFFFF",
  "background_enabled": true
}
```

### 7. `capcut_add_keyframe`
Cria animações suaves usando interpolação por keyframe.

**Exemplo**:
```typescript
{
  "draft_id": "abc123",
  "track_name": "main",
  "property_types": ["scale_x", "scale_y"],
  "times": [0, 2, 4],
  "values": ["1.0", "1.5", "1.0"]
}
```

### 8. `capcut_add_effect`
Aplica efeitos visuais como blur, brilho, saturação, etc.

**Exemplo**:
```typescript
{
  "draft_id": "abc123",
  "effect_name": "blur",
  "start": 0,
  "end": 2,
  "intensity": 0.7
}
```

### 9. `capcut_add_sticker`
Adiciona stickers decorativos ou emojis.

**Exemplo**:
```typescript
{
  "draft_id": "abc123",
  "sticker_url": "https://example.com/emoji.png",
  "start": 1,
  "end": 5,
  "position_x": 0.9,
  "position_y": 0.1,
  "scale": 0.2
}
```

### 10. `capcut_save_draft`
Salva o draft para importar na aplicação do CapCut.

**Exemplo**:
```typescript
{
  "draft_id": "abc123"
}
```

### 11. `capcut_get_duration`
Obtém a duração e os metadados de arquivos de mídia.

**Exemplo**:
```typescript
{
  "url": "https://example.com/video.mp4"
}
```

## 📖 Exemplos de Uso

### Fluxo Completo de Criação de Vídeo

```typescript
// 1. Criar um novo draft
const draft = await capcut_create_draft({
  width: 1920,
  height: 1080,
  fps: 30
});

// 2. Adicionar vídeo de fundo
await capcut_add_video({
  draft_id: draft.draft_id,
  video_url: "https://example.com/background.mp4",
  start: 0,
  end: 10,
  volume: 0.6
});

// 3. Adicionar texto de título
await capcut_add_text({
  draft_id: draft.draft_id,
  text: "Amazing Video",
  start: 1,
  end: 4,
  font_size: 72,
  animation: "fade_in"
});

// 4. Adicionar música de fundo
await capcut_add_audio({
  draft_id: draft.draft_id,
  audio_url: "https://example.com/music.mp3",
  start: 0,
  end: 10,
  volume: 0.5
});

// 5. Adicionar animação de zoom
await capcut_add_keyframe({
  draft_id: draft.draft_id,
  track_name: "main",
  property_types: ["scale_x", "scale_y"],
  times: [0, 5, 10],
  values: ["1.0", "1.2", "1.0"]
});

// 6. Salvar o draft
const result = await capcut_save_draft({
  draft_id: draft.draft_id
});

console.log(`Draft saved to: ${result.draft_url}`);
```

## 🎯 Casos de Uso

- **Geração de Vídeo com IA**: Permite que assistentes de IA criem projetos de vídeo completos
- **Produção de Vídeo em Lote**: Automatiza a criação de múltiplos vídeos a partir de templates
- **Conteúdo para Redes Sociais**: Gera TikTok, Reels e YouTube Shorts automaticamente
- **Conteúdo Educacional**: Cria vídeos tutoriais com texto e áudio sincronizados
- **Automação de Marketing**: Gera vídeos promocionais em escala

## 🔍 Solução de Problemas

### Servidor Não Responde
- Confirme que o servidor VectCutAPI está em execução na porta 9001
- Verifique se a variável de ambiente `CAPCUT_API_URL` está correta
- Verifique a conectividade de rede com o servidor de API

### Erros de Build
```bash
# Limpar e recompilar
rm -rf dist node_modules package-lock.json
npm install
npm run build
```

### Arquivos de Mídia Não Encontrados
- Confirme que todas as URLs de mídia estão acessíveis
- Use links diretos para os arquivos (evite redirecionamentos)
- Verifique se o formato do arquivo é suportado

## 🤝 Contribuindo

Contribuições são bem-vindas! Sinta-se à vontade para enviar um Pull Request.

## 📄 Licença

Licença MIT - sinta-se à vontade para usar isto em seus projetos!

## 🙏 Agradecimentos

- Construído sobre o [VectCutAPI](https://github.com/sun-guannan/VectCutAPI) por sun-guannan
- Usa a especificação do [Model Context Protocol](https://modelcontextprotocol.io/)
- Desenvolvido com o [Anthropic's MCP SDK](https://github.com/modelcontextprotocol/typescript-sdk)

## 📞 Suporte

Para problemas relacionados a:
- **Este Servidor MCP**: Abra uma issue neste repositório
- **VectCutAPI**: Visite https://github.com/sun-guannan/VectCutAPI
- **Aplicação CapCut**: Contate o suporte do CapCut

---

**Nota**: Este é um servidor MCP não oficial para o CapCut Pro. Requer o backend VectCutAPI para funcionar. CapCut é uma marca registrada da Bytedance Ltd.
