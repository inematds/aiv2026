# AIV 2026 — AI Visibility como serviço (AEO/GEO)

Material completo do serviço de **AI Visibility**: tornar uma empresa encontrável, citável e recomendável por IAs públicas (ChatGPT, Claude, Gemini, Perplexity) e instalar um agente de chat comercial no site dela. Este repo tem o **método replicável** (playbook), a **prova** (case do cliente-zero, INEMA) e este **guia para começar do zero**.

**Guia visual:** https://inematds.github.io/aiv2026/guia/

## Estrutura

```
aiv2026/
├── README.md                # este guia — do zero à operação contínua
├── playbook-aeo-geo.md      # O MÉTODO — 5 fases (F0–F4) abstraídas para qualquer cliente:
│                            # passos, critérios de saída, orçamentos, armadilhas reais,
│                            # o que adaptar por cliente, resumo comercial.
├── guia/index.html          # versão visual deste guia (GitHub Pages)
└── case-inema/              # O EXEMPLO — execução completa no cliente-zero:
    ├── prompt-war-game-portugues.md              # o prompt original que gerou o plano
    │                                             # (reutilizável: troque os [placeholders])
    ├── plano-aiv.md                              # plano war-game original (tabelas de
    │                                             # modos de falha, suposições, gates)
    ├── o-que-foi-feito-aiv.md                    # o que existe construído + runbook
    │                                             # operacional + pendências
    └── relatorio-processo-completo-2026-07-10.md # narrativa: decisões, riscos,
                                                  # verificações, o que fez dar certo
```

## O fluxo em uma linha

```
F0 Diagnóstico ──► F1 Catálogo ──► F2 Publicação ──► F3 Agente ──► F4 Loop
  (a régua)        (o conteúdo)    (a exposição)     (a conversão)  (a prova)
```

## As 5 estratégias que sustentam tudo

Estas regras valem em todas as fases — foram elas que fizeram o cliente-zero dar certo:

1. **Nenhuma fase fecha "no olho".** Todo critério de saída é um comando executado com resultado real (curl com user-agent de bot, query no banco, flood de rate limit) — nunca uma opinião.
2. **Nunca inventar dado do cliente.** Toda afirmação vem de fonte real (site, código, documento, conversa com o dono). Sem confirmação → marca `PENDENTE`, e o conteúdo público filtra os PENDENTE.
3. **Medir antes de mexer.** Sem o baseline da F0, nenhum resultado posterior é demonstrável — e a demonstração é o produto.
4. **Testar local antes de publicar** toda mudança de infraestrutura. No cliente-zero isso pegou 2 bugs que só apareceriam em produção (link errado no portal; rewrite vazando o domínio da hospedagem).
5. **Nada de agente em produção sem aprovação explícita do cliente** (10 transcrições revisadas antes do go-live; depois, toda mudança de prompt passa por golden set + aprovação).

---

## Comece do zero (passo a passo)

### Passo 0 — Prepare o terreno

```bash
git clone https://github.com/inematds/aiv2026
cat aiv2026/playbook-aeo-geo.md     # leia ANTES de executar qualquer coisa
```

O que você precisa ter: acesso ao domínio/site do cliente (para proxy e robots.txt), 1–2 h/semana do cliente para revisão durante o catálogo, um decisor para aprovar o agente, hospedagem estática gratuita (GitHub Pages), um Postgres serverless (Supabase) e uma chave de LLM que **nunca** toca o navegador.

### Passo 1 — Gere o seu plano war-game

Abra `case-inema/prompt-war-game-portugues.md`, troque os `[placeholders]` pelo projeto do cliente novo e rode numa IA capaz. O resultado é um plano no formato do `case-inema/plano-aiv.md` — fases com suposições otimista/pessimista, modos de falha com correção e critérios de saída verificáveis. **Planeje as barreiras antes de escrever uma linha de código.**

### Passo 2 — F0: meça o baseline (2 dias)

1. Levante com o cliente ~20 perguntas **neutras** que o público-alvo faria a uma IA (sem citar a marca — prompt enviesado invalida a medição).
2. **Congele a bateria como v1** — nunca altere depois; adições viram v2 medida em paralelo, senão a comparação mensal quebra.
3. Rode cada prompt **3×** em cada IA disponível (mede-se distribuição, não foto).
4. Registre em CSV: data, ferramenta, prompt, execução, cliente citado? (sim/não/parcial), concorrentes citados, **fontes citadas**, erros factuais.
5. Escreva o baseline: % de aparição por ferramenta, top concorrentes, top fontes, erros. As fontes que as IAs citam **no lugar** do cliente são o mapa de onde a autoridade dele precisa existir.

*No cliente-zero: 0% de aparição em 52 medições + colisão de nome com órgão público homônimo.*

### Passo 3 — F1: escreva o catálogo (~50 fichas, 1–2 semanas)

Fichas Markdown + front matter YAML numa pasta versionada (`catalogo/`), por tipo: institucional, produtos/serviços, FAQ (derivada da bateria F0 — perguntas com demanda comprovada), portfólio, cases. Campos obrigatórios:

```yaml
slug, tipo, titulo, resumo, status, fonte, confianca, atualizado_em, relacionados
# regra dura: sem confirmação → confianca: PENDENTE (o validador rejeita publicar PENDENTE)
```

Extração em **lotes de 10** com revisão do cliente por lote (a revisão é o caminho crítico — nunca espere as 50; a F2 pode partir com 20 revisadas). Pergunte cedo: "o que você vende de verdade? o que é gratuito?" — o modelo de negócio que o cliente descreve em conversa quase nunca está documentado, e no cliente-zero mudou 3× durante a escrita.

### Passo 4 — F2: publique para as IAs (1 semana + janela de indexação)

1. **Gerador estático próprio** (catálogo → HTML, sem dependências): resposta direta no 1º parágrafo, números/datas/listas, JSON-LD por tipo (`Organization`, `Course`, `FAQPage`, `Person`, `Article`), `canonical` no domínio do cliente, data de atualização visível, `sitemap.xml`, `robots.txt` e `api/*.json` (a "API" estática que o agente consome).
2. **Hospedagem**: GitHub Pages num repo público dedicado (o repo do programa continua privado).
3. **Proxy no domínio do cliente** (`dominio.com/conhecimento/*` → Pages) — a autoridade acumula no domínio dele. ⚠️ Armadilha real: rewrites nativos de framework podem vazar o domínio da hospedagem no redirect de barra final — teste com build local + curl comparando headers **antes** de publicar.
4. **robots.txt** liberando explicitamente os bots de IA: `OAI-SearchBot`, `ClaudeBot`, `PerplexityBot`, `Googlebot`, `Bingbot` — e apontando o `Sitemap:`.
5. **Envie o sitemap ao Google e ao Bing** (ver seção "Indexação" abaixo).

Gates (todos por comando): páginas respondem 200 no domínio do cliente com conteúdo no HTML; `curl -A "<UA de cada bot>"` retorna 200; JSON-LD validado; sitemap acessível e enviado.

### Passo 5 — F3: instale o agente (2–3 semanas)

Arquitetura: widget vanilla JS em **shadow DOM** → **Edge Function serverless** (a chave do modelo fica no servidor) → LLM barato/rápido → **Postgres** com as tabelas:

```
conversations   # sessões (token anônimo)
messages        # histórico por turno
leads           # capturas do agente
catalogo_fichas # o catálogo dentro do banco (fonte do contexto do agente)
injection_flags # suspeitas de prompt injection (alimenta o loop F4)
```

Decisões que transferem para qualquer cliente: navegação e captura = **tool use com schema fechado** (`navigate_to(rota)` com rota em enum — o modelo não inventa URL; `capture_lead(...)`); **catálogo como única fonte** ("se não está no catálogo, diga que não está registrado e ofereça contato"); rate limit por IP+sessão + `max_tokens` curto + limite de turnos; banco **dedicado** se o cliente já tem infra de produção.

**O cérebro: por que OpenRouter como padrão.** A Edge Function não chama a API do provedor do modelo diretamente — chama o [OpenRouter](https://openrouter.ai), que expõe centenas de modelos atrás de **um único endpoint no formato OpenAI** (`chat/completions`). As razões, na ordem que pesaram no cliente-zero:

1. **Um formato só.** As APIs dos provedores divergem em detalhes traiçoeiros (autenticação, onde vai o system prompt, `max_tokens` obrigatório ou não, formato de tool use, eventos de streaming). O plano original tinha uma tabela inteira de diferenças OpenAI×Anthropic para não portar código às cegas — o OpenRouter normaliza tudo pro formato OpenAI e essa tabela deixa de ser problema.
2. **Trocar de modelo = trocar uma string.** Se o modelo escolhido ficar caro, lento ou for descontinuado, muda-se o nome do modelo na config — sem reescrever a integração. Bom para testar alternativas no gate de qualidade.
3. **Dashboard de custo por requisição/modelo** — é onde se mede o gate "custo médio por conversa ≤ teto acordado", sem instrumentação própria.
4. **Uma chave só, no servidor.** A chave do OpenRouter vive como secret da Edge Function; o navegador nunca a vê.

Trade-off honesto: é um intermediário (pequena latência e margem sobre o preço do modelo) e recursos nativos específicos de um provedor podem não estar disponíveis. Para chat de site com modelo barato, a simplicidade vence. *No cliente-zero: Claude Haiku via OpenRouter — rápido e barato o suficiente para chat, bom o suficiente para seguir roteiro + tools.*

Gates antes do go-live (nenhum é opcional): demo E2E com lead **confirmado com query no banco**; suíte adversarial ≥20 ataques contra a função **publicada** (0 vazamentos, 0 ações fora do schema); DevTools sem nenhuma chamada do navegador à API do modelo; rate limit comprovado com flood; custo por conversa ≤ teto acordado; **cliente aprova 10 transcrições**.

### Passo 6 — F4: ligue o loop e prove o delta (contínuo)

Instale os **4 sinais** antes de esperar tráfego (páginas novas costumam nascer sem analytics — no cliente-zero, as páginas atrás do proxy estático escapavam do analytics do site e precisaram de contadores próprios):

1. Visitas do site principal (o analytics que o cliente já tem).
2. Visitas às páginas de conhecimento (contador próprio se o proxy escapar do analytics).
3. Uso do agente (as tabelas do Postgres).
4. **AI Share of Voice**: re-rode a bateria v1 mensalmente e calcule o delta contra o baseline — este é o número do case.

Loop semanal (com ≥20 conversas/semana; menos → quinzenal): classificador marca injection/frustração/perguntas sem ficha → relatório de 10 linhas → perguntas sem ficha viram fichas novas → mudança de prompt só com golden set + aprovação. Share of voice estagnado após 3 meses → a alavanca vira **autoridade externa** (as fontes que as IAs citam no lugar do cliente).

---

## Operação contínua (depois que está no ar)

Toda atualização de conteúdo percorre **três destinos**: o site público, o banco do agente e os buscadores. Esquecer qualquer um deles deixa uma superfície desatualizada.

### 1. Atualizar uma ficha → republicar o site

```bash
# edite a ficha em catalogo/<tipo>/<slug>.md (com fonte real; sem confirmação → PENDENTE)
node scripts/gerar-site.mjs        # regenera conhecimento/ (HTML + JSON-LD + sitemap + api/*.json)
# copie a saída para o clone do repo público do site e faça push
# o deploy do Pages é automático; o sitemap.xml sai com as novas datas de atualização
```

### 2. Atualizar o banco do agente

O agente responde a partir da tabela `catalogo_fichas` no Postgres — **não** lê os arquivos MD. Depois de mudar qualquer ficha:

```bash
node scripts/sync-catalogo-supabase.mjs   # catalogo/ → tabela catalogo_fichas
```

Sem esse sync, o site diz uma coisa e o agente diz outra. Se a mudança foi no **prompt** do agente (não no catálogo), o caminho é outro: golden set re-executado + aprovação do cliente antes do deploy.

### 3. Indexação: como o `/conhecimento` é atualizado no Google e no Bing

**Setup inicial (uma vez):**
1. Verifique a propriedade do domínio no [Google Search Console](https://search.google.com/search-console) e no [Bing Webmaster Tools](https://www.bing.com/webmasters) (verificação por DNS ou meta tag).
2. Envie o sitemap: `https://dominio-do-cliente/conhecimento/sitemap.xml` em **Sitemaps → Adicionar** (nos dois).
3. Peça **indexação manual** das ~20 páginas estratégicas (Search Console → Inspeção de URL → Solicitar indexação) — acelera a primeira onda.

**A cada atualização de conteúdo:**
- O `sitemap.xml` regenerado já sai com o `lastmod` novo — Google e Bing re-rastreiam sozinhos a partir dele; **não precisa reenviar o sitemap** a cada mudança.
- Mudança importante numa página estratégica → vale pedir reindexação manual daquela URL específica.

**O que esperar e verificar:**
- Janela de indexação: **2–6 semanas** para a primeira onda. Não bloqueia as fases seguintes — corre em paralelo.
- Gate verificável: Search Console mostrando **≥10 páginas indexadas** (relatório "Páginas").
- Teste mensal dos bots de IA (WAF/CDN pode passar a bloqueá-los silenciosamente): `curl -A "OAI-SearchBot" https://dominio/conhecimento/` deve retornar 200 com HTML.
- Os bots de **IA** (OAI-SearchBot, ClaudeBot, PerplexityBot) não dependem do Search Console — eles seguem o `robots.txt` e o sitemap direto. O Search Console/Bing cobrem Google AI Overviews/Gemini e Bing/Copilot, respectivamente.

---

## Resultado do cliente-zero (resumo)

- **Antes (F0):** 0% de aparição do INEMA em 52 medições de prompts neutros; colisão de nome com órgão público homônimo.
- **Construído:** 54 fichas de catálogo → site indexável em `inema.club/conhecimento/*` (JSON-LD, sitemap, bots de IA liberados e testados) → agente de chat com gates de segurança comprovados (22 ataques, 0 vazamentos) → 4 sinais de medição instalados.
- **Depois:** re-medição mensal em andamento (janela de indexação de 2–6 semanas a partir de 2026-07-10).

*Docs do cliente-zero copiados do repo privado do programa em 2026-07-10.*
