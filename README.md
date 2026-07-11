# AIV 2026 — AI Visibility como serviço (AEO/GEO)

Material do serviço de **AI Visibility**: tornar uma empresa encontrável, citável e recomendável por IAs públicas (ChatGPT, Claude, Gemini, Perplexity) e instalar um agente de chat comercial no site dela. Este repo separa o **método replicável** (para novos clientes) da **prova** (o cliente-zero, INEMA, executado em 2026-07).

## Estrutura

```
aiv2026/
├── playbook-aeo-geo.md      # O MÉTODO — 5 fases (F0–F4) abstraídas para qualquer cliente:
│                            # passos, critérios de saída, orçamentos, armadilhas reais,
│                            # o que adaptar por cliente, resumo comercial.
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

## Como usar com um cliente novo

1. Ler `playbook-aeo-geo.md` de ponta a ponta — é o roteiro de execução.
2. Usar `case-inema/plano-aiv.md` como modelo do plano detalhado a adaptar (as tabelas de modos de falha transferem quase inteiras).
3. Usar o baseline do case (0% de aparição) como material de venda: o diagnóstico F0 de 2 dias produz o mesmo "antes" para o cliente novo.
4. O `case-inema/o-que-foi-feito-aiv.md` mostra o que o entregável final concreto parece (repos, scripts, comandos, gates cumpridos).

## Resultado do cliente-zero (resumo)

- **Antes (F0):** 0% de aparição do INEMA em 52 medições de prompts neutros; colisão de nome com órgão público homônimo.
- **Construído:** 54 fichas de catálogo → site indexável em `inema.club/conhecimento/*` (JSON-LD, sitemap, bots de IA liberados e testados) → agente de chat com gates de segurança comprovados (22 ataques, 0 vazamentos) → 4 sinais de medição instalados.
- **Depois:** re-medição mensal em andamento (janela de indexação de 2–6 semanas a partir de 2026-07-10).

*Docs do cliente-zero copiados do repo privado do programa em 2026-07-10.*
