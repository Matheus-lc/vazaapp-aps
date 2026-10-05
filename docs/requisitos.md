# Regras e requisitos — VazaApp 2.0

Esta é uma proposta de análise, com escopo ampliado autorizado pela equipe. Os IDs da versão anterior foram reorganizados; a correspondência histórica está em [refatoração](refatoracao.md). Os requisitos e critérios descrevem comportamento esperado, não evidência de software implementado.

## Convenções

- `RN`: restrição ou política do negócio.
- `RF`: comportamento que o software deve oferecer.
- `RNF`: qualidade ou restrição técnica mensurável/verificável.
- `UC`: objetivo do ator.
- `DG`: diagrama.
- Status geral: **Proposto**; especificação revisada documentalmente, sujeita à validação final da equipe.

## Regras de negócio

| ID | Regra |
|---|---|
| RN-001 | Cada conta possui nome, e-mail único e senha; telefone e foto são opcionais. E-mail é normalizado antes da comparação. |
| RN-002 | Somente contas ativas com credenciais válidas iniciam sessão. Encerrar sessão invalida a sessão corrente. Operações administrativas exigem o papel Administrador. |
| RN-003 | Dados de conta, perfil, destinatários, pedidos, sugestões, favoritos e avaliações pertencem ao usuário; nenhum usuário pode consultar ou alterar os dados de outro. |
| RN-004 | Recuperação de acesso usa token de uso único com validade de 15 minutos enviado ao e-mail cadastrado; a resposta pública não revela se uma conta existe. A redefinição por token não exige senha anterior e invalida todas as sessões da conta. |
| RN-005 | Excluir conta exige confirmação e senha atual; elimina os dados pessoais e seus registros dependentes e invalida suas sessões, sem excluir categorias ou modelos compartilhados. |
| RN-006 | O perfil de contexto é opcional e pode conter profissão, trabalho, moradia e contexto social/acadêmico; apenas informações fornecidas são usadas na personalização. |
| RN-007 | Destinatário exige nome, tipo (familiar, amigo, colega, superior ou outro) e proximidade (baixa, média ou alta). |
| RN-008 | Um pedido em rascunho exige título. Para gerar, exige situação descrita, categoria ativa e destinatário com tipo e proximidade válidos; contexto complementar é opcional. Uma operação nova de UC-017 direto realiza a primeira geração em rascunho; pedidos com_sugestao solicitam alternativas pelo UC-018. Repetir uma chave já confirmada segue RN-024. |
| RN-009 | Somente pedidos em rascunho podem ser editados. Pedido com sugestão preserva seus dados como fotografia do contexto. Excluir qualquer pedido exige confirmação e remove suas sugestões, favoritos e avaliações dependentes. |
| RN-010 | Geração utiliza somente modelos ativos de categoria ativa, compatíveis com o tipo de destinatário e a proximidade do pedido. Candidatos com mais tags presentes no contexto informado têm prioridade; comparação sem distinção entre maiúsculas/minúsculas e desempate por identificador crescente. Sem contexto ou tags compatíveis, usa o desempate estável. |
| RN-011 | Personalização substitui apenas parâmetros permitidos (nome_destinatario, situacao, contexto, nome_usuario) com dados fornecidos; contexto ausente é omitido sem inventar fatos. |
| RN-012 | Sugestão, vínculo ao pedido e histórico são persistidos em uma única transação. Só após confirmação da transação a sugestão é apresentada como concluída e o pedido passa a com_sugestao. |
| RN-013 | Nova sugestão reutiliza a fotografia do pedido e exclui modelos já utilizados nesse pedido. Se não há alternativa, informa indisponibilidade e mantém as sugestões existentes. |
| RN-014 | Um usuário pode favoritar uma sugestão própria uma única vez. Remover dos favoritos não exclui a sugestão nem o histórico. |
| RN-015 | Copiar transfere o texto da sugestão própria para a área de transferência mediante ação do usuário; não envia mensagens ao destinatário. |
| RN-016 | Cada sugestão admite uma avaliação do proprietário, com nota inteira de 1 a 5 e comentário opcional de até 500 caracteres. Avaliar é opcional; atualização substitui a avaliação existente. |
| RN-017 | Categoria possui nome único e descrição. Desativar categoria impede novas gerações e cadastro/ativação de modelos nela, preservando pedidos e sugestões históricos. |
| RN-018 | Modelo exige texto, categoria ativa e pelo menos um tipo de destinatário e uma proximidade permitidos. Tags de contexto são opcionais, mantidas como termos não vazios e sem duplicatas após normalização. Parâmetros fora da lista permitida são rejeitados. |
| RN-019 | Desativar modelo o remove das próximas seleções sem modificar sugestões já geradas. Atualizar um modelo também preserva o texto histórico das sugestões. |
| RN-020 | Consulta administrativa de avaliações apresenta categoria, modelo, nota, comentário e data, sem campos estruturados de nome, e-mail, destinatário ou contexto pessoal; não concede acesso aos pedidos dos usuários. Comentário livre pode conter informações fornecidas pelo autor, portanto a interface não promete anonimização integral. |
| RN-021 | Na alteração autenticada dos dados da conta (UC-005), alterar e-mail ou senha exige senha atual. O novo e-mail deve permanecer único. Alterar senha nesse caso invalida as demais sessões da conta; recuperação por token segue RN-004. |
| RN-022 | Excluir destinatário exige confirmação e preserva a fotografia do destinatário nos pedidos já criados; esses pedidos continuam consultáveis e aptos a gerar com os dados preservados. |
| RN-023 | Consultas sem registros ou sem correspondência ao filtro retornam uma lista vazia com orientação; ausência de resultado não é falha de persistência. |
| RN-024 | Geração usa uma chave de operação: repetir a mesma solicitação já confirmada devolve a mesma sugestão antes de revalidar o catálogo. A exclusão de modelos anteriores e o estado apropriado do pedido são conferidos no registro para impedir duplicação concorrente no mesmo pedido. |

## Requisitos funcionais

O texto abaixo define os comportamentos verificáveis. O fluxo de cada UC detalha entradas, decisões, alternativas e exceções; os RF não descrevem botões ou nomes de telas.

| ID | Requisito | UC principal |
|---|---|---|
| RF-001 | O sistema deve permitir criar uma conta com dados válidos e e-mail único. | UC-001 — Cadastrar conta |
| RF-002 | O sistema deve permitir iniciar uma sessão de usuário ou administrador com credenciais válidas. | UC-002 — Autenticar usuário |
| RF-003 | O sistema deve permitir encerrar a sessão corrente e impedir sua reutilização. | UC-003 — Encerrar sessão |
| RF-004 | O sistema deve permitir redefinir a senha por um token enviado ao e-mail da conta. | UC-004 — Recuperar acesso |
| RF-005 | O sistema deve permitir atualizar nome, contato, foto, e-mail ou senha da própria conta. | UC-005 — Alterar dados da conta |
| RF-006 | O sistema deve permitir eliminar a própria conta e seus dados dependentes após confirmação. | UC-006 — Excluir conta |
| RF-007 | O sistema deve permitir conhecer as informações de contexto atualmente armazenadas. | UC-007 — Consultar perfil de contexto |
| RF-008 | O sistema deve permitir cadastrar, corrigir ou limpar informações opcionais de contexto. | UC-008 — Atualizar perfil de contexto |
| RF-009 | O sistema deve permitir salvar um destinatário reutilizável em pedidos futuros. | UC-009 — Cadastrar destinatário |
| RF-010 | O sistema deve permitir localizar e consultar os próprios destinatários. | UC-010 — Consultar destinatários |
| RF-011 | O sistema deve permitir corrigir nome, tipo ou proximidade de um destinatário para pedidos futuros. | UC-011 — Atualizar destinatário |
| RF-012 | O sistema deve permitir remover um destinatário preservando os dados dos pedidos anteriores. | UC-012 — Excluir destinatário |
| RF-013 | O sistema deve permitir salvar um rascunho com situação e destinatário quando disponíveis. | UC-013 — Criar pedido de desculpa |
| RF-014 | O sistema deve permitir localizar os próprios pedidos e consultar seus detalhes e sugestões. | UC-014 — Consultar pedidos |
| RF-015 | O sistema deve permitir completar ou corrigir dados de um pedido antes da primeira geração. | UC-015 — Atualizar pedido em rascunho |
| RF-016 | O sistema deve permitir remover um pedido e os seus registros dependentes após confirmação. | UC-016 — Excluir pedido |
| RF-017 | O sistema deve permitir obter uma sugestão personalizada e registrada para um pedido válido. | UC-017 — Gerar sugestão personalizada |
| RF-018 | O sistema deve permitir obter outra alternativa para o mesmo pedido sem repetir os dados. | UC-018 — Solicitar nova sugestão |
| RF-019 | O sistema deve permitir localizar sugestões próprias já geradas e seu contexto preservado. | UC-019 — Consultar histórico de sugestões |
| RF-020 | O sistema deve permitir disponibilizar o texto de uma sugestão própria para uso em outro aplicativo. | UC-020 — Copiar sugestão |
| RF-021 | O sistema deve permitir guardar uma sugestão própria na coleção de favoritos. | UC-021 — Favoritar sugestão |
| RF-022 | O sistema deve permitir localizar sugestões marcadas como favoritas. | UC-022 — Consultar favoritos |
| RF-023 | O sistema deve permitir retirar uma sugestão da coleção de favoritos mantendo o histórico. | UC-023 — Remover sugestão dos favoritos |
| RF-024 | O sistema deve permitir registrar uma avaliação opcional de sugestão própria, com nota obrigatória de 1 a 5 e comentário opcional. | UC-024 — Avaliar sugestão |
| RF-025 | O sistema deve permitir corrigir uma avaliação anteriormente registrada. | UC-025 — Atualizar avaliação |
| RF-026 | O sistema deve permitir remover uma avaliação própria mantendo a sugestão. | UC-026 — Excluir avaliação |
| RF-027 | O sistema deve permitir disponibilizar uma nova categoria para classificar pedidos e modelos. | UC-027 — Cadastrar categoria |
| RF-028 | O sistema deve permitir localizar categorias ativas ou inativas e consultar seus dados. | UC-028 — Consultar categorias |
| RF-029 | O sistema deve permitir corrigir nome e descrição ou reativar uma categoria. | UC-029 — Atualizar categoria |
| RF-030 | O sistema deve permitir impedir novas gerações na categoria preservando os registros históricos. | UC-030 — Desativar categoria |
| RF-031 | O sistema deve permitir adicionar um modelo elegível para personalização. | UC-031 — Cadastrar modelo de desculpa |
| RF-032 | O sistema deve permitir localizar e consultar modelos do catálogo. | UC-032 — Consultar modelos de desculpa |
| RF-033 | O sistema deve permitir corrigir texto e compatibilidades ou reativar um modelo. | UC-033 — Atualizar modelo de desculpa |
| RF-034 | O sistema deve permitir retirar um modelo das próximas gerações mantendo o histórico. | UC-034 — Desativar modelo de desculpa |
| RF-035 | O sistema deve permitir consultar avaliações do catálogo sem expor dados pessoais dos pedidos. | UC-035 — Consultar avaliações recebidas |

RF-018 reutiliza o comportamento de RF-017 com a exclusão dos modelos já utilizados no pedido (RN-013). A relação `include` representa essa reutilização obrigatória. A autenticação é precondição das operações protegidas e não um `include` de cada UC.

## Requisitos não funcionais

As metas de qualidade são propostas para a futura implementação. O cenário de desempenho serve como referência inicial e deverá ser validado pela equipe; não foi medido neste repositório de análise.

| ID | Categoria | Requisito e forma de verificação | Aplicação |
|---|---|---|---|
| RNF-001 | Segurança | As operações devem verificar sessão, papel e propriedade no servidor. Senhas são armazenadas por hash resistente e não reversível; transporte usa TLS. Tokens de recuperação são protegidos, expiram e não aparecem em logs. | UC-001 a UC-035, conforme sessão/papel exigidos |
| RNF-002 | Privacidade | Consultas de usuário isolam seus dados. Consultas administrativas de avaliações omitem campos estruturados de identidade, destinatário e contexto. O formulário de comentário informa que o texto será lido pela administração e orienta não inserir dados pessoais. | UC-006 a UC-026; UC-035 |
| RNF-003 | Usabilidade | Mensagens em português explicam sucesso, pendências, vazio e falhas sem códigos internos. A interface deve oferecer rótulos claros e navegação por teclado, inclusive nas confirmações de exclusão. | Todos os UCs com interação humana |
| RNF-004 | Desempenho | Meta preliminar de projeto: percentil 95 de geração em até 5 s e de consultas em até 2 s, com 20 usuários simultâneos e catálogo de 1.000 modelos em ambiente de referência a documentar na implementação. | UC-007, UC-010, UC-014, UC-017 a UC-019, UC-022, UC-028, UC-032, UC-035 |
| RNF-005 | Compatibilidade | A interface web deve funcionar em navegadores modernos de desktop e dispositivos móveis, sem plugin. Se a API de área de transferência estiver indisponível, oferecer cópia manual. | Todos os UCs; alternativa específica em UC-020 |
| RNF-006 | Integridade | Persistência deve respeitar transações, integridade referencial e recuperação após falhas. Geração confirma sugestão, vínculo, histórico e estado juntos; exclusões eliminam dependências atomicamente; repetição da mesma chave não duplica uma geração. | UCs de escrita, especialmente UC-006, UC-016, UC-017, UC-018 |
| RNF-007 | Observabilidade | Falhas e gerações devem ter identificador de correlação, instante e resultado técnico, sem registrar senhas, tokens, texto pessoal do pedido ou conteúdo do contexto. Logs não substituem o histórico consultável. | UC-004, UC-017, UC-018 e exceções técnicas de persistência |
| RNF-008 | Manutenibilidade | Separar interface, coordenação da aplicação, regras do domínio e persistência. Categorias e modelos são mantidos pelo administrador sem alteração de código, respeitando a validação dos parâmetros. | UC-017, UC-018, UC-027 a UC-034; DG-002 |

## Dados e decisões do domínio

| Elemento | Informação e responsabilidade |
|---|---|
| Conta | Nome, e-mail normalizado, credencial protegida, contatos opcionais, papel e sessões. Administradores são provisionados pela operação; autoconcessão de papel não faz parte do cadastro público. |
| PerfilContexto | Informações opcionais do proprietário. Editar/limpar o perfil não altera pedidos já fotografados. |
| Destinatário | Nome, tipo e proximidade reutilizáveis. Pedido aceita destinatário cadastrado ou dados informados diretamente. |
| Pedido | Título, situação, categoria, fotografia do destinatário e contexto, proprietário e estado (`rascunho` ou `com_sugestao`). Durante o rascunho, a fotografia pode ser atualizada. |
| Categoria | Nome único, descrição, estado ativo/inativo. A escolha explícita da categoria comunica a natureza da situação; o texto livre a detalha. |
| ModeloDesculpa | Categoria, texto com parâmetros permitidos, tipos e proximidades compatíveis, contexto/tags de classificação e estado. A ordenação usa compatibilidade com o contexto informado e desempate estável por identificador. |
| Sugestão | Texto final preservado, pedido, modelo/versão utilizada e instante da geração. Favorito e avaliação referenciam a sugestão. |
| OperaçãoGeracao | Chave de operação, pedido, proprietário e resultado confirmado. Reexecutar a mesma chave consulta o resultado antes de revalidar a elegibilidade atual do catálogo. |
| Histórico | Visão das sugestões confirmadas; não exige uma segunda cópia do texto. A escrita é atômica com a sugestão. |
| Avaliação | Nota de 1 a 5 e comentário opcional até 500 caracteres; uma por sugestão. O comentário pode conter dados inseridos pelo autor, portanto a interface orienta não compartilhar dados pessoais. A administração recebe campos estruturados minimizados; isso não é uma garantia de anonimização do texto livre. |

## Fora do escopo aprovado

Geração por serviço externo de IA, envio automático de mensagens, integração com redes sociais, pagamentos e gestão pública de administradores. O mecanismo de geração permanece baseado em catálogo, como no projeto original.
