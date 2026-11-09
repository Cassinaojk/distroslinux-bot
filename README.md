# Blog Distros Linux — Robô de Notícias 3.0

Robô GitHub Actions para coletar notícias de fontes de tecnologia/Linux, traduzir e gerar matérias originais em português do Brasil com IA e publicar no Blogger.

**Blog:** [https://distroslinuxoficial.blogspot.com](https://distroslinuxoficial.blogspot.com)

## Principais mudanças da versão 3.0

- **Filtro de data rigoroso**: o robô só aceita notícias publicadas **hoje**. Qualquer notícia de ontem ou mais antiga é descartada automaticamente.
- **Imagens específicas por distro**: se o título da notícia mencionar uma distro (ex: Ubuntu, Fedora, Arch), o robô usa uma imagem específica daquela distro, alternando entre duas variantes (ex: `ubuntu.png` e `ubuntu 2.png`).
- **Fallback inteligente**: se a notícia não for de uma distro específica, o robô usa uma sequência cíclica de 12 imagens genéricas.
- **Anti-duplicidade via API do Blogger**: antes de chamar a IA, o robô verifica no seu blog se já existe um post com as mesmas palavras-chave da notícia. Se sim, pula para a próxima.

## Fontes ativas (6)

| Fonte | URL base | Feed principal |
|---|---|---|
| Linux.com | https://www.linux.com/ | `feed/` |
| Phoronix | https://www.phoronix.com/ | `rss.php` |
| LinuxToday | https://www.linuxtoday.com/ | `feed/` |
| Pplware — Linux | https://pplware.sapo.pt/category/linux/ | `feed/` |
| Linuxize | https://linuxize.com/ | `index.xml` |
| It's FOSS | https://itsfoss.com/ | `rss/` |

## IA (provedores em ordem de fallback)
1. **Gemini** — modelo principal `gemini-3.5-flash-lite`
2. **Gemini-2** — modelo fallback `gemini-3.6-flash`
3. **Groq** — modelo `openai/gpt-oss-20b`
4. **Mistral** — modelo `mistral-small-latest`

- Até **6 chamadas de texto por execução** (`MAX_GEMINI_TEXT_CALLS_PER_RUN=6`).
- Ao detectar quota/limite em um provedor, o robô passa automaticamente para o próximo.
- Se todos os provedores atingirem quota, a execução para e o restante fica para a próxima.

## Publicação
- Até **1 matéria por execução** (`MAX_POSTS_PER_RUN=1`).
- Deduplicação por URL da fonte e por URL do blog.
- A imagem de destaque é escolhida com base no título da notícia:
  - **Se mencionar uma distro** (Ubuntu, Fedora, Arch, Debian, Linux Mint, Manjaro, Pop!_OS, Red Hat, openSUSE, Kubuntu, Xubuntu, Zorin OS, elementary OS, Deepin, CentOS): usa a imagem específica da distro em `bot/imagens/distros/`, alternando entre `<distro>.png` e `<distro> 2.png`.
  - **Se não mencionar distro**: usa a sequência de 12 imagens genéricas em `bot/imagens/` (`Distribuições Linux (1).png` a `Distribuições Linux (12).png`).
- Vídeos incorporados (YouTube, Vimeo, etc.) são extraídos automaticamente da fonte.
- A fonte fica registrada em comentário HTML invisível (`BLOG_DISTROS_LINUX_SOURCE_URL`).
- O robô traduz automaticamente o conteúdo para **português do Brasil** antes de publicar.
- Apenas **1 imagem extra** é adicionada ao corpo do post (a `og:image` da fonte, após o 2º parágrafo).

## Filtro de Tecnologia
O robô usa uma lista de termos como `linux`, `ubuntu`, `fedora`, `debian`, `open source`, `kernel`, `segurança`, `programação`, etc., para garantir que apenas notícias de tecnologia sejam publicadas. Se houver dúvida, a regra é publicar.

## Filtro de data
- **`MAX_AGE_DAYS = 0`**: só aceita notícias publicadas **hoje**.
- Notícias antigas são descartadas antes mesmo de chamar a IA, economizando chamadas e evitando publicações fora do prazo.

## Anti-duplicidade
Além da originalidade em relação à fonte, o robô bloqueia publicações repetidas no blog por:

1. **API do Blogger (`check_blogger_for_duplicate`)**: busca no próprio blog se já existe post com as mesmas palavras-chave principais da notícia. Roda **antes** da IA, economizando chamadas e tempo.
2. **Similaridade de título**: depois que a IA gera o título, compara com os títulos já publicados por Jaccard de palavras (≥ 0.55).
3. **Fingerprint de conteúdo**: extrai entidades (nomes próprios, distros, versões) e vocabulário raro para comparação profunda.

Se qualquer camada detectar duplicata, a notícia é **ignorada** e o robô passa para a próxima da fila.

## Originalidade
A verificação bloqueia apenas sinais fortes de reprodução literal:
- sequência de **15 palavras ou mais** idênticas à fonte;
- sobreposição de **8-grams acima de 12%**.

## Secrets necessários
- `BLOGGER_BLOG_ID`
- `GOOGLE_CLIENT_ID`
- `GOOGLE_CLIENT_SECRET`
- `BLOGGER_REFRESH_TOKEN`
- `GEMINI_API_KEY`

### Secrets opcionais
- `GEMINI_API_KEY_2` (segunda chave Gemini)
- `GROQ_API_KEY`
- `MISTRAL_API_KEY`

## Variáveis de ambiente opcionais
- `VERBOSE_LOG=true` — ativa log detalhado
- `MAX_POSTS_PER_RUN` — padrão `1`
- `MAX_GEMINI_TEXT_CALLS_PER_RUN` — padrão `6`
- `GEMINI_MODEL` / `GEMINI_FALLBACK_MODEL`
- `GROQ_MODEL` / `MISTRAL_MODEL`
- `GITHUB_REPOSITORY` — owner/repo (default `Cassinaojk/distroslinuxoficial`)
- `BLOG_HOME_URL` — URL do blog (default `https://distroslinuxoficial.blogspot.com`)

## Estrutura de arquivos
