# DG-002 — Sequência de geração de sugestão personalizada

**Estado:** revisado na documentação; não implementado.

Este diagrama realiza o **UC-017 — Gerar sugestão personalizada**. Representa seu fluxo principal e os cenários A1/A2 e E1/E2/E3. O UC-018 — Solicitar nova sugestão reutiliza a mesma política e a mesma fotografia do pedido, acrescentando os modelos anteriores à lista de exclusão. Há uma única sequência central; ela não pretende representar as interações dos demais 33 casos de uso.

O [código Mermaid editável](../diagramas/DG-002-sequencia-gerar-sugestao.mmd) usa `sequenceDiagram`, sete participantes, mensagens com retorno, fragmentos `alt` e uma repetição `loop`. As alternativas aparecem nos pontos em que alteram o resultado do cenário. O resultado de uma geração somente é apresentado como concluído depois da confirmação da persistência.

## Cenários e limites

O usuário solicita a geração para um pedido próprio. O sistema autentica a sessão, verifica a propriedade, recupera a fotografia do pedido, valida os dados mínimos e a categoria, consulta modelos ativos compatíveis, seleciona e personaliza um modelo, registra a operação de forma atômica e apresenta o resultado confirmado.

A fotografia inclui os dados do destinatário e o contexto copiados anteriormente para o pedido. A geração não substitui esses dados por informações atuais do perfil ou do destinatário. Essa preservação permite que um pedido continue coerente depois de mudanças ou exclusão do destinatário. A personalização usa apenas os parâmetros permitidos e os dados fornecidos; um contexto ausente é omitido.

| Cenário | Origem | Comportamento e resultado |
| --- | --- | --- |
| Principal | P1–P9 | Uma sugestão, seu vínculo, um evento de histórico e o estado `com_sugestao` são confirmados; o texto é apresentado. |
| A1 — Sem candidato compatível | P6 | Informar indisponibilidade e encerrar sem nova sugestão ou evento de histórico; preservar os registros existentes. |
| A2 — Repetição de operação confirmada | P8; detecção antecipada em P3 | Recuperar a sugestão da mesma chave e apresentá-la sem criar outra sugestão ou histórico. Se detectada em P3, saltar P4–P7. |
| E1 — Pedido inapto | P4 | Informar os dados pendentes, a categoria inativa ou o estado incompatível; encerrar sem registrar uma sugestão. O usuário deve regularizar o pedido pelas operações autorizadas. |
| E2 — Sem acesso | P2 | Negar a operação quando a sessão for inválida ou o pedido não pertencer ao usuário; não apresentar seus dados. |
| E3 — Registro não concluído | P8 | Desfazer a transação, manter o estado anterior e informar a falha; permitir nova tentativa com a mesma chave. |

**Idempotência e concorrência.** A chave de operação acompanha a solicitação desde a interface. O retorno de P3 inclui o estado de uma operação já confirmada para que a repetição não seja recusada por uma categoria posteriormente desativada ou pela ausência de novos modelos. P8 também verifica a chave dentro da transação: se outra execução confirmou a mesma operação nesse intervalo, devolve o resultado existente. Uma chave diferente não permite registrar novamente um modelo já usado no pedido; um conflito não resolvido impede a confirmação, desfaz a transação e segue E3. Esses controles são decisões de projeto derivadas de RN-024, sem constituir novos objetivos do usuário. Uma chave nova de geração inicial exige pedido em rascunho; pedidos já gerados usam UC-018. O modo deriva do caso acionado, sem escolha arbitrária pelo usuário. A transação confere novamente o estado para tratar mudança concorrente conforme E3.

**Seleção.** O catálogo restringe os candidatos pela categoria ativa, situação ativa do modelo, tipo do destinatário e proximidade. Depois de autorizar o acesso, P3 recupera os modelos anteriormente usados junto com a fotografia. A geração inicial usa exclusões vazias; na realização de UC-018, os modelos anteriores alimentam a lista de exclusão. A política prioriza o maior número de tags presentes no contexto informado, sem distinção de maiúsculas/minúsculas, e usa o identificador crescente para desempatar. Sem contexto ou correspondência de tags, o identificador dá uma ordem estável. O `loop` representa essa avaliação de cada candidato; não cria uma sequência artificial de ações de interface. O diagrama não fixa pesos ou uma fórmula de pontuação que não tenham sido definidos no domínio.

## Responsabilidades dos participantes

| Participante | Papel | Responsabilidade e origem |
| --- | --- | --- |
| Usuário | Ator | Solicitar uma sugestão para um pedido próprio e receber o resultado; UC-017 P1/P9. |
| TelaSugestao | Interface/boundary | Receber a solicitação, encaminhar pedido e chave, apresentar confirmação, pendências ou erro; não escolher modelos nem inventar conteúdo. |
| GeracaoController | Aplicação | Validar a sessão, receber a operação, encaminhar a execução e traduzir o resultado para a interface; P2, RN-002 e RNF-001. |
| GeracaoService | Coordenação do caso de uso | Coordenar acesso, fotografia, validação, catálogo, política, transação e resposta; P2–P9. Não concentra a política de personalização nem a escrita transacional. |
| PoliticaGeracao | Domínio | Validar mínimos, avaliar e ordenar modelos, aplicar exclusões e personalizar apenas com os dados permitidos; P4/P6/P7, RN-008/RN-010/RN-011/RN-013. |
| CatalogoRepository | Persistência do catálogo | Consultar a situação da categoria e recuperar modelos ativos compatíveis; P4/P5, RN-008/RN-010. |
| PedidoRepository | Persistência e unidade de trabalho | Recuperar a fotografia somente para o proprietário, localizar uma operação confirmada e coordenar a escrita atômica de sugestão, vínculo, histórico e estado; P2/P3/P8, RN-003/RN-012/RN-024. |

`PedidoRepository` agrupa a persistência do agregado pedido e a unidade de trabalho para manter o diagrama legível com sete participantes. Na implementação, essa responsabilidade pode ser distribuída entre repositórios de pedido/sugestão/histórico e um componente de transação, desde que todos participem da mesma transação. A elegibilidade do modelo e do catálogo deve ser conferida de forma consistente no registro. O agrupamento não transforma o banco de dados em ator nem dá à interface acesso direto à persistência.

## Rastreabilidade entre passos e mensagens

Os números P1–P9 se referem aos passos do fluxo principal do UC-017. A numeração automática do Mermaid ordena mensagens e retornos; ela não substitui os identificadores dos passos.

| Passo do UC | Mensagens ou fragmentos correspondentes | Regras e requisitos |
| --- | --- | --- |
| P1 — Solicitar geração | Usuário → TelaSugestao; `gerarSugestao(pedidoId, chaveOperacao, modo)` | RF-017; RN-024 |
| P2 — Autenticar e verificar propriedade | `autenticarSessao()`; `buscarFotografiaPropriaEOperacao(...)`; `alt` de sessão e propriedade | RF-017; RN-002/RN-003; RNF-001 |
| P3 — Recuperar fotografia e operação | Retorno com fotografia, estado da operação e modelos anteriores para UC-018; nota sobre contexto e destinatário preservados | RF-017/RF-018; RN-009/RN-022/RN-024 |
| P4 — Validar mínimos e categoria | `consultarEstadoCategoria(...)`; `validarMinimos(...)`; `alt` de pendências e estado | RF-017; RN-008/RN-010 |
| P5 — Consultar modelos compatíveis | `buscarModelosAtivosCompativeis(categoriaId, tipo, proximidade)` | RF-017/RF-018; RN-010 |
| P6 — Ordenar e selecionar | `selecionarModelo(...)`; `loop` de avaliação; `ordenarPorTagsPresentesEIdESelecionar()`; `alt` de ausência | RF-017/RF-018; RN-010/RN-013 |
| P7 — Personalizar | `personalizar(modeloSelecionado, fotografia)` e retorno do texto | RF-017; RN-011 |
| P8 — Registrar de forma atômica | `registrarSugestaoAtomica(...)`; confirmação, repetição concorrente ou rollback | RF-017/RF-018; RN-012/RN-024; RNF-006/RNF-007 |
| P8/A2 — Recuperar uma operação já confirmada | `recuperarSugestaoConfirmada(...)`; retorno da sugestão existente; salto de P4–P7 quando detectada em P3 | RF-017/RF-018; RN-024; RNF-006 |
| P9 — Apresentar resultado | Service → Controller → Tela → Usuário, somente com resultado confirmado | RF-017; RN-012; RNF-004 |

O UC-017 está diretamente relacionado a RN-003, RN-008, RN-010, RN-011, RN-012 e RN-024. RN-002 é transversal à autenticação. RN-009 e RN-022 explicam a preservação da fotografia, e RN-013 aplica-se à reutilização da realização por UC-018. Esses vínculos complementam o fluxo sem adicionar casos de uso artificiais.

Os RNFs relevantes são **RNF-001 — Segurança**, **RNF-004 — Desempenho**, **RNF-006 — Integridade** e **RNF-007 — Observabilidade**. A meta preliminar de RNF-004 é p95 de geração de até 5 segundos e de consultas de até 2 segundos, no cenário de 20 usuários simultâneos e 1.000 modelos. É uma meta proposta para implementação e medição futuras. A sequência não comprova o atendimento a essa meta. Integridade exige atomicidade e ausência de duplicações; observabilidade deve permitir correlacionar uma operação e sua falha conforme o requisito central, sem alterar o resultado do cenário.

## Checklist da revisão documental

| Critério do professor | Evidência | Situação |
| --- | --- | --- |
| Cenário de origem identificado | UC-017 e seus cenários nomeados no cabeçalho e na tabela | Revisado na documentação; não implementado |
| Ator e participantes necessários | Sete participantes, com responsabilidades definidas | Revisado na documentação; não implementado |
| Ordem compatível com o caso de uso | Mapeamento explícito P1–P9; antecipação de A2 explicada | Revisado na documentação; não implementado |
| Alternativa ou exceção representada | A1/A2 e E1/E2/E3 em fragmentos `alt` | Revisado na documentação; não implementado |
| Repetição com finalidade real | Avaliação dos modelos candidatos por `loop` | Revisado na documentação; não implementado |
| RN/RF/RNF rastreáveis | Relações por passo e mensagens na matriz | Revisado na documentação; não implementado |
| Responsabilidades distribuídas | Interface, aplicação, domínio e persistência separados | Revisado na documentação; não implementado |
| Resultado correspondente à pós-condição | Apresentação de sucesso somente após confirmação; rollback em E3 | Revisado na documentação; não implementado |
| Comportamento concorrente explicado | Chave conferida na transação e exclusão conferida no registro | Revisado na documentação; não implementado |
| Validação do software | Não existe implementação nem medição nesta entrega | Pendente de implementação e testes |

## Fontes e orientação do professor

- [ATIVIDADE03 — Análise e projeto](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/atividades/1_bim/ATIVIDADE03_analise_projeto.md): especificação, checklist, alternativas/exceções e rastreabilidade.
- [ATIVIDADE04 — Diagrama de sequência](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/atividades/1_bim/ATIVIDADE04_DIAGRAMA_SEQUENCIA.MD): participantes, `alt`, `loop`, distribuição de responsabilidades e justificativa derivada do caso de uso.
- [Aula de validação de casos de uso e sequência](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/aulas/SEMANA08/aula_validacao_casos_de_uso_diagrama_sequencia.pdf), páginas 60, 70–75, 85–92 e 99–102: escolha de cenário, responsabilidades, revisão e entregáveis. A separação dos participantes adotada aqui é uma decisão de projeto fundamentada nessa orientação didática.

Essas fontes orientam a modelagem. As regras específicas do VazaApp são as do contrato refatorado aprovado para esta entrega.
