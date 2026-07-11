# O que foi feito — Programa AIV (registro completo + runbook)

*2026-07-10 · Este documento registra TUDO que existe construído no programa AIV, onde cada coisa vive, como operar cada peça, e o que ainda está em aberto. A narrativa de decisões e riscos da sessão está em `relatorio-processo-completo-2026-07-10.md`; o método abstraído para outros clientes está em `../playbook-aeo-geo.md`.*

---

## Visão geral

O AIV tornou o INEMA encontrável/citável por IAs públicas em 5 fases. Estado atual:

| Fase | Estado | Entregável central |
|---|---|---|
| F0 Diagnóstico | ✅ Concluída | Baseline: 0% de aparição em 52 medições |
| F1 Catálogo | ✅ Escrita (54 fichas, revisão do Nei pendente) | `catalogo/` — fonte única MD+YAML |
| F2 Publicação | ✅ No ar | `inema.club/conhecimento/*` (54 páginas + JSON-LD + sitemap) |
| F3 Agente | ✅ Construído e testado | Widget de chat + Edge Function no Supabase dedicado |
| F4 Loop | 🟡 Infra de medição pronta, loop aguarda tráfego | 4 sinais de medição instalados |

---

## F0 — Diagnóstico (baseline)

**O que existe:**
- `diagnostico/bateria-v1.md` — 20 prompts neutros congelados (NUNCA alterar; adições viram bateria v2 medida em paralelo).
- `diagnostico/medicoes.csv` — 52 medições registradas (data, ferramenta, prompt, execução, INEMA citado?, concorrentes, fontes, erros).
- `diagnostico/baseline-2026-07.md` — resultado: **0% de aparição do INEMA** em prompts neutros; colisão de nome com INEMA-BA (órgão ambiental); bio do Nei desatualizada nas fontes que as IAs usam.
- Scripts de medição: `scripts/f0-gemini.mjs` (tem resume — retoma de onde parou quando a cota free renova), `scripts/f0-codex.mjs`, `scripts/f0-consolidar.mjs` (consolida os raw JSONL em CSV). Raw em `diagnostico/raw/*.jsonl`.

**Como remedir (mensal, na F4):**
```bash
cd <repo-do-programa>
# requer GEMINI_API_KEY no ambiente
node scripts/f0-gemini.mjs        # roda a bateria v1 no Gemini
node scripts/f0-codex.mjs         # idem via Codex/ChatGPT
node scripts/f0-consolidar.mjs    # consolida raw/*.jsonl → medicoes.csv
# comparar % de aparição contra baseline-2026-07.md e registrar delta
```

**Pendências F0 (não bloqueiam nada):** completar Gemini quando a cota renovar; medir ChatGPT/Perplexity se surgirem chaves.

---

## F1 — Catálogo (54 fichas)

**O que existe:** `catalogo/{institucional,cursos,projetos,faq,servicos,cases}/` — 10 institucionais, 10 cursos, 16 projetos, 11 FAQ, 5 serviços, 1 case (o próprio AIV). Cada ficha é MD com front matter YAML (`slug`, `tipo`, `titulo`, `resumo`, `status`, `fonte`, `confianca`, `atualizado_em`, `relacionados`).

**Regras de ouro:**
- `catalogo/` é a **fonte única**. Nunca inventar dado: tudo vem de fonte real (código do portal, sites ao vivo, GitHub, conversa com o Nei). Sem confirmação → seção `PENDENTE` na ficha (o gerador filtra PENDENTE do HTML público).
- Arquitetura de níveis refletida nas fichas: **INEMA.club** (gratuito) → **INEMA.pro** (R$42/mês ou R$327/ano) → **INEMA Imersão** (ex.: R$2.997) → **INEMA Singular** (mentoria/consultoria, sob negociação). Marca "VIP" descontinuada. Modelo de negócio: o INEMA não implementa para cliente — é curadoria + comunidade; projetos são open source, nascidos de lives/imersões.

**Como adicionar/editar uma ficha:**
1. Criar/editar o `.md` na pasta do tipo certo, com front matter completo e fonte real.
2. Regenerar e publicar o site (ver F2 abaixo).
3. Se o agente de chat deve conhecer a mudança: rodar o sync do catálogo pro Supabase (ver F3 abaixo).

**Pendências F1:** revisão linha a linha do Nei (fichas estão `status: rascunho`); tagline final do Singular; nome da variante anual do Singular; formato definitivo do Imersão.

---

## F2 — Publicação (site público)

**O que existe:**
- `scripts/gerar-site.mjs` — catálogo → HTML semântico + JSON-LD por tipo + sitemap.xml + robots.txt + `api/*.json`. Sem dependências externas. Checagem de 404 interna embutida. Saída em `conhecimento/` (**gitignored no `aiv` — nunca commitar aqui**).
- Repo público dedicado com GitHub Pages via Actions (espelho não-canônico; o canônico é `inema.club/conhecimento/*`).
- Proxy no portal: `src/proxy.ts` serve `inema.club/conhecimento/*` de forma transparente. **Não usar `rewrites()` do next.config** — perde a barra final e vaza o domínio do GitHub no redirect (bug real encontrado em teste local). `trailingSlash: true` no next.config é necessário. `src/app/robots.ts` libera os bots de IA e aponta o sitemap.
- Sitemap enviado ao Google Search Console e Bing Webmaster Tools (feito pelo Nei em 2026-07-10).

**Como republicar após mudar o catálogo:**
```bash
cd <repo-do-programa>
node scripts/gerar-site.mjs                          # gera conhecimento/
rsync -a --delete --exclude .git --exclude .github \
  conhecimento/ <clone-do-repo-do-site>/             # preservar workflows do Pages
cd <clone-do-repo-do-site>
git add -A && git commit -m "Atualiza catálogo" && git push
# deploy é automático (Actions → Pages). NÃO monitorar Vercel/Pages — termina no push.
```

**Verificação rápida (os critérios de saída da F2):**
```bash
curl -s -o /dev/null -w "%{http_code}" https://inema.club/conhecimento/
curl -s -o /dev/null -w "%{http_code}" -A "ClaudeBot" https://inema.club/conhecimento/sitemap.xml
```

**Pendências F2:** Search Console mostrar ≥10 páginas indexadas (janela de até 6 semanas — checar na F4, a partir de ~2026-08-20).

---

## F3 — Agente de chat

**O que existe:**
- Projeto Supabase **dedicado** (região sa-east-1) — separado do Supabase de visitas do portal, por decisão do Nei.
- Código no repo do **portal**: `supabase/migrations/` (schema: `conversations`, `messages`, `leads`, `catalogo_fichas`, `injection_flags`), `supabase/functions/chat/` (Edge Function Deno), `src/components/AgenteChat/` (widget vanilla em shadow DOM, zero deps).
- Cérebro: **OpenRouter → Claude Haiku** (decisão do Nei; formato OpenAI normalizado, dispensa a tabela de diferenças da API Anthropic do plano). Chave da OpenRouter é secret da Edge Function — o navegador nunca vê.
- Tools de schema fechado: `navigate_to` (enum de rotas reais — o modelo não inventa URL) e `capture_lead`. Rate limit por IP+sessão (12 req OK, 429 no 13º — testado com flood real). Heurística de suspeita de injection gravada em `injection_flags`.
- v1 **sem streaming** (fallback previsto no plano; SSE fica pra v2).
- Suíte adversarial: `scripts/suite-adversarial.mjs` (repo do portal), 22 ataques contra a função publicada — 0 vazamentos, 0 ações fora do schema. Transcrições em `scripts/suite-adversarial-resultado.json`.

**Como sincronizar o catálogo com o agente (após mudar fichas):**
```bash
cd <repo-do-programa>
node scripts/sync-catalogo-supabase.mjs   # catalogo/ → tabela catalogo_fichas
```

**Como re-rodar a suíte adversarial (após qualquer mudança de prompt):**
```bash
cd <repo-do-portal>
node scripts/suite-adversarial.mjs        # gate: 0 vazamentos, 0 ações fora do schema
```

**Pendências F3 (ação do Nei):**
1. ~~Env vars `NEXT_PUBLIC_SUPABASE_CHAT_URL` e `NEXT_PUBLIC_SUPABASE_CHAT_ANON_KEY` no Vercel~~ — **feito** (commit `f90aca0`).
2. Aprovar 10 transcrições de conversa antes do go-live oficial (critério de saída do plano).
3. Medir custo médio por conversa no dashboard da OpenRouter após tráfego real.
4. (Opcional) Rotacionar o JWT secret do Supabase — uma chave foi parcialmente exposta em saída de terminal durante o setup; risco baixo, não commitada nem publicada.

---

## F4 — Medição (infra pronta, loop aguarda tráfego)

**Decisão do Nei:** esperar tráfego real antes de ligar o loop semanal. Antes de esperar, foi coberta a lacuna de rastreamento — `/conhecimento/*`, `eventos.inema.pro` e `pay.inema.pro` não tinham NENHUM contador (o Plausible do portal não executa em páginas servidas via proxy estático).

**Os 4 sinais de medição instalados:**
1. **Visitas gerais do portal** — Plausible + Supabase de visitas (já existia; Plausible exige login do Nei).
2. **Visitas conhecimento/eventos/pay** — contadores `counterapi.dev` (namespaces `inema-conhecimento`, `inema-eventos`, `inema-pay`; o do portal é `inema-pro`). Leitura pública, sem login:
   ```bash
   curl -s https://api.counterapi.dev/v1/inema-conhecimento/visitas
   ```
3. **Uso do agente** — tabelas `conversations`/`messages`/`leads` no Supabase `inema-agente`.
4. **AI Share of Voice** — remedir a bateria v1 mensalmente (comandos na seção F0 acima) e comparar com o baseline.

**O que falta pra F4 de verdade (quando houver tráfego):** loop semanal do plano — classificador de conversas (injection/frustração/perguntas sem ficha), relatório `loop/relatorio-YYYY-WW.md`, golden set de regressão, proposta de mudança de prompt só com aprovação do Nei.

---

## Mapa de repositórios e credenciais

| Repo | Visibilidade | Papel |
|---|---|---|
| repo do programa (AIV) | privado | planejamento, catálogo-fonte, scripts, diagnóstico |
| repo do site de conhecimento | público | site gerado + GitHub Pages |
| repo do portal | — | inema.club (Next.js/Vercel): proxy, robots, widget, Edge Function |

- Autor/committer de todo commit: a conta oficial do projeto.
- Publicar = commit + push. Nunca tocar no dashboard do Vercel.
- Chaves (Gemini pros scripts F0, OpenRouter pro agente) vivem em variáveis de ambiente e secrets da Edge Function — nunca no repo.
- Barreira que estourar orçamento de fase → registrar em `doc/decisoes.md` + avisar o Nei.
