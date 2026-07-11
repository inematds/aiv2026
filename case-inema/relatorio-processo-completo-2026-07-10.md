# Relatório do processo — Programa AIV, sessão de 2026-07-10

## O que estive fazendo e qual o propósito

Numa única sessão longa, avancei o programa AIV (AI Visibility do INEMA) por 3 fases inteiras do plano (`doc/plano-aiv.md`) e comecei a preparar a quarta:

- **F1 — Catálogo**: terminei de escrever as fichas de conhecimento do INEMA (institucional, cursos, projetos, FAQ, serviços, cases).
- **F2 — Publicação**: transformei o catálogo em site público, indexável por IAs, servido em `inema.club/conhecimento/*`.
- **F3 — Agente**: construí o assistente de chat do site (guia + qualifica + captura lead).
- **F4 — Medição (preparação)**: identifiquei e corrigi uma lacuna de rastreamento antes de começar a medir de verdade.

**Propósito de fundo**: o programa existe porque o diagnóstico inicial (F0, sessão anterior) mostrou que o INEMA tem 0% de aparição em perguntas neutras de IA sobre o próprio nicho, e colisão de nome com um órgão ambiental. Tudo que fiz nesta sessão mira reverter isso — dar ao INEMA conteúdo estruturado que IAs conseguem citar, e depois um agente que converte quem chega.

## Recursos que me foram informados e que usei

- **Documentos do repo**: `doc/plano-aiv.md` (plano aprovado, com critérios de saída explícitos por fase), `doc/como_as_ias_publicas_conhecem_o_inema.md` (lista das 20 páginas estratégicas), `doc/catalogos_conhecimento_inema.md` (schema recomendado de fichas), `diagnostico/bateria-v1.md` (as 20 perguntas reais testadas na F0).
- **Fontes de fato**: código-fonte do portal, sites ao vivo (`inema.club`, `inema.pro`, `eventos.inema.pro`, `pay.inema.pro`) buscados ao vivo, GitHub oficial do INEMA.
- **Decisões de negócio que o Nei me deu em conversa** (não estavam documentadas antes): o INEMA não implementa sob contrato, é curadoria + comunidade; a marca INEMA.VIP foi descontinuada; a arquitetura final é club → pro → Imersão → Singular; o cérebro do agente é OpenRouter (não API Anthropic direta); o backend do agente deve ser um Supabase **separado** do que o portal já usa.
- **Credenciais que o Nei forneceu no momento em que precisei**: escopo `workflow` do `gh` (F2), login do `supabase` CLI (F3), chave da OpenRouter (F3).
- **Ferramentas**: fetch de páginas ao vivo pra pesquisa (nunca inventar dado), `agent-browser` pra testar o widget de verdade num navegador, Bash pra builds/testes/deploys, Supabase CLI, GitHub CLI.

## O que construí (concreto)

- **54 fichas** de catálogo em `catalogo/{institucional,cursos,projetos,faq,servicos,cases}/` (repo do programa).
- **Gerador de site** `scripts/gerar-site.mjs` — parser de front matter e conversor markdown→HTML sem dependências externas, JSON-LD por tipo, sitemap, `robots.txt`, `api/*.json`.
- **Site publicado**: repo público dedicado, GitHub Pages via Actions, 54 páginas no ar.
- **Proxy no portal**: `src/proxy.ts` + `trailingSlash: true` + `src/app/robots.ts`, servindo `inema.club/conhecimento/*` de forma transparente.
- **Agente de chat**: Supabase dedicado `inema-agente`, schema Postgres (`conversations`/`messages`/`leads`/`catalogo_fichas`/`injection_flags`), Edge Function `chat` (OpenRouter/Claude Haiku, tools `navigate_to`/`capture_lead`, rate limit, heurística de suspeita de injection), widget vanilla TS em shadow DOM.
- **Suíte de teste adversarial**: `scripts/suite-adversarial.mjs`, 22 ataques rodados contra a função real.
- **Rastreamento de visita** adicionado em 3 frentes que estavam sem nenhum dado: `/conhecimento/*`, `eventos.inema.pro`, `pay.inema.pro` (mesmo padrão `counterapi.dev` que `inema.pro` já usava).
- Fix de um bug real no portal (cards de curso/projeto redirecionando pro lugar errado).

## Como cheguei a esse resultado

1. **Pesquisa de fonte real antes de escrever qualquer coisa.** Toda ficha, todo preço, toda URL veio de código-fonte lido, site ao vivo buscado, ou conversa direta com o Nei — nunca de suposição. Quando a fonte não confirmava algo, marquei `PENDENTE` em vez de inventar (ex.: preço do INEMA Singular, tagline final).
2. **Construir em camadas, testando cada uma antes de empilhar a próxima.** Escrevi o gerador → testei localmente (links quebrados, JSON-LD válido) → só então publiquei. Escrevi o proxy → testei com `next build && next start` local → só então publiquei. Escrevi a Edge Function → testei contra a API real → só então considerei pronta.
3. **Parar e perguntar quando faltou credencial ou informação, em vez de simular.** Aconteceu 3 vezes (escopo do `gh`, acesso ao Supabase certo, chave da OpenRouter) — em nenhuma inventei um resultado ou contornei com gambiarra.
4. **Reagir a correções de negócio em tempo real.** Quando o Nei trouxe informação nova (ex.: "o INEMA não implementa pra cliente", depois "a marca VIP vai virar Imersão + Singular"), voltei e corrigi tudo que já tinha escrito baseado na suposição antiga, em vez de deixar inconsistente.

## Quais decisões tomei

**Decisões técnicas minhas, com a razão:**
- **v1 do agente sem streaming** — o plano previa esse fallback explicitamente; não dava pra testar o parsing de SSE às cegas sem credenciais ainda, então preferi entregar uma versão simples e correta a uma streaming não testada.
- **Proxy via `src/proxy.ts` em vez de `rewrites()` do `next.config`** — descobri (testando local) que o rewrite padrão perde a barra final ao reconstruir `:path*` e vaza o domínio real da hospedagem (`*.github.io`) no redirect. Troquei pra uma abordagem que usa o pathname original.
- **`trailingSlash: true` global no portal** — necessário pro proxy funcionar; avaliei o efeito colateral (rotas nativas como `/stats` passam a redirecionar pra versão com barra) como aceitável e testei antes de publicar.
- **Heurística de suspeita de prompt injection gravada em `injection_flags`** — o plano listava isso como correção de risco; implementei mesmo sem ter sido pedido de novo, porque estava no escopo original.
- **Não commitar `conhecimento/` no repo `aiv`** — é saída gerada, a fonte de verdade é `catalogo/`; segue a convenção já documentada no `CLAUDE.md` do projeto.

**Decisões de negócio do Nei, que eu apliquei:**
- Repo `conhecimento` separado do `aiv` (resolve Pages × privado).
- Supabase do agente separado do Supabase do portal ("no outro temos mais coisas").
- OpenRouter em vez de API Anthropic direta.
- Arquitetura final de 4 níveis (club/pro/Imersão/Singular) substituindo o VIP.

## O que verifiquei

- Build do Next.js a cada mudança de código (várias vezes ao longo da sessão).
- Links internos de todas as 54 fichas (checagem automática de 404 embutida no gerador).
- JSON-LD válido em todas as páginas geradas (parse programático, não só visual).
- 5 user-agents de bots de IA reais (`OAI-SearchBot`, `ClaudeBot`, `PerplexityBot`, `Googlebot`, `Bingbot`) retornando 200.
- O proxy/rewrite testado localmente (`next build && next start`) antes de publicar — foi assim que achei o bug da barra final.
- Widget testado visualmente de verdade num navegador (`agent-browser`): screenshot fechado, screenshot aberto, envio de mensagem, erro de rede tratado sem quebrar a página.
- Edge Function testada contra a API real (não simulada): tool use (`navigate_to`, `capture_lead`) funcionando, conversa multi-turn com `session_token` persistindo, rate limit confirmado com flood de 15 requisições (12 OK, 3 seguintes 429), lead capturado e **confirmado direto no Postgres** via query (depois removido, era teste).
- Suíte adversarial de 22 ataques rodada contra a função publicada; onde a heurística automática sinalizou algo, revisei manualmente o texto da resposta antes de aceitar como "ok" ou "problema real".

## Quais riscos considerei

- **Vazamento de segredo**: uma chave Supabase foi parcialmente exposta na minha própria saída de terminal por falha de redação. Reconheci o erro, avaliei o risco como baixo (não foi commitada nem publicada, projeto sem tráfego), e ofereci rotação manual como opção.
- **Vazamento do domínio real do backend**: o bug do proxy fazia o navegador ver o domínio da hospedagem (`*.github.io`) num redirect em vez de ficar transparente em `inema.club` — corrigido antes de publicar.
- **Prompt injection / jailbreak do agente**: testado com 22 ataques reais antes de considerar a F3 pronta, não assumido como "resolvido só porque o system prompt pede".
- **Alucinação de preço/oferta/URL pelo agente**: mitigado com tools de schema fechado (enum de rotas reais) + regra explícita de "só o catálogo" no prompt — testado, não só declarado.
- **Rate limit / abuso de custo**: implementado e testado com flood real, não só código escrito.
- **Publicar conteúdo de fichas ainda não revisadas pelo Nei**: mitigado filtrando as seções `PENDENTE` do HTML público — o catálogo interno guarda a incerteza, o site público não.
- **Mexer em produção sem necessidade**: nunca chequei status de deploy do Vercel nem mexi no dashboard, por instrução direta do Nei — só `git push` e sigo.
- **Misturar dados do agente com o banco de produção existente**: resolvido criando projeto Supabase novo e dedicado, a pedido do Nei.

## Quais etapas foram essenciais

- **Pesquisa de fonte real antes de escrever cada ficha** — sem isso, o catálogo teria conteúdo inventado, o que teria contaminado tudo que veio depois (site público, contexto do agente).
- **Testar localmente antes de publicar em toda mudança de infraestrutura** — pegou 2 bugs reais que só apareceriam em produção se eu não tivesse testado antes (link do portal antigo, barra final no proxy novo).
- **Rodar a suíte adversarial contra a função já publicada**, não como exercício teórico — sem isso, eu estaria apenas confiando que o prompt "deveria" funcionar.
- **Parar pra perguntar quando faltou credencial**, em vez de assumir ou simular um resultado.

## Como provei que a resposta estava correta

Não declarei nenhum critério de saída como cumprido sem rodar um comando real e mostrar o resultado real:

- JSON-LD: parseado programaticamente em todas as páginas, não só "parece certo visualmente".
- Bots de IA: `curl` de verdade com cada user-agent, retorno de código HTTP real.
- Proxy: `next build && next start` local, `curl` real, comparação de headers antes/depois do fix.
- Rate limit: flood real de 15 requisições, códigos de resposta reais (12×200, 3×429).
- Lead capturado: query direta no Postgres via API REST do Supabase, não só "a função retornou `lead_captured: true`".
- Suíte adversarial: 22 respostas reais lidas e revisadas manualmente onde a heurística sinalizou algo, não só contagem de "quantos passaram".

## O que tornou essa entrega melhor

- **Disciplina de não inventar dado de negócio.** Toda vez que uma informação não estava confirmada (preço do Singular, nome final da oferta, se APIs-para-iniciantes existe como curso), registrei como `PENDENTE` em vez de preencher com algo plausível.
- **Reagir a correções em vez de deixar inconsistência acumular.** Quando o modelo de negócio mudou 3 vezes ao longo da sessão (curadoria-não-implementação → transição do VIP → arquitetura final de 4 níveis), voltei e corrigi tudo que já existia baseado na versão anterior, em vez de deixar fichas contraditórias entre si.
- **Identificar lacunas que ninguém pediu explicitamente.** A ausência total de rastreamento em `/conhecimento/`, `eventos.inema.pro` e `pay.inema.pro` não foi apontada pelo Nei — eu percebi ao verificar como o Plausible do portal funciona (não executa em páginas via proxy estático) e corrigi antes que isso virasse um período inteiro de dados perdidos.
- **Transparência sobre um erro real** (a chave exposta) em vez de omitir.
