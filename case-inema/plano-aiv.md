# Plano AIV — AI Visibility do INEMA (War Game F0–F4)

*Versão 1.0 · 2026-07-10 · Escrito para um agente começando do zero, sem contexto anterior.*

---

## Preâmbulo

### O que é este projeto

O **AIV** (AI Visibility) é o programa que torna o **INEMA** — plataforma brasileira de formação prática em IA, agentes e automação, criada por Nei Maldaner — **encontrável, citável e recomendável por sistemas de IA** (ChatGPT, Claude, Gemini, Perplexity, Google AI Overviews), e que instala no site um **agente de chat comercial** que guia visitantes, qualifica interesse e captura leads.

O INEMA é o cliente-zero. O resultado documentado vira case e depois produto vendável ("Catálogo de Conhecimento para IA" / auditoria AEO-GEO) para outras empresas.

### Contexto que você (agente executor) precisa saber

- **Portal INEMA** = `inema.club`, projeto Next.js no Vercel (repo próprio do portal).
- **Este repo** = o repo privado do programa AIV. Todo o conteúdo do AIV vive aqui.
- **Publicar = commit + push.** O deploy é automático (webhook git → Vercel / GitHub Pages). **Nunca** interagir com o dashboard do Vercel, nem re-disparar deploys, nem monitorar status de deploy. Tarefa termina no push.
- **Autor de todo commit:** a conta oficial do projeto (author E committer).
- **Documentos-fonte** em `doc/`: tese de mercado (`catalogo_conhecimento_ia.md`), estrutura dos catálogos (`catalogos_conhecimento_inema.md`), visibilidade externa (`como_as_ias_publicas_conhecem_o_inema.md`), prompt original do war game (`prompt-war-game-portugues.md`), proposta aprovada (`proposta-aiv.md`).
- **Aprovações:** mudanças de comportamento do agente de chat em produção exigem aprovação explícita do Nei. Mudanças no portal (rewrite, robots) são pontuais e também passam por ele.

### Por que war game

Este plano **assume que barreiras vão aparecer**: indexação que não acontece, rewrite que quebra, modelo que alucina, injection, custo estourando. Cada fase declara suposição otimista E pessimista, modos de falha com correção, critérios de saída **verificáveis** (nada fecha "no olho") e orçamento de tentativas (quando parar de insistir e replanejar). Nenhuma fase é dada como pronta sem o teste correspondente executado e o resultado real registrado.

---

## Caminho crítico

```
F0 Diagnóstico ──► F1 Catálogo (50 fichas) ──► F2 Publicação ──► F3 Agente ──► F4 Loop
   (2 dias)           (1–2 semanas)              (1 semana build      (2–3        (contínuo)
                                                  + 2–6 sem. de       semanas)
                                                  indexação em
                                                  paralelo)
```

Dependências reais (o resto pode sobrepor):

1. **F0 antes de tudo** — sem baseline, nenhum resultado posterior é mensurável.
2. **F1 → F2**: as páginas públicas são geradas das fichas. F2 pode começar com as primeiras 20 fichas prontas; não precisa das 50.
3. **F1 → F3**: o agente responde a partir do catálogo. F3 pode iniciar o esqueleto técnico (Edge Function, tabelas, widget) em paralelo à F1, mas o **gate de qualidade de conversa** exige ≥30 fichas revisadas.
4. **A janela de indexação da F2 (2–6 semanas) NÃO bloqueia F3** — corre em paralelo.
5. **F4 só liga depois de F3 em produção** (loop do agente) e de F2 publicada (re-medição de share of voice).

O caminho crítico de esforço humano é **F1 (revisão das fichas)** — todo o resto é automatizável; a revisão do Nei não é. Mitigação: lotes de 10 fichas.

---

## Stack

| Camada | Escolha | Por quê |
|---|---|---|
| Fonte de conteúdo | Markdown + front matter YAML no repo `aiv/catalogo/` | Versionável, legível por humano e máquina, é o formato dos docs-fonte |
| Validação de fichas | Script Node (`scripts/validar-fichas.mjs`) com schema por tipo de ficha | Critério de saída verificável da F1 |
| Site público | HTML estático gerado por script Node (`scripts/gerar-site.mjs`) → pasta `conhecimento/` → GitHub Pages | Self-contained, zero dependência de build externo, publica por push |
| Exposição no domínio | **Rewrite no Vercel**: `inema.club/conhecimento/:path*` → `<org>.github.io/conhecimento/:path*` (repo público dedicado, separado do repo do programa que é privado) | Autoridade AEO acumula em `inema.club`; conteúdo atualiza sem tocar no portal |
| API de conteúdo | JSON estático gerado junto com o site (`conhecimento/api/*.json`) | "API" sem servidor: o agente e terceiros consomem os mesmos arquivos |
| Cérebro do agente | **Claude Haiku** (`claude-haiku-4-5-20251001`) via API Anthropic | Definido no prompt original; custo/latência adequados a chat de site |
| Backend do agente | **Supabase Edge Functions** (Deno) — o navegador **nunca** chama a API do modelo | Definido no prompt original; chave da API fica no servidor |
| Persistência | **Supabase Postgres**: `conversations`, `messages`, `leads`, `eval_runs` | Definido no prompt original; alimenta o loop da F4 |
| Widget | Vanilla JS embarcado no portal (1 `<script>`) | Sem framework, sem conflito com o Next.js do portal |
| Diagnóstico/medição | Planilha CSV + relatórios MD em `diagnostico/` | Começa manual; automação só depois que o processo estabilizar |

## Estrutura do repositório

```
aiv/
├── doc/                    # documentos-fonte, proposta, este plano
├── catalogo/               # FONTE ÚNICA — fichas MD+YAML
│   ├── institucional/  ├── cursos/  ├── projetos/
│   ├── conceitos/      ├── faq/     ├── servicos/  └── cases/
├── conhecimento/           # SAÍDA gerada — HTML público (GitHub Pages)
│   ├── index.html  ├── <slug>/index.html  ├── sitemap.xml  └── api/*.json
├── scripts/                # gerar-site.mjs, validar-fichas.mjs, extrair-ficha.mjs
├── diagnostico/            # baseline-YYYY-MM.md + medicoes.csv
├── agente/
│   ├── widget/             # JS do widget + abas de ação rápida
│   ├── supabase/           # edge functions + migrations SQL
│   ├── prompts/            # system prompt versionado + roteiros de tour
│   └── evals/              # golden set + prompts adversariais + runner
└── loop/                   # relatorios semanais + propostas de mudança
```

**Regra de ouro:** `catalogo/` é a fonte única. `conhecimento/` é gerado — nunca editar à mão. O agente responde só com o que está no catálogo.

---

## Metodologia

1. **Fase só fecha com critérios de saída executados** — cada critério é um comando, teste ou artefato conferível, não uma opinião.
2. **Orçamento de tentativas** — cada fase declara quantas tentativas/quanto tempo antes de parar e replanejar. Estourou → registra a barreira em `doc/decisoes.md` e escala para o Nei.
3. **Suposição pessimista é plano, não medo** — para cada uma existe correção pré-desenhada (tabela mestre).
4. **Nada de produção sem regressão + aprovação** — vale para prompts do agente e mudanças no portal.
5. **Medir antes e depois** — F0 é a régua; toda melhoria alegada aponta para uma medição.

---

## F0 — Diagnóstico (baseline de AI Share of Voice)

**Objetivo:** registrar, de forma reproduzível, como ChatGPT, Claude, Gemini e Perplexity respondem hoje a ~20 perguntas reais sobre o nicho do INEMA — quem citam, com quais fontes, com quais erros.

**Bateria mínima (20 prompts, 3 execuções cada, por ferramenta):**
"Quem ensina agentes de IA no Brasil?" · "O que é INEMA?" · "O INEMA.club é bom para aprender IA?" · "Quem é Nei Maldaner?" · "Qual curso prático de Claude Code em português?" · "Como aprender a criar agentes sem saber programar?" · "Compare opções de formação em IA no Brasil" · variações com cidade/perfil/preço.

**Registro (CSV `diagnostico/medicoes.csv`):** data, ferramenta, prompt, execução nº, resposta resumida, INEMA citado? (sim/não/parcial), concorrentes citados, fontes citadas, erros da IA, ação recomendada.

- **Suposição otimista:** em 1 dia a bateria completa roda; o INEMA já aparece em alguma resposta via GitHub/YouTube.
- **Suposição pessimista:** respostas variam muito entre execuções (é o esperado — por isso 3 execuções: mede-se distribuição, não foto); o INEMA não aparece em nenhuma; alguma ferramenta indisponível ou sem modo de busca.

**Modos de falha prováveis → correção:**
| Falha | Correção |
|---|---|
| Variância alta entre execuções | Já embutida: 3 execuções/prompt; reportar % de aparição, não sim/não |
| Ferramenta sem acesso/busca desligada | Registrar N/A e seguir; baseline parcial vale mais que nenhum |
| Prompts enviesados ("o INEMA é bom?") induzem citação | Bateria fixa com maioria de prompts neutros (sem citar INEMA) |
| Resultado 100% negativo desanima | Reenquadrar: é a prova do problema e o "antes" do case |

**Critérios de saída (verificáveis):**
1. `diagnostico/medicoes.csv` com ≥ 20 prompts × ≥ 3 ferramentas × 3 execuções.
2. `diagnostico/baseline-2026-MM.md` com: % de aparição do INEMA por ferramenta, top concorrentes citados, top fontes citadas, lista de erros factuais das IAs.
3. Commit + push no repo.

**Orçamento:** 2 dias corridos; 1 re-tentativa por ferramenta indisponível; se 2+ ferramentas inacessíveis, fechar com as disponíveis e registrar.

---

## F1 — Catálogo MVP (50 fichas)

**Objetivo:** 50 fichas padronizadas, revisadas e versionadas em `catalogo/`: 10 institucionais, 10 cursos, 10 projetos, 10 FAQ, 5 serviços, 5 cases.

**Ficha-padrão (front matter obrigatório):**

```yaml
---
slug: formacao-agentes-ia
tipo: curso            # institucional|curso|projeto|conceito|faq|servico|case
titulo: Formação em Agentes de IA
resumo: >-             # 1–2 frases que respondem a pergunta diretamente
  ...
publico: [profissionais, empresarios]
status: rascunho       # rascunho|revisado|publicado
revisado_por: null
fonte: URL-ou-documento
confianca: alta        # alta|media|PENDENTE
atualizado_em: 2026-07-10
relacionados: [claude-code, mcp]
---
```

**Processo:** inventário das fontes (portal, GitHub oficial, cursos publicados, YouTube) → extração assistida por IA com a regra **"não invente; marque PENDENTE"** → lote de 10 → revisão do Nei → `status: revisado` → próximo lote.

- **Suposição otimista:** o material existente cobre 80%+ das fichas; extração rende 5–8 fichas/hora; Nei revisa 1 lote/dia.
- **Suposição pessimista:** fontes contraditórias ou desatualizadas (versões antigas de cursos); extração alucina detalhes plausíveis; revisão vira gargalo de semanas; números/preços não existem publicados.

**Modos de falha prováveis → correção:**
| Falha | Correção |
|---|---|
| Alucinação na extração | Campo `fonte` obrigatório por ficha; sem fonte → `confianca: PENDENTE`; validador rejeita publicação de PENDENTE |
| Fontes contraditórias | A mais recente vence; divergência anotada no corpo e sinalizada na revisão |
| Gargalo de revisão | Lotes de 10; F2 pode partir com 20 revisadas; nunca esperar as 50 |
| Fichas inconsistentes entre si | `scripts/validar-fichas.mjs` roda em cada commit: schema por tipo, campos obrigatórios, slugs únicos, links `relacionados` existentes |
| Dados sensíveis/preço que Nei não quer público | Campo `visibilidade: interna` — entra no catálogo do agente, não no site |

**Critérios de saída (verificáveis):**
1. `node scripts/validar-fichas.mjs` → 0 erros em 50 fichas.
2. ≥ 50 fichas com `status: revisado` e `revisado_por` preenchido.
3. 0 fichas `publicado` com `confianca: PENDENTE`.
4. As 20 fichas correspondentes às 20 páginas estratégicas (doc `como_as_ias_publicas...`) existem e estão revisadas.

**Orçamento:** 2 semanas; máx. 2 rodadas de extração por ficha (não converge → escrever à mão); se na semana 2 houver < 30 revisadas, reduzir MVP para 30 e registrar.

---

## F2 — Publicação AEO/GEO

**Objetivo:** as fichas viram páginas HTML públicas servidas em `inema.club/conhecimento/...` (GitHub Pages atrás de rewrite do Vercel), indexáveis, com dados estruturados e sitemap enviado.

**Entregas técnicas:**
1. `scripts/gerar-site.mjs`: catálogo → `conhecimento/` — HTML semântico, resposta direta no 1º parágrafo, JSON-LD por tipo (`Organization`, `Course`, `FAQPage`, `Person`, `Article`), `<link rel="canonical">` apontando para `https://inema.club/conhecimento/<slug>`, data de atualização visível, `sitemap.xml` e `api/*.json`.
2. GitHub Pages ativo no repo público dedicado (branch main, raiz) — repo separado do repo do programa (que continua privado, só planejamento/scripts/diagnóstico).
3. No portal (única mudança, PR pontual): rewrite `/conhecimento/:path*` → `https://<org>.github.io/conhecimento/:path*`; robots.txt liberando `OAI-SearchBot`, `Claude-SearchBot`, `PerplexityBot`, `Googlebot`, `Bingbot` (+ `GPTBot`/`ClaudeBot` se Nei aprovar treino) e apontando `Sitemap: https://inema.club/conhecimento/sitemap.xml`.
4. Sitemap enviado ao Google Search Console e Bing Webmaster Tools (ação do Nei; fornecer passo a passo).
5. `sameAs` no JSON-LD `Organization` amarrando site ↔ GitHub ↔ YouTube ↔ LinkedIn.

- **Suposição otimista:** rewrite funciona de primeira; Google indexa as primeiras páginas em dias; rich results test passa.
- **Suposição pessimista:** rewrite quebra links relativos/assets; Vercel ou Cloudflare bloqueia bots de IA silenciosamente (challenge/403 só para eles); indexação leva 4–6 semanas; conteúdo duplicado `github.io` vs `inema.club` compete consigo mesmo.

**Modos de falha prováveis → correção:**
| Falha | Correção |
|---|---|
| Links/assets quebram atrás do rewrite | Gerador usa só caminhos relativos dentro de `conhecimento/`; teste automático: crawl local das páginas geradas checando 404 |
| Duplicidade github.io × inema.club | `canonical` para `inema.club` em toda página (github.io vira espelho não-canônico) |
| Bot de IA bloqueado por WAF/Cloudflare | Teste com `curl -A "<UA do bot>"` para cada bot listado; se challenge/403, ajustar regra — critério de saída explícito |
| Indexação lenta | Esperado: janela de 2–6 semanas em paralelo, não bloqueia F3; pedir indexação manual das 20 páginas no Search Console |
| JSON-LD inválido | Validar no Rich Results Test / validator.schema.org antes do push |
| Página vaga que IA não cita | Template força: resposta direta no 1º parágrafo, números, datas, listas, FAQ no fim |

**Critérios de saída (verificáveis):**
1. As 20 páginas estratégicas respondem 200 em `https://inema.club/conhecimento/<slug>` **e** o HTML contém o conteúdo (não shell vazio).
2. `curl -A` com o user-agent de cada bot liberado retorna 200 + HTML em ≥ 3 páginas testadas.
3. JSON-LD de 3 tipos diferentes passa no validador sem erro.
4. Sitemap acessível em `inema.club/conhecimento/sitemap.xml` e enviado (screenshot/confirmação do Nei).
5. Search Console mostra ≥ 10 páginas indexadas (critério da janela: até 6 semanas, verificado em F4).

**Orçamento:** 1 semana de build + 3 tentativas para o rewrite (falhou 3× → fallback: subdomínio `conhecimento.inema.club` via CNAME, registrado como decisão); janela de indexação de 6 semanas monitorada mas não bloqueante.

---

## F3 — Agente de chat no site (war game do prompt original)

**Objetivo:** widget de chat em `inema.club` com abas flutuantes de ação rápida ("Me guie pelo site"), tour guiado que navega páginas reais narrando, qualificação do visitante durante o tour, introdução natural da oferta diante de sinais de compra e captura de lead. Cérebro: Claude Haiku via Supabase Edge Function; conversas em Postgres; **o navegador nunca acessa a API do modelo**.

### Arquitetura

```
Visitante ── widget JS (portal) ── HTTPS ──► Edge Function `chat` (Deno)
                 │                                  │ x-api-key (secret)
                 │ executa ações de tour            ▼
                 │ {navigate_to, highlight}     API Anthropic (Haiku)
                 ▼                                  │
            páginas reais                           ▼
                                        Postgres: conversations/messages/leads
                                                    ▲
                        catálogo (api/*.json) ──────┘  contexto por relevância
```

**Decisões de desenho:**
- **Tour = tool use, não texto livre.** O modelo recebe tools `navigate_to(rota)` e `capture_lead(nome, email, interesse)`; `rota` é um **enum fechado** com as rotas reais do site. O widget executa a navegação. Modelo não inventa URL — o schema não deixa.
- **Catálogo como única fonte:** a Edge Function seleciona fichas relevantes (busca simples por palavras-chave/tags nos JSON da F2) e injeta no contexto. Instrução dura: "se não está no catálogo, diga que não está registrado e ofereça contato".
- **Sessão anônima** com token gerado pela função (`conversation_id`), rate limit por IP+sessão, tamanho máximo de mensagem, orçamento de tokens por conversa.
- **Streaming SSE** da Edge Function para o widget (latência percebida baixa com Haiku).

### Onde a API do Claude difere dos hábitos OpenAI (não portar cegamente)

| Hábito OpenAI | Na API Anthropic (Messages) |
|---|---|
| `Authorization: Bearer sk-...` | Header **`x-api-key`** + **`anthropic-version: 2023-06-01`** obrigatório |
| Mensagem `{"role":"system"}` no array | **`system` é parâmetro top-level**; role `system` no array = erro |
| `max_tokens` opcional | **`max_tokens` obrigatório** — esquecê-lo = 400 na primeira chamada |
| Roles em qualquer ordem | Mensagens devem **alternar user/assistant**, começando por `user` — ao gravar/reler histórico do Postgres, garantir a alternância (mesclar consecutivas do mesmo role) |
| `functions`/`tool_calls` + `function_call` | Tools com **`input_schema`** (JSON Schema); a resposta vem como **bloco de conteúdo `tool_use`**; o resultado volta como mensagem `user` com bloco **`tool_result`** (não role `tool`) |
| `finish_reason` | **`stop_reason`**: `end_turn`, `max_tokens`, `stop_sequence`, `tool_use` — tratar `tool_use` no loop e `max_tokens` como resposta truncada |
| Streaming: chunks uniformes `delta.content` | **SSE com eventos tipados**: `message_start`, `content_block_start`, `content_block_delta`, `message_delta`, `message_stop` — o parser do widget/função é diferente |
| `response_format: json_object` | **Não existe JSON mode** — forçar JSON via tool use (`tool_choice`) ou prefill; aqui o tour já usa tools, então saída estruturada = tool |
| `temperature` 0–2, `frequency_penalty`, `logit_bias` | `temperature` **0–1**; penalties/logit_bias **não existem** — não portar esses parâmetros |
| Sem prefill | **Prefill suportado**: última mensagem `assistant` inicia a resposta (útil para manter persona/idioma); sem espaço em branco no fim |
| Cache implícito | **Prompt caching explícito** com `cache_control` — usar no system + fichas do catálogo (bloco grande e estável) para cortar custo/latência por turno |
| Erros 429 | Além de 429, **529 `overloaded_error`** — retry com backoff + mensagem de fallback no widget ("um instante…") |

- **Suposição otimista:** Haiku conduz o tour com naturalidade seguindo o roteiro + tools; latência < 2 s/turno com streaming; primeiros leads na primeira semana.
- **Suposição pessimista:** modelo desvia do roteiro, fica genérico ou "vendedor demais"; visitantes tentam injection ("ignore suas instruções…"); Edge Function esbarra em timeout/limites do plano Supabase; custo por conversa acima do aceitável; widget conflita com CSS/JS do portal.

**Modos de falha prováveis → correção:**
| Falha | Correção |
|---|---|
| Prompt injection via visitante | Input do visitante nunca vira instrução: system fixo, catálogo read-only, tools com schema fechado; suite adversarial (≥ 20 ataques) como gate; flag de suspeita gravada p/ F4 |
| Alucinar página/oferta inexistente | Rotas = enum no tool schema; ofertas só do catálogo; eval automática checa URLs e claims contra o catálogo |
| Conversa genérica/engessada | Gate de ≥ 30 fichas; roteiros de tour por persona em `agente/prompts/`; eval humana de 10 conversas antes do go-live |
| Vazamento do system prompt | Teste adversarial dedicado; system não contém segredo algum (defesa em profundidade: nada sensível lá) |
| Custo bomba (loop/abuso) | `max_tokens` curto, limite de turnos por conversa, rate limit IP+sessão, orçamento diário com corte + alerta |
| Timeout do Edge Function | Streaming desde o 1º token; Haiku (rápido); orçamento de contexto (fichas top-k, não o catálogo todo) |
| Widget quebra o portal | JS isolado (shadow DOM), sem dependências; testar nas 5 páginas principais + mobile |
| CORS/CSP | Origem do portal na allowlist da função; testar preflight no staging |

**Critérios de saída (verificáveis):**
1. **Demo E2E gravada:** visitante abre aba "Me guie pelo site" → tour navega ≥ 3 páginas reais com narração → responde ≥ 2 perguntas de qualificação → sinal de compra → lead salvo em `leads` no Postgres.
2. Suite adversarial (≥ 20 ataques): 0 vazamentos de instrução, 0 ações fora do schema.
3. Busca no DevTools/network: **nenhuma** chamada do navegador para `api.anthropic.com`.
4. Custo médio por conversa medido em staging ≤ limite acordado com Nei.
5. Rate limit comprovado (script de flood recebe 429 da função).
6. Aprovação explícita do Nei sobre 10 transcrições de conversas de teste.

**Orçamento:** 3 semanas; até 5 iterações de prompt para o gate de qualidade (não passou → reduzir escopo: tour roteirizado fixo com respostas livres só no Q&A, registrado como decisão); 2 tentativas de arquitetura de streaming (falhou → resposta não-streamada com indicador de digitação).

---

## F4 — Loop de medição e melhoria contínua

**Objetivo:** (a) re-medir mensalmente o AI Share of Voice contra o baseline da F0; (b) rodar o **loop semanal de autoavaliação do agente** exigido pelo prompt original — análise de todas as conversas em busca de prompt injection e adaptação do estilo de fala aos visitantes reais, **dentro de limites rígidos**, sem nunca perder o profissionalismo, e **nada em produção sem avaliação de regressão + aprovação explícita do Nei**.

**Mecânica semanal (agente + humano):**
1. Job puxa as conversas da semana do Postgres.
2. Classificador (Haiku, prompt fixo) marca: tentativas de injection, frustração do visitante, perguntas sem resposta no catálogo, momentos de queda de qualidade.
3. Gera `loop/relatorio-YYYY-WW.md`: nº de conversas, leads, taxa de conclusão de tour, injections detectadas, top perguntas sem ficha (→ viram fichas novas na F1 contínua), proposta de ajuste de estilo (se houver).
4. Proposta de mudança = diff no prompt versionado em `agente/prompts/` + execução do **golden set** (≥ 15 conversas de referência re-executadas e comparadas: qualificação correta? tom profissional? zero alucinação?).
5. Nei aprova → merge + push. Sem aprovação, nada muda.

**Limites rígidos do estilo (hardcoded, fora do alcance do loop):** sempre PT-BR profissional; nunca prometer resultado garantido; nunca inventar preço/prazo; nunca pressionar após "não"; identidade e credenciais fixas.

- **Suposição otimista:** volume de conversas dá sinal útil já na semana 2; share of voice sobe no mês 2–3 conforme a indexação da F2 pega.
- **Suposição pessimista:** poucas conversas (tráfego baixo) → sinal estatístico fraco; adaptação de estilo degrada sutilmente ("drift" simpático demais); share of voice não se move em 3 meses.

**Modos de falha prováveis → correção:**
| Falha | Correção |
|---|---|
| Drift de personalidade acumulado | Limites hardcoded + eval de profissionalismo no golden set + diff sempre revisado por humano |
| Mudança passa na regressão mas piora em produção | Golden set cresce com casos reais ruins de cada semana; rollback = revert do commit do prompt |
| Poucos dados | Threshold: < 20 conversas/semana → loop roda quinzenal; relatório aponta ações de tráfego, não de prompt |
| Share of voice estagnado após 3 meses | Diagnóstico dirigido: fontes que as IAs citam no lugar → plano de prova externa (YouTube, LinkedIn, parceiros, reviews) — a alavanca passa a ser autoridade externa, não mais conteúdo |
| Relatório que ninguém lê | 10 linhas no topo: números da semana + 1 decisão pedida |

**Critérios de saída (o loop não "fecha" — critérios do primeiro ciclo):**
1. 1 relatório semanal completo gerado a partir de conversas reais.
2. 1 proposta de mudança processada ponta a ponta (golden set executado + decisão do Nei registrada), nem que seja "rejeitada".
3. 1 re-medição mensal (`diagnostico/medicoes.csv` acrescido) comparada ao baseline com delta calculado.

**Orçamento:** setup em 1 semana; operação contínua com teto de ~2 h/semana de atenção humana; se exceder por 3 semanas seguidas, automatizar o gargalo ou reduzir frequência.

---

## Tabela mestre de cenários de falha

| # | Categoria | Cenário | Como detectar | Como recuperar |
|---|---|---|---|---|
| 1 | Infraestrutura | Rewrite do Vercel quebra ou é removido num deploy do portal | Monitor simples (curl semanal de 3 URLs `/conhecimento`) falha | Restaurar regra no `next.config`/`vercel.json`; fallback: subdomínio CNAME |
| 2 | Infraestrutura | GitHub Pages fora do ar / build da Pages falha | Mesma sonda semanal; e-mail do GitHub | Conteúdo é estático no repo: republicar; incidente passa sozinho na maioria dos casos |
| 3 | Infraestrutura | Limites do plano Supabase (invocações/duração de função, linhas) | Dashboard Supabase; erros 5xx na função | Rate limit mais agressivo; upgrade de plano é decisão do Nei |
| 4 | Infraestrutura | Cloudflare/WAF desafia bots de IA silenciosamente | Teste mensal `curl -A` com UA de cada bot | Regra de allow por UA/ASN; documentar em `doc/decisoes.md` |
| 5 | API do modelo | 429 / 529 overloaded em horário de pico | `stop_reason`/status logados por turno em `messages` | Retry c/ backoff (máx. 2) → mensagem de fallback simpática no widget |
| 6 | API do modelo | Resposta truncada (`stop_reason: max_tokens`) | Campo logado; eval semanal conta ocorrências | Aumentar `max_tokens` do turno ou instruir respostas mais curtas |
| 7 | API do modelo | Deprecação/migração de versão do Haiku | Changelog Anthropic; erro de modelo inexistente | Nome do modelo em config única; golden set re-roda na troca antes do push |
| 8 | API do modelo | `tool_use` com input inválido/inesperado | Validação do input contra o schema na função | Recusar e re-pedir ao modelo (1×); persistir → responder sem tool e logar |
| 9 | API do modelo | Histórico viola alternância user/assistant ao reidratar do banco | Erro 400 da API | Normalizador que mescla mensagens consecutivas do mesmo role antes de enviar |
| 10 | Frontend | Widget conflita com CSS/JS do portal | Teste manual nas 5 páginas principais + mobile a cada mudança | Shadow DOM + zero dependências; desativar via flag no portal |
| 11 | Frontend | Adblocker bloqueia o script do widget | Teste com uBlock; % de sessões sem widget carregado | Servir do mesmo domínio do portal (não de terceiro); degradar sem erro |
| 12 | Frontend | Tour navega e o contexto do chat se perde entre páginas | Teste E2E do tour | `conversation_id` em sessionStorage; widget re-hidrata do Postgres ao carregar |
| 13 | Qualidade | Conversa genérica/engessada (proibida pelo prompt original) | Eval humana de amostras; nota de qualidade no loop semanal | Mais fichas no contexto; roteiros por persona; ajuste de prompt via F4 |
| 14 | Qualidade | Alucinação de oferta, preço ou página | Eval automática compara claims × catálogo; reclamações | Regra "só catálogo"; ficha faltante → criar ficha (F1 contínua) |
| 15 | Qualidade | Idioma errado ou tom vendedor-agressivo | Classificador semanal + limites hardcoded | Prefill de persona; correção de prompt com regressão |
| 16 | Segurança | Prompt injection (visitante instrui o agente) | Classificador semanal + heurísticas na função (padrões conhecidos) | System imutável, tools com schema fechado; sessão marcada entra no golden set |
| 17 | Segurança | Scraping/flood do endpoint da função (custo bomba) | Rate limit atingido; orçamento diário de tokens | 429 por IP+sessão; corte diário com alerta; chave nunca no cliente |
| 18 | Segurança | PII sensível digitada pelo visitante fica no banco | Scan semanal de PII nas mensagens | Minimizar retenção (política de expurgo); PII de lead só nos campos de `leads` |
| 19 | Segurança | Spam de leads falsos | Leads com padrão repetitivo; e-mails inválidos | Validação de e-mail, honeypot no widget, rate limit de `capture_lead` |
| 20 | Loop | Golden set desatualizado aprova mudança ruim | Piora em produção na semana seguinte (métricas do relatório) | Rollback por revert; caso ruim entra no golden set (aprende com o furo) |
| 21 | Loop | Auto-adaptação foge dos limites (drift) | Diff de prompt sempre revisado; eval de profissionalismo | Limites hardcoded fora do alcance do loop; revert |
| 22 | Loop | Métricas de share of voice mal comparadas (bateria mudou) | Versão da bateria registrada no CSV | Bateria congelada v1; adições viram v2 medidas em paralelo |

---

## Sequência de execução resumida (para o agente que vai executar)

1. Ler este plano + `doc/proposta-aiv.md`. Confirmar acesso: repo `aiv`, portal, Supabase, chave Anthropic (server-side).
2. **F0**: montar bateria, rodar, registrar CSV + baseline. *Gate: critérios F0.*
3. **F1**: inventário → extração em lotes de 10 → revisão Nei. *Gate: validador 0 erros, ≥30–50 revisadas.*
4. **F2**: gerador + Pages + PR do rewrite/robots no portal + sitemap. *Gate: critérios F2 (indexação corre em paralelo).*
5. **F3**: migrations + Edge Function + widget + prompts + evals. *Gate: demo E2E + suite adversarial + aprovação Nei.*
6. **F4**: ligar o loop; primeiro ciclo completo; re-medição mensal.
7. Toda barreira que estourar orçamento → `doc/decisoes.md` + escalar ao Nei. Todo fechamento de fase → commit com o autor oficial do projeto + push.
