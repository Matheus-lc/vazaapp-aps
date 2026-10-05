# Refatoração da análise — VazaApp 2.0

## Origem e escopo

A base foi o [repositório original da equipe](https://github.com/isabvenancio/atividade_aps_3), composto por README, sete especificações e links de um diagrama no diagrams.net. O contexto principal permanece: gerar sugestões de desculpas personalizadas a partir de situação, contexto, destinatário e proximidade.

A equipe autorizou ampliar a análise para cerca de 35 casos de uso, adotar Mermaid para os dois tipos de diagrama e publicar a nova versão em um repositório separado. O número de casos é uma decisão de escopo da equipe; não é uma exigência atribuída ao professor.

## Correspondência com a versão anterior

| Caso anterior | Realização na versão 2.0 | Mudança |
|---|---|---|
| UC-01 — Cadastrar usuário | UC-001 — Cadastrar conta | Acrescenta autenticação, recuperação e manutenção de conta como objetivos próprios. |
| UC-02 — Manter perfil de contexto | UC-007 — Consultar perfil de contexto; UC-008 — Atualizar perfil de contexto | Separa consulta e manutenção, mantendo dados opcionais e sua finalidade. |
| UC-03 — Criar pedido de desculpa | UC-013 — Criar pedido de desculpa | Cria um rascunho com dados disponíveis e resultado persistido definido. |
| UC-04 — Informar situação do pedido | Fluxos de UC-013 e UC-015 — Atualizar pedido em rascunho | Situação, categoria, destinatário e contexto são informações do pedido, reunidas no objetivo de criar ou completar um rascunho. |
| UC-05 — Gerar sugestão personalizada | UC-017 — Gerar sugestão personalizada; DG-002 | Define seleção por catálogo, personalização, atomicidade, idempotência e falhas. |
| UC-06 — Solicitar nova sugestão | UC-018 — Solicitar nova sugestão | Reutiliza a geração por `include`, excluindo modelos anteriores do mesmo pedido. |
| UC-07 — Fornecer feedback | UC-024 — Avaliar sugestão; UC-025 — Atualizar avaliação; UC-026 — Excluir avaliação | Define nota, comentário, cardinalidade e manutenção pelo proprietário. |

Os novos objetivos abrangem conta/acesso, destinatários reutilizáveis, consulta e exclusão de pedidos, histórico, cópia, favoritos e administração do catálogo. O destinatário não precisa possuir conta no VazaApp nem interage com o sistema para receber uma sugestão; por isso, não aparece como ator.

## Decisões da nova modelagem

1. **Objetivos completos:** consultar, cadastrar, atualizar, excluir e desativar são separados quando produzem resultados distintos. Validação, digitação de campos, confirmação e escrita no banco permanecem passos do fluxo.
2. **Fotografia do pedido:** os dados do destinatário e do contexto são copiados para o pedido. Atualizações posteriores no cadastro não alteram uma sugestão histórica. O rascunho pode ser corrigido; depois da primeira geração o contexto fica preservado.
3. **Geração por catálogo:** somente categorias e modelos ativos e compatíveis são usados. Não foi acrescentado um serviço de IA externo.
4. **Integridade:** sucesso exige confirmação da sugestão, vínculo, histórico e estado em uma única transação. A mesma chave de operação recupera a sugestão já confirmada.
5. **Requisitos definidos:** as referências RN/RF/RNF agora possuem texto verificável. A numeração foi reorganizada para esta versão; IDs antigos não devem ser interpretados automaticamente com os novos textos.
6. **Mermaid:** DG-001 usa a representação `flowchart` demonstrada pelo professor; DG-002 usa `sequenceDiagram`. As fontes `.mmd`, os blocos Markdown e os links de visualização/edição são entregues juntos.
7. **Sequência central:** a realização de UC-017 representa principal, alternativas e exceções; as responsabilidades dos participantes são justificadas a partir da análise.
8. **Novo repositório:** esta entrega é uma versão autônoma da documentação e preserva a identificação acadêmica da equipe e a referência à origem.

## Base didática

- [ATIVIDADE02](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/atividades/1_bim/ATIVIDADE02.md): organização e identificação de regras, requisitos e artefatos.
- [ATIVIDADE03](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/atividades/1_bim/ATIVIDADE03_analise_projeto.md): objetivos, gatilhos, fluxos, revisão e rastreabilidade.
- [ATIVIDADE04](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/atividades/1_bim/ATIVIDADE04_DIAGRAMA_SEQUENCIA.MD): cenário central, participantes, mensagens, decisões e responsabilidades.
- [SEMANA05.pdf](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/aulas/SEMANA05/SEMANA05.pdf), p. 14–16, 43 e 50: granularidade por objetivo e representação Mermaid de casos de uso.
- [Casos de uso 2.0](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/aulas/SEMANA05/casos-uso_2.0.pdf) e [exemplo](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/aulas/SEMANA05/caso-uso-exemplo.pdf): atores, fronteira, casos e relações.
- [Aula de validação e sequência](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/aulas/SEMANA08/aula_validacao_casos_de_uso_diagrama_sequencia.pdf), p. 99–102: checklist, UC revisado, sequência com alternativa/exceção e justificativa.

Os exemplos da disciplina orientam a forma de modelar. As regras específicas de um sistema usado como exemplo não foram transferidas para o domínio do VazaApp.
