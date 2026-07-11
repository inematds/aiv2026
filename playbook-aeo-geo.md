# Playbook AEO/GEO — método replicável para novos clientes

*v1.0 · 2026-07-10 · Abstraído da execução no cliente-zero (INEMA). Este é o método vendável: tornar uma empresa encontrável, citável e recomendável por IAs públicas (ChatGPT, Claude, Gemini, Perplexity) e instalar um agente de chat comercial no site dela. O registro de como isso foi executado no INEMA está em `case-inema/o-que-foi-feito-aiv.md`; o plano detalhado original (com todas as tabelas de modos de falha) em `case-inema/plano-aiv.md`.*

---

## A tese em uma frase

IAs públicas só citam quem tem **conteúdo estruturado, verificável e indexável** respondendo às perguntas que os usuários realmente fazem — a maioria das empresas não tem, e isso é mensurável antes e depois.

## O método em 5 fases

```
F0 Diagnóstico ──► F1 Catálogo ──► F2 Publicação ──► F3 Agente ──► F4 Loop
  (a régua)        (o conteúdo)    (a exposição)     (a conversão)  (a prova)
```

Regras que valem para todas as fases (aprendidas no cliente-zero, não negociáveis):

1. **Nenhuma fase fecha "no olho"** — todo critério de saída é um comando executado com resultado real registrado.
2. **Nunca inventar dado do cliente.** Toda afirmação vem de fonte real (site, código, documento, conversa com o dono). Sem confirmação → marca `PENDENTE`, e o conteúdo público filtra os PENDENTE.
3. **Medir antes de mexer.** Sem F0, nenhum resultado posterior é demonstrável — e a demonstração é o produto.
4. **Testar local antes de publicar** toda mudança de infraestrutura. No cliente-zero isso pegou 2 bugs que só apareceriam em produção.
5. **Mudanças de comportamento do agente em produção exigem aprovação explícita do cliente.**

---

## F0 — Diagnóstico (AI Share of Voice baseline)

**Pergunta que responde:** "quando alguém pergunta a uma IA sobre o seu nicho, você aparece?"

**Passos:**
1. Levantar com o cliente ~20 perguntas **neutras** que o público-alvo dele faria a uma IA (sem citar a marca — prompts enviesados invalidam a medição). Ex. genérico: "quem oferece X no Brasil?", "qual o melhor curso/serviço de Y?", "compare opções de Z".
2. Congelar a bateria como **v1** — nunca alterar depois (adições viram v2 medida em paralelo, senão a comparação mensal quebra).
3. Rodar cada prompt **3×** em cada IA disponível (variância entre execuções é esperada; mede-se distribuição, não foto). Automatizar por API onde houver chave; manual onde não houver.
4. Registrar em CSV: data, ferramenta, prompt, execução, cliente citado? (sim/não/parcial), concorrentes citados, **fontes citadas**, erros factuais da IA.
5. Escrever o baseline: % de aparição por ferramenta, top concorrentes, top fontes, erros. **As fontes que as IAs citam no lugar do cliente são o mapa de onde a autoridade dele precisa existir.**

**Entregável pro cliente:** relatório de baseline — costuma ser o momento "uau" da venda (no cliente-zero: 0% de aparição + colisão de nome com um órgão público homônimo + bio do fundador desatualizada nas fontes).

**Critérios de saída:** CSV com ≥20 prompts × ≥3 execuções nas ferramentas disponíveis; baseline escrito. Ferramenta inacessível → registrar N/A e seguir (baseline parcial > nenhum).

**Orçamento:** 2 dias.

**O que adaptar por cliente:** as 20 perguntas (derivam do nicho); quais IAs importam pro público dele.

---

## F1 — Catálogo de conhecimento (~50 fichas)

**Pergunta que responde:** "o que a IA deveria saber sobre você — em formato que ela consiga citar?"

**Estrutura:** fichas Markdown + front matter YAML numa pasta versionada, por tipo: **institucional** (quem é, o que faz, desambiguação de nome se houver colisão), **produtos/serviços** (com preço real ou PENDENTE), **FAQ** (derivadas diretamente da bateria F0 — perguntas já testadas, não inventadas), **projetos/portfólio**, **cases**. Campos obrigatórios: `slug`, `tipo`, `titulo`, `resumo` (responde a pergunta em 1–2 frases), `status`, `fonte`, `confianca`, `atualizado_em`, `relacionados`.

**Passos:**
1. Inventário de fontes reais: site do cliente, repositórios, materiais publicados, redes.
2. Extração assistida por IA em **lotes de 10**, com a regra dura "não invente; marque PENDENTE".
3. Revisão do cliente por lote (a revisão humana é o caminho crítico do programa inteiro — nunca esperar as 50; a F2 pode partir com 20 revisadas).
4. Validador automático: schema por tipo, slugs únicos, links `relacionados` existentes, nenhuma ficha pública com `confianca: PENDENTE`.

**Armadilhas vividas no cliente-zero:**
- **O modelo de negócio que o cliente descreve em conversa quase nunca está documentado** — e muda a interpretação de tudo. Perguntar cedo: "o que você vende de verdade? o que é gratuito? qual a arquitetura de ofertas?". No cliente-zero o modelo mudou 3× durante a escrita e exigiu voltar e corrigir tudo que já existia.
- FAQ construída sobre a bateria F0 rende as melhores fichas: são perguntas com demanda comprovada.
- Lacuna honesta > ficha inventada: se o cliente não tem a oferta que a pergunta pede, a ficha documenta a lacuna (e vira insumo de produto pra ele).

**Critérios de saída:** validador com 0 erros; ≥30–50 fichas revisadas pelo cliente; fichas cobrindo as ~20 páginas estratégicas do diagnóstico.

**Orçamento:** 1–2 semanas (o gargalo é a revisão do cliente, não a escrita).

**O que adaptar por cliente:** tipos de ficha (e-commerce ≠ consultoria ≠ educação); campo `visibilidade: interna` para dados que alimentam o agente mas não vão ao site.

---

## F2 — Publicação AEO/GEO

**Pergunta que responde:** "as IAs conseguem ler, entender e atribuir esse conteúdo ao domínio do cliente?"

**Arquitetura de referência (barata e resiliente):** gerador estático próprio (catálogo → HTML, sem dependências de build) → hospedagem estática gratuita (GitHub Pages) → **proxy/rewrite no domínio do cliente** (a autoridade acumula no domínio dele, não no da hospedagem) → sitemap enviado ao Google Search Console e Bing Webmaster Tools.

**Cada página gerada precisa ter:** resposta direta no 1º parágrafo; números, datas e listas (IAs citam especificidade); JSON-LD do tipo certo (`Organization`, `Course`, `FAQPage`, `Person`, `Article`…); `canonical` apontando pro domínio do cliente (a hospedagem vira espelho não-canônico); data de atualização visível; `api/*.json` estático (a "API" que o agente da F3 consome, sem servidor).

**robots.txt** do domínio liberando explicitamente os bots de IA: `OAI-SearchBot`, `ClaudeBot`/`Claude-SearchBot`, `PerplexityBot`, `Googlebot`, `Bingbot` (GPTBot/treino: decisão do cliente) + `Sitemap:` apontado.

**Armadilhas vividas no cliente-zero:**
- **Rewrites nativos de framework podem vazar o domínio da hospedagem** em redirects de barra final (no Next.js, `rewrites()` perdeu a trailing slash ao reconstruir `:path*` e expôs o `github.io` no `Location`). Solução: proxy em código usando o pathname original. **Testar com build local + curl comparando headers antes de publicar.**
- WAF/CDN do cliente pode desafiar bots de IA silenciosamente — por isso o teste com user-agent real é critério de saída, não opcional.
- Páginas atrás de proxy estático podem escapar do analytics do site principal — verificar rastreamento (ver F4).

**Critérios de saída (todos por comando, não por opinião):**
1. Páginas estratégicas respondem 200 no domínio do cliente com o conteúdo no HTML.
2. `curl -A "<UA de cada bot>"` retorna 200 + HTML.
3. JSON-LD validado programaticamente (≥3 tipos).
4. Sitemap acessível e enviado aos webmaster tools (com confirmação do cliente).
5. (janela de 2–6 semanas, não bloqueia F3) Search Console mostra ≥10 páginas indexadas.

**Orçamento:** 1 semana de build; 3 tentativas pro rewrite (falhou → fallback: subdomínio via CNAME); indexação corre em paralelo.

**O que adaptar por cliente:** onde está o domínio (Vercel/Cloudflare/WordPress muda o mecanismo do proxy); se o cliente já tem blog/CMS, o catálogo pode gerar pra dentro dele.

---

## F3 — Agente de chat comercial

**Pergunta que responde:** "quem chega ao site sai guiado, qualificado e — havendo interesse — vira lead?"

**Arquitetura de referência:** widget vanilla JS em **shadow DOM** (zero conflito com o site do cliente, zero dependências) → Edge Function serverless (a chave do modelo **nunca** toca o navegador) → LLM barato/rápido via OpenRouter (Claude Haiku no cliente-zero) → Postgres (`conversations`, `messages`, `leads`, `catalogo_fichas`, `injection_flags`).

**O cérebro: OpenRouter como padrão.** A Edge Function não chama a API do provedor do modelo diretamente — chama o OpenRouter, que expõe centenas de modelos atrás de **um único endpoint no formato OpenAI**. Por quê:

1. **Um formato só.** As APIs dos provedores divergem em detalhes traiçoeiros (autenticação, onde vai o system prompt, `max_tokens` obrigatório ou não, formato de tool use, eventos de streaming). O plano do cliente-zero tinha uma tabela inteira de diferenças OpenAI×Anthropic para não portar código às cegas — com o OpenRouter essa tabela deixa de ser problema.
2. **Trocar de modelo = trocar uma string.** Se o modelo ficar caro, lento ou for descontinuado, muda-se o nome na config — sem reescrever a integração. Útil nas iterações do gate de qualidade.
3. **Dashboard de custo por requisição/modelo** — é onde se mede o gate "custo médio por conversa ≤ teto acordado", sem instrumentação própria.
4. **Uma chave só, no servidor** (secret da Edge Function; o navegador nunca a vê).

Trade-off honesto: é um intermediário (pequena latência e margem sobre o preço do modelo) e recursos nativos exclusivos de um provedor podem não estar disponíveis. Para chat de site com modelo barato, a simplicidade vence.

**Decisões de desenho que transferem para qualquer cliente:**
- **Navegação e captura = tool use com schema fechado, nunca texto livre.** `navigate_to(rota)` com `rota` sendo enum das rotas reais do site — o modelo não consegue inventar URL porque o schema não deixa. `capture_lead(nome, email, interesse)` idem.
- **Catálogo como única fonte de verdade** do agente: a função injeta as fichas relevantes no contexto; instrução dura "se não está no catálogo, diga que não está registrado e ofereça contato". É isso que impede alucinação de preço/oferta.
- **Rate limit por IP+sessão + `max_tokens` curto + limite de turnos** — proteção de custo testada com flood real, não só escrita.
- **Heurística de detecção de injection** gravando flags no banco (alimenta o loop da F4).
- **v1 sem streaming é um fallback legítimo** — simples e correto > SSE não testado. Streaming vira v2.
- Banco **dedicado** se o cliente já tem infraestrutura com dados de produção ("não mexer no que já funciona").

**Gates antes do go-live (nenhum é opcional):**
1. Demo E2E real: tour navega ≥3 páginas, qualifica, captura lead **confirmado com query no banco** (não só no retorno da função).
2. **Suíte adversarial de ≥20 ataques contra a função publicada** (jailbreak, vazamento de prompt, abuso de tool, alucinação): 0 vazamentos, 0 ações fora do schema. Ler as respostas sinalizadas manualmente, não só contar passes.
3. DevTools: nenhuma chamada do navegador pra API do modelo.
4. Rate limit comprovado com flood (contar os 429).
5. Custo por conversa medido ≤ teto acordado.
6. **Cliente aprova 10 transcrições reais** antes de ligar.

**Orçamento:** 2–3 semanas; até 5 iterações de prompt no gate de qualidade (não passou → reduzir escopo pra tour roteirizado fixo).

**O que adaptar por cliente:** rotas do enum, personas do tour, tom de voz (limites hardcoded: idioma, nunca prometer resultado, nunca inventar preço, nunca pressionar após "não"), modelo/provedor conforme orçamento.

---

## F4 — Loop de medição e melhoria

**Pergunta que responde:** "está funcionando — e como provar?"

**Os 4 sinais a instalar (verificar TODOS antes de esperar tráfego — páginas novas costumam nascer sem analytics):**
1. Visitas do site principal (o analytics que o cliente já tem).
2. Visitas às páginas de conhecimento (atenção: proxy estático pode escapar do analytics principal — no cliente-zero usamos contadores `counterapi.dev`, públicos e sem login).
3. Uso do agente (as próprias tabelas do Postgres).
4. **AI Share of Voice: re-rodar a bateria v1 mensalmente** e calcular o delta contra o baseline — este é o número do case.

**Loop semanal (liga quando houver ≥20 conversas/semana; menos que isso → quinzenal):**
1. Classificador automático marca nas conversas: injection, frustração, perguntas sem ficha no catálogo, queda de qualidade.
2. Relatório de 10 linhas no topo: números da semana + 1 decisão pedida ao cliente.
3. Perguntas sem ficha → viram fichas novas (F1 é contínua).
4. Proposta de mudança de prompt = diff versionado + **golden set** (≥15 conversas de referência re-executadas) + aprovação do cliente. Sem aprovação, nada muda. Rollback = revert do commit.
5. Share of voice estagnado após 3 meses → a alavanca deixa de ser conteúdo e passa a ser **autoridade externa** (as fontes que as IAs citam no lugar do cliente, mapeadas na F0: YouTube, LinkedIn, parceiros, reviews).

**Orçamento:** setup 1 semana; operação ≤2 h/semana de atenção humana.

---

## Resumo comercial

| Fase | Duração | Entregável pro cliente |
|---|---|---|
| F0 | 2 dias | Relatório de baseline (o "antes" — frequentemente 0%) |
| F1 | 1–2 semanas | Catálogo de conhecimento versionado (~50 fichas revisadas) |
| F2 | 1 semana + 2–6 sem. de indexação | Páginas públicas no domínio dele, indexáveis por IA |
| F3 | 2–3 semanas | Agente de chat com gates de segurança comprovados |
| F4 | contínuo | Relatório mensal de share of voice + loop de melhoria |

**Pré-requisitos do cliente:** acesso ao domínio/site (pro proxy e robots), 1–2 h/semana de revisão durante a F1, decisor disponível pra aprovar transcrições do agente, e as credenciais na hora certa (não antes).

**O diferencial do método:** tudo é medido (antes/depois), nada é inventado (PENDENTE explícito), nenhum gate fecha sem comando executado — e o próprio processo do cliente-zero está documentado como prova (`case-inema/o-que-foi-feito-aiv.md` + `case-inema/relatorio-processo-completo-2026-07-10.md`).
