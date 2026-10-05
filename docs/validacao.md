# Validação documental — VazaApp 2.0

## Método e limite

Revisão da especificação e dos modelos com base no checklist do professor, conferência da cadeia RN ↔ RF/RNF ↔ UC ↔ fluxo ↔ critério/teste, leitura independente dos cenários centrais e renderização dos dois diagramas principais no Mermaid Live. A revisão verifica os artefatos de análise; não substitui validação com usuários nem testes de uma aplicação implementada.

## Checklist da especificação

| Critério | Evidência | Resultado documental |
|---|---|---|
| Identificação e nome orientado a objetivo | UC-001 a UC-035, nomes iguais no catálogo, especificações e DG-001 | Conferido |
| Ator principal e atores secundários | Papéis externos definidos; Serviço de e-mail somente na recuperação | Conferido |
| Objetivo e gatilho | Presentes nas 35 especificações | Conferido |
| Pré-condições e entradas | Informações e estado necessários separados das decisões do fluxo | Conferido |
| Principal com responsabilidades | Passos numerados indicam ator ou sistema e resultado da ação | Conferido |
| Alternativas e exceções | Cenários A/E com passo de origem e retorno ou encerramento | Conferido |
| Pós-condições de sucesso e falha | Resultado persistido ou preservação do estado anterior declarados | Conferido |
| Regras e requisitos definidos | 24 RN, 35 RF e 8 RNF com texto; referências identificadas | Conferido |
| Critérios e cenários previstos | 103 critérios de aceite e 103 cenários de teste com identificadores por UC | Conferido; testes de software não executados |
| Rastreabilidade | Matriz por UC/regra, cadeia central e mensagens do DG-002 | Conferido |
| Granularidade | Casos representam cadastro, consulta, manutenção, geração ou utilização; validação e persistência são passos | Conferido |
| Marcadores incompletos | Especificações sem “A definir”, “TODO” ou “FIXME” | Conferido nas especificações |

## Checklist dos diagramas

| Critério | Evidência | Resultado documental |
|---|---|---|
| Fronteira e atores externos | DG-001: fronteira VazaApp; quatro atores externos | Conferido |
| Todos os objetivos identificados | 35 nós UC no diagrama completo; agrupamentos não contam como casos | Conferido |
| Associações sem significado de sequência | Linhas sólidas sem seta ligam participação do ator | Conferido |
| Reutilização obrigatória | UC-018 `include` UC-017; login tratado como precondição | Conferido |
| Leitura por área | Seis vistas com os mesmos IDs da visão completa | Conferido |
| Sequência derivada de um cenário | DG-002 realiza UC-017 P1–P9 e explica a reutilização por UC-018 | Conferido |
| Participantes e responsabilidades | Sete participantes: ator, interface, aplicação, domínio e persistência | Conferido e justificado |
| Decisões e repetição | `alt` para validação, candidatos, idempotência e transação; `loop` para candidatos | Conferido |
| Alternativa/exceção | A1/A2 e E1/E2/E3 representados | Conferido |
| Sucesso e integridade | Sucesso só após confirmação; rollback sem sugestão parcial | Conferido |
| Sintaxe e compartilhamento | Dois diagramas principais renderizados no Mermaid Live; links incluem código correspondente à fonte | Conferido |

## Problemas identificados e corrigidos

| Problema | Por que importava | Correção |
|---|---|---|
| Apenas sete objetivos, sem manutenção/consulta de parte do escopo ampliado | A análise não cobria as funcionalidades adicionais autorizadas pela equipe | Catálogo de 35 objetivos, com especificação individual |
| RN/RF/RNF referenciados sem texto na versão anterior | Impedia conferir a origem de uma decisão | Requisitos definidos e matriz navegável |
| Senha atual exigida sem delimitar recuperação | Contradizia redefinição por token | RN-021 limita senha atual ao UC-005; RN-004 permite token e invalida todas as sessões |
| Avaliação chamada de “nota opcional” | Não correspondia à faixa obrigatória quando uma avaliação é enviada | A avaliação é opcional; nota de 1 a 5 é obrigatória nessa operação |
| Leitura do pedido antes da autorização na alternativa | Contradizia isolamento de dados | UC-018 encaminha referência/chave; leitura ocorre após autorização no caso incluído |
| Replay validado tarde demais | Catálogo alterado poderia impedir devolver uma operação já confirmada | P3 detecta operação confirmada; A2 salta P4–P7 e recupera o resultado em P8 |
| Geração inicial em pedido já concluído | Seleção sem exclusões poderia reencontrar modelo usado e terminar em conflito | UC-017 direto exige rascunho para chave nova; UC-018 realiza alternativas; estado rechecado no registro |
| Tags usadas para ordenar sem manutenção no catálogo | Responsabilidade não era sustentada pela análise | Tags opcionais cadastradas, consultadas e atualizadas; prioridade e desempate definidos em RN-010 |
| Garantia absoluta de ausência de dados pessoais no comentário | Texto livre pode conter informação inserida pelo autor | Minimização dos campos estruturados e orientação sobre comentário, sem promessa de anonimização integral |
| Ponto e vírgula literal em mensagem Mermaid | Separava instruções e causava erro de sintaxe | Mensagens ajustadas e sequência renderizada novamente |

## Verificações para a implementação futura

Os cenários de teste descritos nas especificações deverão ser executados quando houver software. Merecem atenção a recuperação com token expirado/usado, o acesso a recurso de outro proprietário, exclusões com dependências, a categoria desativada, as duas gerações concorrentes e a repetição da chave após mudança do catálogo. O desempenho e a compatibilidade continuam como metas a medir, não como resultados comprovados.

## Fontes

[ATIVIDADE03](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/atividades/1_bim/ATIVIDADE03_analise_projeto.md), exercícios 12–15, define checklist e rastreabilidade. [ATIVIDADE04](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/atividades/1_bim/ATIVIDADE04_DIAGRAMA_SEQUENCIA.MD), exercícios 17–24, define participantes, decisões, responsabilidades e sequência do cenário escolhido. A [aula de validação](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/aulas/SEMANA08/aula_validacao_casos_de_uso_diagrama_sequencia.pdf), p. 102, reúne os entregáveis mínimos.
