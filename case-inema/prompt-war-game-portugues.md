# Prompt de War Game

Planeje a construção completa de [PROJETO], mas faça isso como um **war game**. Não assuma que será possível ir do ponto A ao ponto B seguindo apenas o caminho feliz.

## O que estamos construindo

Um agente de chat que fica no meu site, com abas flutuantes de ações rápidas acima do widget, por exemplo: **"Me guie pelo site"**.

Ao escolher uma aba, o agente inicia um tour guiado que navega por páginas reais e subpáginas, narrando o que está acontecendo e fazendo perguntas ao visitante para avaliar se existe encaixe com o que oferecemos.

Quando surgirem sinais de compra, o agente introduz naturalmente o que oferecemos, minhas credenciais e nossos [workshops/serviços], e então captura o lead.

Ele se adapta conforme a conversa evolui. Nada deve parecer genérico ou engessado.

O cérebro será o [Claude Haiku], chamado a partir de [Supabase Edge Functions], com todas as conversas armazenadas em [Supabase Postgres]. O navegador nunca acessa diretamente a API do modelo.

## Como planejar

- Divida a construção em fases. Para cada fase, entregue:
  - objetivo;
  - suposição otimista;
  - suposição pessimista;
  - modos de falha mais prováveis, com correções;
  - critérios de saída verificáveis;
  - orçamento de tomada ou tentativa.

- Adicione uma tabela mestre com todos os cenários de falha que você conseguir prever, incluindo:
  - infraestrutura;
  - particularidades da API do modelo;
  - frontend;
  - qualidade da conversa;
  - segurança;
  - loop de aprendizado.

  Para cada cenário, inclua como detectar o problema e como recuperar.

- Aponte exatamente onde a API do [Claude] é diferente dos hábitos comuns de uso da [OpenAI], para que nada seja portado de forma cega.

- Desenhe um loop semanal de autoavaliação que:
  1. analise todas as conversas em busca de prompt injection;
  2. adapte o estilo de fala do agente aos visitantes reais, dentro de limites rígidos.

  O agente deve melhorar com o tempo sem nunca perder o profissionalismo central.

  Nenhuma mudança deve ir para produção sem uma avaliação de regressão e minha aprovação explícita.

## Entregáveis

1. Um plano em Markdown escrito para um agente começando do zero, sem nenhum contexto anterior. O plano deve incluir:
   - preâmbulo;
   - caminho crítico;
   - stack;
   - estrutura;
   - metodologia.

2. Uma versão visual em HTML do mesmo plano, com diagramas gerados para:
   - arquitetura;
   - cada fase do projeto.

Assuma que você encontrará barreiras, não importa o quão inteligente você pense que é. Planeje essas barreiras antes de escrever uma única linha de código.
