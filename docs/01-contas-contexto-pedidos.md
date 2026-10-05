# Casos de uso — Contas, contexto, destinatários e pedidos

Estas especificações descrevem objetivos observáveis dos atores, com início, resultado e tratamento de alternativas. Os identificadores de requisitos funcionais correspondem aos casos de uso: UC-001 realiza RF-001, e assim sucessivamente. As regras RN referenciadas constam do catálogo de requisitos e regras de negócio. Todos os casos têm status **Proposto**, sujeitos à validação acadêmica. A prioridade indica a importância para o escopo proposto.

## UC-001 — Cadastrar conta

- **Objetivo:** criar uma conta com dados válidos e e-mail único.
- **Ator principal:** Visitante.
- **Atores secundários:** nenhum.
- **Gatilho:** o visitante solicita criar uma conta.
- **Precondições:** o visitante ainda não iniciou uma sessão para a operação.
- **Entradas:** nome, e-mail e senha; telefone e foto opcionais.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O visitante solicita o cadastro de uma conta.
2. O sistema informa os dados obrigatórios e opcionais para o cadastro.
3. O visitante fornece nome, e-mail e senha, podendo informar telefone e foto, e confirma o cadastro.
4. O sistema valida os dados obrigatórios, normaliza o e-mail e verifica sua unicidade.
5. O sistema registra a conta como ativa, com os dados fornecidos e papel de Usuário.
6. O sistema confirma o cadastro concluído e disponibiliza a autenticação.

### Fluxos alternativos

- **A1 — Corrigir dados (origem: passo 4):** se falta um dado obrigatório ou o e-mail é inválido, o sistema identifica o problema e mantém os dados para correção. O visitante corrige os dados e retorna ao passo 3.
- **A2 — E-mail já utilizado (origem: passo 4):** o sistema rejeita o cadastro duplicado e orienta o visitante a usar outro e-mail ou recuperar o acesso. Se ele escolhe outro e-mail, retorna ao passo 3; se escolhe recuperar acesso, este caso termina sem cadastro e UC-004 pode ser iniciado.
- **A3 — Cancelar (origem: passo 3):** o visitante desiste antes de confirmar; o sistema encerra o caso sem criar conta.

### Exceções

- **E1 — Falha ao registrar (origem: passo 5):** o sistema informa que não concluiu o cadastro, não confirma uma conta inexistente e encerra a tentativa. Uma nova tentativa repete a verificação de unicidade.

### Pós-condições

- **Sucesso:** conta ativa criada, identificada por e-mail normalizado e único; a senha não é apresentada nas consultas de conta.
- **Falha/cancelamento:** nenhuma nova conta é confirmada; contas existentes permanecem preservadas.

### Rastreabilidade e verificação

- **RF-001:** permitir ao visitante cadastrar uma conta com nome, e-mail único e senha, com telefone e foto opcionais.
- **Regras:** RN-001.
- **CA-001.1:** o cadastro aceita os três dados obrigatórios sem exigir telefone ou foto.
- **CT-001.1:** informar nome `Ana Lima`, e-mail `ana.lima@example.com` ainda não usado e uma senha válida; omitir telefone e foto. Esperado: uma conta ativa e confirmação de sucesso.
- **CA-001.2:** a comparação de e-mails usa a normalização, impedindo contas duplicadas.
- **CT-001.2:** com `ana.lima@example.com` já cadastrado, tentar ` ANA.LIMA@example.com `. Esperado: rejeição por duplicidade e manutenção de uma única conta.
- **CA-001.3:** dados obrigatórios ausentes não geram conta.
- **CT-001.3:** confirmar o cadastro sem nome. Esperado: orientação para corrigir o nome, sem conta criada.

## UC-002 — Autenticar usuário

- **Objetivo:** iniciar uma sessão de usuário ou administrador com credenciais válidas.
- **Ator principal:** Visitante.
- **Atores secundários:** nenhum.
- **Gatilho:** o visitante solicita acesso autenticado.
- **Precondições:** existe uma conta ativa para o fluxo de sucesso; o papel dessa conta já foi definido.
- **Entradas:** e-mail e senha.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O visitante solicita iniciar uma sessão.
2. O sistema solicita e-mail e senha.
3. O visitante informa as credenciais e solicita autenticação.
4. O sistema verifica as credenciais e a condição ativa da conta.
5. O sistema inicia uma sessão associada à conta e ao seu papel.
6. O sistema disponibiliza as operações permitidas ao Usuário ou ao Administrador autenticado.

### Fluxos alternativos

- **A1 — Credenciais recusadas (origem: passo 4):** diante de credenciais inválidas ou conta inativa, o sistema não inicia sessão e informa que o acesso não foi autorizado. O visitante pode corrigir os dados, retornando ao passo 3, ou encerrar a tentativa.
- **A2 — Recuperar acesso (origem: passo 3):** o visitante escolhe recuperar a senha; este caso termina sem iniciar sessão e UC-004 pode ser iniciado.
- **A3 — Acesso administrativo (origem: passo 6):** se a conta tem papel Administrador, o sistema disponibiliza as operações administrativas previstas. O caso termina com sessão administrativa ativa; o visitante não escolhe nem promove seu próprio papel durante a autenticação.

### Exceções

- **E1 — Serviço de autenticação indisponível (origem: passos 4 ou 5):** o sistema informa a impossibilidade de concluir o acesso e encerra a tentativa sem sessão utilizável.

### Pós-condições

- **Sucesso:** sessão ativa vinculada à conta autenticada e ao papel autorizado.
- **Falha/cancelamento:** nenhuma sessão válida é iniciada pela tentativa recusada.

### Rastreabilidade e verificação

- **RF-002:** permitir autenticação de contas ativas, aplicando as permissões de seu papel.
- **Regras:** RN-002.
- **CA-002.1:** credenciais corretas de conta ativa iniciam sessão com seu papel.
- **CT-002.1:** autenticar uma conta ativa de Usuário com sua senha correta. Esperado: acesso às operações de Usuário e ausência de autorização administrativa.
- **CA-002.2:** conta inativa ou senha incorreta não inicia sessão.
- **CT-002.2:** autenticar uma conta inativa com senha correta. Esperado: acesso recusado e nenhuma sessão ativa criada.
- **CA-002.3:** operações administrativas dependem do papel cadastrado.
- **CT-002.3:** autenticar uma conta ativa com papel Administrador. Esperado: acesso às operações administrativas sem alteração do papel durante o login.

## UC-003 — Encerrar sessão

- **Objetivo:** encerrar a sessão corrente e impedir sua reutilização.
- **Ator principal:** Usuário / Administrador.
- **Atores secundários:** nenhum.
- **Gatilho:** o ator solicita encerrar a sessão em uso.
- **Precondições:** o ator possui uma sessão corrente.
- **Entradas:** solicitação de encerramento e identificação da sessão corrente.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O ator solicita encerrar sua sessão corrente.
2. O sistema identifica a sessão associada à solicitação.
3. O sistema invalida essa sessão.
4. O sistema confirma o encerramento e passa a tratar as próximas ações como ações de visitante.

### Fluxos alternativos

- **A1 — Sessão já encerrada ou expirada (origem: passo 2):** o sistema constata que a sessão já não é válida, informa que não há sessão ativa e encerra o caso com o mesmo resultado de acesso encerrado.

### Exceções

- **E1 — Falha na invalidação (origem: passo 3):** o sistema informa que não pôde confirmar o encerramento e encerra a tentativa. O ator pode repetir a operação; a confirmação de saída somente é emitida após constatar a invalidação.

### Pós-condições

- **Sucesso:** a sessão corrente não autoriza novas operações protegidas; outras sessões da conta não são encerradas por este caso.
- **Falha:** o encerramento não é apresentado como concluído sem a confirmação da invalidação.

### Rastreabilidade e verificação

- **RF-003:** permitir que Usuário e Administrador encerrem a sessão corrente.
- **Regras:** RN-002.
- **CA-003.1:** a sessão encerrada não pode ser reutilizada.
- **CT-003.1:** encerrar a sessão de um Usuário e tentar consultar seus pedidos usando a mesma sessão. Esperado: solicitação de autenticação, sem exposição dos pedidos.
- **CA-003.2:** encerrar uma sessão não encerra outra sessão da mesma conta.
- **CT-003.2:** manter duas sessões da mesma conta, encerrar a primeira e consultar o perfil pela segunda. Esperado: primeira inválida e segunda ainda autorizada.

## UC-004 — Recuperar acesso

- **Objetivo:** redefinir a senha por um token enviado ao e-mail da conta.
- **Ator principal:** Visitante.
- **Ator secundário:** Serviço de e-mail.
- **Gatilho:** o visitante solicita recuperação por não conseguir usar a senha atual.
- **Precondições:** não é necessário possuir sessão; existe uma conta com o e-mail informado para que a redefinição seja concluída.
- **Entradas:** e-mail da conta; posteriormente, token recebido e nova senha.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O visitante solicita recuperar o acesso e informa o e-mail da conta.
2. O sistema recebe a solicitação e consulta a existência de uma conta correspondente sem revelar o resultado ao visitante.
3. Para a conta encontrada, o sistema cria um token de uso único válido por 15 minutos e solicita ao Serviço de e-mail o envio das instruções de recuperação.
4. O Serviço de e-mail aceita o envio; o sistema apresenta a resposta neutra: “Se houver uma conta para o e-mail informado, você receberá instruções para recuperar o acesso”.
5. O visitante acessa as instruções recebidas e apresenta o token ao sistema.
6. O sistema verifica se o token corresponde à conta, está dentro da validade e ainda não foi utilizado.
7. O visitante informa e confirma uma nova senha.
8. O sistema valida a nova senha, registra a redefinição, inutiliza o token utilizado e invalida todas as sessões existentes da conta.
9. O sistema confirma a redefinição e disponibiliza a autenticação com a nova senha.

### Fluxos alternativos

- **A1 — E-mail sem conta correspondente (origem: passo 2):** o sistema não cria token nem solicita envio, apresenta a mesma resposta neutra do passo 4 e encerra a solicitação sem redefinição.
- **A2 — Token inválido, expirado ou utilizado (origem: passo 6):** o sistema rejeita a redefinição e informa que é necessário solicitar nova recuperação. O visitante pode retornar ao passo 1; o token recusado não altera a senha.
- **A3 — Nova senha incompleta ou confirmação divergente (origem: passo 8):** o sistema indica o problema e retorna ao passo 7. Ao receber a correção, verifica novamente a validade e o uso do token antes de gravar.
- **A4 — Visitante não conclui a recuperação (origem: passos 5 ou 7):** o visitante abandona o processo; a senha permanece inalterada e o token perde a validade ao completar 15 minutos.

### Exceções

- **E1 — Falha de envio (origem: passos 3 ou 4):** o sistema registra que as instruções não foram encaminhadas e conserva a resposta pública neutra, sem expor a existência da conta. A solicitação termina sem redefinir a senha; o visitante pode repetir a recuperação.
- **E2 — Falha na redefinição (origem: passo 8):** o sistema não confirma a troca nem consome o token sem a gravação correspondente. Informa que a operação não foi concluída e encerra a tentativa; uma nova tentativa exige token ainda válido e não utilizado.

### Pós-condições

- **Sucesso:** senha redefinida e token utilizado invalidado; o visitante deve autenticar-se para iniciar sessão.
- **Solicitação recebida sem resgate:** resposta neutra apresentada; a senha permanece inalterada.
- **Falha/cancelamento:** nenhuma redefinição é confirmada com token inválido ou sem gravação bem-sucedida.

### Rastreabilidade e verificação

- **RF-004:** permitir recuperar acesso sem sessão por token de uso único enviado ao e-mail cadastrado, válido por 15 minutos e com resposta pública neutra.
- **Regras:** RN-004.
- **CA-004.1:** o token válido permite uma redefinição e não pode ser reutilizado.
- **CT-004.1:** solicitar recuperação, usar o token após 5 minutos e tentar usá-lo novamente. Esperado: primeira redefinição concluída e segunda recusada, mantendo a senha definida na primeira.
- **CA-004.2:** token fora da validade não permite trocar a senha.
- **CT-004.2:** apresentar o token após 16 minutos. Esperado: rejeição e senha anterior preservada.
- **CA-004.3:** a resposta pública não identifica se há conta para o e-mail.
- **CT-004.3:** solicitar recuperação para um e-mail cadastrado e para `sem.conta@example.com`, não cadastrado. Esperado: mesma mensagem neutra nas duas solicitações; envio somente para a conta existente.

## UC-005 — Alterar dados da conta

- **Objetivo:** atualizar nome, contato, foto, e-mail ou senha da própria conta.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita alterar seus dados cadastrais.
- **Precondições:** sessão válida de Usuário; a conta consultada pertence ao usuário autenticado.
- **Entradas:** nome, e-mail, telefone e foto desejados; nova senha quando houver troca; senha atual para alterar e-mail ou senha.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O usuário solicita alterar os dados da própria conta.
2. O sistema apresenta os dados atuais editáveis, sem apresentar a senha.
3. O usuário informa as alterações e, para troca de e-mail ou senha, fornece também a senha atual.
4. O sistema valida os campos, verifica a senha atual quando exigida e verifica a unicidade do novo e-mail após normalização.
5. O sistema grava as alterações na própria conta.
6. Quando há troca de senha, o sistema invalida as demais sessões da conta.
7. O sistema confirma a atualização e apresenta os dados atualizados, sem revelar senhas.

### Fluxos alternativos

- **A1 — Atualizar somente dados não sensíveis (origem: passo 3):** o usuário altera nome, telefone ou foto sem trocar e-mail ou senha. O sistema prossegue ao passo 4 sem exigir senha atual para essas alterações.
- **A2 — Limpar dado opcional (origem: passo 3):** o usuário remove telefone ou foto. O sistema aceita a ausência desses dados e prossegue ao passo 4.
- **A3 — Corrigir dados (origem: passo 4):** e-mail já utilizado, nome obrigatório ausente, e-mail inválido ou senha atual incorreta impedem a gravação. O sistema informa o problema, preserva os dados anteriores e retorna ao passo 3.
- **A4 — Cancelar (origem: passo 3):** o usuário desiste das alterações; o caso termina mantendo os dados anteriormente registrados.

### Exceções

- **E1 — Sessão inválida ou conta não autorizada (origem: passos 1 a 5):** o sistema recusa a operação e encerra a tentativa sem consultar ou alterar dados de outra conta.
- **E2 — Falha ao salvar (origem: passo 5):** o sistema informa que não concluiu a atualização e encerra a tentativa sem confirmar os novos dados.

### Pós-condições

- **Sucesso:** dados próprios atualizados; e-mail permanece único; troca de senha invalida as demais sessões.
- **Falha/cancelamento:** dados anteriores preservados, sem alteração em contas de terceiros.

### Rastreabilidade e verificação

- **RF-005:** permitir editar a própria conta, exigindo senha atual para trocar e-mail ou senha e mantendo a unicidade do e-mail.
- **Regras:** RN-001, RN-003, RN-021.
- **CA-005.1:** a alteração de contato opcional não depende de trocar credenciais.
- **CT-005.1:** remover telefone e foto da própria conta, mantendo nome e e-mail. Esperado: atualização concluída e dados opcionais vazios.
- **CA-005.2:** a troca de e-mail exige senha atual correta e e-mail disponível.
- **CT-005.2:** informar um e-mail já pertencente a outra conta, mesmo com senha atual correta. Esperado: rejeição, preservando o e-mail anterior.
- **CA-005.3:** a troca de senha encerra as demais sessões.
- **CT-005.3:** com duas sessões da conta, alterar a senha pela primeira, informando a senha atual correta. Esperado: nova senha registrada e segunda sessão recusada na próxima operação protegida.

## UC-006 — Excluir conta

- **Objetivo:** eliminar a própria conta e seus dados dependentes após confirmação.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita excluir sua conta.
- **Precondições:** sessão válida de Usuário e conta própria existente.
- **Entradas:** confirmação explícita de exclusão e senha atual.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O usuário solicita excluir a própria conta.
2. O sistema informa que serão eliminados seus dados de conta, perfil, destinatários, pedidos, sugestões, favoritos e avaliações, e solicita confirmação e senha atual.
3. O usuário confirma a exclusão e fornece a senha atual.
4. O sistema verifica a confirmação, a senha e a titularidade da conta.
5. O sistema elimina a conta e todos os seus registros dependentes, preservando categorias e modelos compartilhados.
6. O sistema invalida as sessões da conta excluída.
7. O sistema confirma a exclusão e encerra o acesso autenticado.

### Fluxos alternativos

- **A1 — Cancelar exclusão (origem: passo 3):** o usuário não confirma ou cancela; o sistema termina o caso mantendo a conta e seus registros.
- **A2 — Senha incorreta (origem: passo 4):** o sistema recusa a exclusão e permite nova informação da senha, retornando ao passo 3, ou cancelamento sem alterações.

### Exceções

- **E1 — Sessão inválida ou conta não autorizada (origem: passos 1 a 4):** o sistema recusa a operação e encerra a tentativa sem eliminar dados.
- **E2 — Falha na exclusão (origem: passo 5):** o sistema não confirma conclusão parcial e preserva a consistência da conta e de seus registros dependentes. Informa a falha e encerra a tentativa sem apresentar a conta como excluída.

### Pós-condições

- **Sucesso:** conta, dados pessoais e dependências eliminados; nenhuma sessão dessa conta permanece válida; catálogo compartilhado preservado.
- **Falha/cancelamento:** a conta não é apresentada como excluída e nenhum dado de terceiros ou do catálogo é removido.

### Rastreabilidade e verificação

- **RF-006:** permitir excluir a própria conta após confirmação e senha atual, eliminando dependências e invalidando suas sessões.
- **Regras:** RN-003, RN-005.
- **CA-006.1:** a exclusão abrange os dados dependentes da conta sem remover o catálogo compartilhado.
- **CT-006.1:** excluir uma conta com destinatário, pedido, sugestão, favorito e avaliação, após confirmação e senha correta. Esperado: todos esses registros eliminados; categorias e modelos continuam disponíveis para outras contas.
- **CA-006.2:** senha incorreta ou ausência de confirmação não exclui a conta.
- **CT-006.2:** confirmar a exclusão com senha incorreta. Esperado: conta e dados preservados, sem confirmação de exclusão.
- **CA-006.3:** nenhuma sessão da conta excluída autoriza acesso.
- **CT-006.3:** excluir a conta enquanto ela possui duas sessões e tentar consultar dados pela outra sessão. Esperado: acesso recusado.

## UC-007 — Consultar perfil de contexto

- **Objetivo:** conhecer as informações de contexto atualmente armazenadas.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita consultar seu perfil de contexto.
- **Precondições:** sessão válida de Usuário.
- **Entradas:** identificação da conta autenticada.
- **Prioridade / status:** Média / Proposto.

### Fluxo principal

1. O usuário solicita consultar o próprio perfil de contexto.
2. O sistema identifica o proprietário pela sessão e recupera as informações de contexto desse usuário.
3. O sistema apresenta profissão, trabalho, moradia e contextos social e acadêmico que foram fornecidos.
4. O usuário consulta o perfil apresentado.

### Fluxos alternativos

- **A1 — Perfil ainda não preenchido (origem: passo 2):** o sistema apresenta ausência de informações de contexto e orienta que o preenchimento é opcional. O caso termina com consulta concluída; o usuário pode iniciar UC-008.
- **A2 — Perfil parcialmente preenchido (origem: passo 3):** o sistema apresenta somente os valores registrados e identifica os demais campos como não informados. Prossegue ao passo 4, sem completar informações por suposição.

### Exceções

- **E1 — Sessão inválida ou perfil não autorizado (origem: passos 1 ou 2):** o sistema recusa a consulta e encerra a tentativa sem apresentar perfil de terceiros.
- **E2 — Falha na recuperação (origem: passo 2):** o sistema informa que não pôde consultar os dados e encerra a tentativa. Não apresenta a falha como perfil vazio.

### Pós-condições

- **Sucesso:** informações próprias consultadas, ou ausência de perfil informada; nenhum dado é modificado.
- **Falha:** consulta não concluída e dados preservados.

### Rastreabilidade e verificação

- **RF-007:** permitir consultar o próprio perfil de contexto, incluindo a situação de perfil não preenchido.
- **Regras:** RN-003, RN-006, RN-023.
- **CA-007.1:** a consulta apresenta apenas as informações efetivamente registradas.
- **CT-007.1:** consultar perfil com profissão `Estudante` e demais campos vazios. Esperado: profissão apresentada e ausência dos demais valores, sem fatos inventados.
- **CA-007.2:** ausência de perfil é um resultado válido da consulta.
- **CT-007.2:** consultar o perfil de uma conta recém-criada. Esperado: indicação de perfil não preenchido e orientação de preenchimento opcional.
- **CA-007.3:** a consulta não revela contexto de outra conta.
- **CT-007.3:** usando a sessão de Ana, solicitar identificação de perfil de Bruno. Esperado: acesso recusado e nenhum contexto de Bruno apresentado.

## UC-008 — Atualizar perfil de contexto

- **Objetivo:** cadastrar, corrigir ou limpar informações opcionais de contexto.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita preencher ou alterar seu contexto.
- **Precondições:** sessão válida de Usuário.
- **Entradas:** profissão, trabalho, moradia e contexto social/acadêmico, todos opcionais; indicação de campos que devem ser limpos.
- **Prioridade / status:** Média / Proposto.

### Fluxo principal

1. O usuário solicita atualizar o próprio perfil de contexto.
2. O sistema apresenta os valores existentes, ou campos não preenchidos quando não há perfil.
3. O usuário informa, corrige ou limpa os campos desejados e confirma a atualização.
4. O sistema verifica a titularidade e a validade dos dados informados, sem exigir preenchimento integral do perfil.
5. O sistema registra as informações fornecidas e remove dos campos limpos os valores anteriores.
6. O sistema confirma a atualização e apresenta o contexto registrado.

### Fluxos alternativos

- **A1 — Limpar todo o contexto (origem: passo 3):** o usuário solicita deixar todos os campos vazios. O sistema aceita o perfil vazio e prossegue ao passo 4.
- **A2 — Corrigir entrada inválida (origem: passo 4):** o sistema informa o problema nos dados fornecidos e retorna ao passo 3 sem alterar os valores registrados.
- **A3 — Cancelar (origem: passo 3):** o usuário desiste antes da confirmação; o caso termina mantendo o contexto anterior.

### Exceções

- **E1 — Sessão inválida ou perfil não autorizado (origem: passos 1 a 5):** o sistema recusa a operação e encerra a tentativa sem alterar contexto próprio ou de terceiros.
- **E2 — Falha ao salvar (origem: passo 5):** o sistema informa que não concluiu a atualização e encerra a tentativa sem confirmar novos valores.

### Pós-condições

- **Sucesso:** perfil próprio atualizado, inclusive com valores vazios quando solicitado; pedidos com fotografia já preservada não são reescritos por esta alteração.
- **Falha/cancelamento:** perfil anterior preservado.

### Rastreabilidade e verificação

- **RF-008:** permitir preencher, corrigir ou limpar o próprio perfil opcional de contexto.
- **Regras:** RN-003, RN-006.
- **CA-008.1:** um perfil pode ser registrado parcialmente.
- **CT-008.1:** informar somente profissão `Professor`, mantendo os demais campos vazios. Esperado: profissão salva sem exigência dos campos opcionais.
- **CA-008.2:** limpar campos remove valores anteriores sem excluir a conta.
- **CT-008.2:** limpar todos os campos de um perfil preenchido. Esperado: contexto vazio e conta preservada.
- **CA-008.3:** alterações canceladas não substituem o perfil.
- **CT-008.3:** modificar o campo trabalho e cancelar antes de confirmar. Esperado: trabalho anterior mantido.

## UC-009 — Cadastrar destinatário

- **Objetivo:** salvar um destinatário reutilizável em pedidos futuros.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita registrar uma pessoa como destinatário.
- **Precondições:** sessão válida de Usuário.
- **Entradas:** nome; tipo familiar, amigo, colega, superior ou outro; proximidade baixa, média ou alta.
- **Prioridade / status:** Média / Proposto.

### Fluxo principal

1. O usuário solicita cadastrar um destinatário.
2. O sistema informa os dados obrigatórios e as opções de tipo e proximidade.
3. O usuário informa nome, tipo e proximidade e confirma o cadastro.
4. O sistema valida o nome e as opções informadas, atribuindo a propriedade do registro à conta autenticada.
5. O sistema registra o destinatário na coleção desse usuário.
6. O sistema confirma o cadastro e disponibiliza o destinatário para escolha em pedidos futuros.

### Fluxos alternativos

- **A1 — Dados incompletos ou opção inválida (origem: passo 4):** o sistema indica os campos que precisam de correção e retorna ao passo 3, sem cadastrar o destinatário.
- **A2 — Cancelar (origem: passo 3):** o usuário abandona o cadastro antes da confirmação; o caso termina sem registro.

### Exceções

- **E1 — Sessão inválida (origem: passos 1 a 5):** o sistema recusa o cadastro e encerra a tentativa sem criar registro para outra conta.
- **E2 — Falha ao registrar (origem: passo 5):** o sistema informa a falha e encerra a tentativa sem confirmar o destinatário como salvo.

### Pós-condições

- **Sucesso:** destinatário com nome, tipo e proximidade válidos registrado como pertencente ao usuário.
- **Falha/cancelamento:** nenhum destinatário novo é confirmado; registros anteriores preservados.

### Rastreabilidade e verificação

- **RF-009:** permitir cadastrar destinatários próprios reutilizáveis, com nome, tipo e proximidade obrigatórios.
- **Regras:** RN-003, RN-007.
- **CA-009.1:** os três campos válidos permitem cadastrar destinatário próprio.
- **CT-009.1:** cadastrar `Rita`, tipo `familiar`, proximidade `alta`. Esperado: destinatário salvo e disponível somente para o usuário proprietário.
- **CA-009.2:** ausência de tipo ou proximidade impede o cadastro.
- **CT-009.2:** informar nome e tipo `amigo`, deixando proximidade vazia. Esperado: orientação para escolher a proximidade e nenhum registro salvo.

## UC-010 — Consultar destinatários

- **Objetivo:** localizar e consultar os próprios destinatários.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita consultar seus destinatários cadastrados.
- **Precondições:** sessão válida de Usuário.
- **Entradas:** filtros opcionais por nome, tipo e proximidade; identificação de um destinatário próprio quando desejar seus detalhes.
- **Prioridade / status:** Média / Proposto.

### Fluxo principal

1. O usuário solicita a consulta de seus destinatários, podendo informar filtros.
2. O sistema consulta somente os destinatários pertencentes à conta autenticada e aplica os filtros fornecidos.
3. O sistema apresenta os destinatários encontrados, com nome, tipo e proximidade.
4. O usuário escolhe um destinatário para consultar seus detalhes.
5. O sistema verifica novamente a titularidade e apresenta os dados do destinatário escolhido.

### Fluxos alternativos

- **A1 — Nenhum destinatário encontrado (origem: passo 2):** o sistema apresenta lista vazia e orienta o usuário a cadastrar um destinatário ou rever os filtros. O caso termina com consulta concluída; o usuário pode iniciar UC-009 ou repetir o passo 1.
- **A2 — Consultar somente a lista (origem: passo 4):** o usuário não escolhe um registro; o caso termina com a lista consultada.
- **A3 — Alterar filtros (origem: passo 3):** o usuário modifica ou remove os filtros e retorna ao passo 1.

### Exceções

- **E1 — Sessão inválida ou destinatário não autorizado (origem: passos 1, 2 ou 5):** o sistema recusa a consulta e encerra a tentativa sem revelar destinatários de terceiros.
- **E2 — Registro removido durante a consulta (origem: passo 5):** o sistema informa que o destinatário não está mais disponível e retorna ao passo 2 para atualizar a lista.
- **E3 — Falha de consulta (origem: passos 2 ou 5):** o sistema informa que não pôde recuperar os dados e encerra a tentativa, sem apresentar a falha como ausência de destinatários.

### Pós-condições

- **Sucesso:** lista ou detalhes de destinatários próprios consultados, inclusive lista vazia quando aplicável.
- **Falha:** nenhum destinatário é modificado ou exposto indevidamente.

### Rastreabilidade e verificação

- **RF-010:** permitir listar, filtrar e consultar detalhes dos destinatários próprios.
- **Regras:** RN-003, RN-007, RN-023.
- **CA-010.1:** filtros retornam somente destinatários próprios correspondentes.
- **CT-010.1:** Ana tem Rita/familiar/alta e Paulo/colega/baixa; filtrar por `familiar`. Esperado: somente Rita na lista, sem destinatários de outras contas.
- **CA-010.2:** ausência de resultado é tratada como consulta concluída.
- **CT-010.2:** consultar uma conta sem destinatários. Esperado: lista vazia e orientação para cadastro, sem mensagem de falha de armazenamento.
- **CA-010.3:** detalhes de destinatário de terceiros não podem ser consultados.
- **CT-010.3:** usando a sessão de Ana, solicitar detalhes de um destinatário de Bruno. Esperado: acesso recusado e nenhum dado apresentado.

## UC-011 — Atualizar destinatário

- **Objetivo:** corrigir nome, tipo ou proximidade de um destinatário para pedidos futuros.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita alterar um destinatário próprio.
- **Precondições:** sessão válida de Usuário; destinatário próprio cadastrado.
- **Entradas:** identificação do destinatário; novo nome, tipo e/ou proximidade.
- **Prioridade / status:** Média / Proposto.

### Fluxo principal

1. O usuário identifica um destinatário próprio e solicita atualizá-lo.
2. O sistema verifica a titularidade e apresenta os dados atuais.
3. O usuário corrige os dados desejados e confirma a atualização.
4. O sistema valida nome, tipo e proximidade obrigatórios.
5. O sistema atualiza o destinatário cadastrado, preservando as fotografias já armazenadas em pedidos.
6. O sistema confirma a alteração e apresenta os novos dados para uso em pedidos futuros.

### Fluxos alternativos

- **A1 — Dados incompletos ou inválidos (origem: passo 4):** o sistema aponta os problemas e retorna ao passo 3, mantendo o cadastro anterior.
- **A2 — Cancelar (origem: passo 3):** o usuário desiste; o caso termina sem atualizar o destinatário.

### Exceções

- **E1 — Sessão inválida, registro inexistente ou não autorizado (origem: passos 1, 2 ou 5):** o sistema recusa a operação e encerra a tentativa sem alterar registros próprios ou de terceiros.
- **E2 — Falha ao atualizar (origem: passo 5):** o sistema informa que não concluiu a atualização e encerra a tentativa sem confirmar novos valores.

### Pós-condições

- **Sucesso:** destinatário próprio atualizado; pedidos anteriores conservam a fotografia que já registraram.
- **Falha/cancelamento:** dados do destinatário e dos pedidos preservados.

### Rastreabilidade e verificação

- **RF-011:** permitir corrigir nome, tipo e proximidade de destinatário próprio sem reescrever fotografias de pedidos anteriores.
- **Regras:** RN-003, RN-007.
- **CA-011.1:** alterações válidas passam a ser usadas em novas escolhas do destinatário.
- **CT-011.1:** alterar Rita de tipo `colega` e proximidade `baixa` para `amigo` e `alta`, depois escolhê-la em um novo pedido. Esperado: novo pedido recebe os valores atualizados.
- **CA-011.2:** os dados armazenados em pedidos anteriores são preservados.
- **CT-011.2:** após a alteração do teste anterior, consultar um pedido criado quando Rita era colega/baixa. Esperado: fotografia desse pedido ainda registra colega/baixa.
- **CA-011.3:** os campos obrigatórios não podem ser removidos do cadastro.
- **CT-011.3:** tentar salvar o destinatário com nome vazio. Esperado: rejeição e nome anterior mantido.

## UC-012 — Excluir destinatário

- **Objetivo:** remover um destinatário preservando os dados dos pedidos anteriores.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita excluir um destinatário próprio.
- **Precondições:** sessão válida de Usuário; destinatário próprio existente.
- **Entradas:** identificação do destinatário e confirmação explícita de exclusão.
- **Prioridade / status:** Média / Proposto.

### Fluxo principal

1. O usuário identifica um destinatário e solicita excluí-lo.
2. O sistema verifica a titularidade, apresenta o registro e informa que os dados já preservados em pedidos serão mantidos.
3. O usuário confirma a exclusão.
4. O sistema remove o destinatário da coleção do usuário, preservando as fotografias dos pedidos existentes.
5. O sistema confirma a exclusão e deixa de disponibilizar esse cadastro para novas escolhas.

### Fluxos alternativos

- **A1 — Cancelar (origem: passo 3):** o usuário não confirma a exclusão; o caso termina mantendo o destinatário.

### Exceções

- **E1 — Sessão inválida ou destinatário não autorizado (origem: passos 1, 2 ou 4):** o sistema recusa a operação e encerra a tentativa sem remover dados de terceiros.
- **E2 — Destinatário já removido (origem: passos 2 ou 4):** o sistema informa que o registro não está mais disponível e encerra o caso sem modificar os pedidos existentes.
- **E3 — Falha ao excluir (origem: passo 4):** o sistema informa que não concluiu a exclusão e encerra a tentativa sem confirmação de remoção.

### Pós-condições

- **Sucesso:** destinatário excluído da coleção; pedidos anteriores permanecem consultáveis e podem gerar sugestões usando os dados preservados, se atenderem aos demais requisitos de geração.
- **Falha/cancelamento:** nenhuma fotografia de pedido é eliminada; a remoção não é apresentada como concluída sem confirmação.

### Rastreabilidade e verificação

- **RF-012:** permitir excluir destinatário próprio após confirmação, preservando sua fotografia nos pedidos anteriores.
- **Regras:** RN-003, RN-022.
- **CA-012.1:** o destinatário excluído deixa de aparecer na coleção.
- **CT-012.1:** excluir Rita com confirmação e consultar destinatários. Esperado: Rita ausente da coleção.
- **CA-012.2:** a exclusão não elimina os dados necessários de um pedido anterior.
- **CT-012.2:** criar pedido válido com Rita, excluir o destinatário e consultar o pedido. Esperado: nome, tipo e proximidade preservados; a exclusão do cadastro não impede por si só a geração.
- **CA-012.3:** cancelamento mantém o destinatário.
- **CT-012.3:** solicitar exclusão de Paulo e cancelar a confirmação. Esperado: Paulo permanece cadastrado.

## UC-013 — Criar pedido de desculpa

- **Objetivo:** salvar um rascunho com situação e destinatário quando disponíveis.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita iniciar um novo pedido de desculpa.
- **Precondições:** sessão válida de Usuário; nenhum pedido anterior ou destinatário cadastrado é exigido.
- **Entradas:** título obrigatório; situação, categoria, destinatário e contexto complementar opcionais nesta etapa. O destinatário pode ser escolhido da coleção própria ou informado diretamente com nome, tipo e proximidade.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O usuário solicita criar um pedido de desculpa.
2. O sistema informa que o título é obrigatório para salvar o rascunho e apresenta os dados opcionais do pedido.
3. O usuário informa o título, acrescenta os dados disponíveis e confirma o salvamento.
4. O sistema valida o título e os dados fornecidos; se há destinatário, verifica nome, tipo e proximidade, e, se escolhido de um cadastro, sua titularidade.
5. O sistema registra o pedido como `rascunho`, pertencente ao usuário, e guarda uma fotografia dos dados informados, incluindo os dados do destinatário escolhido ou informado diretamente.
6. O sistema confirma o rascunho salvo e informa quais dados ainda são necessários para gerar uma sugestão.

### Fluxos alternativos

- **A1 — Salvar somente com título (origem: passo 3):** o usuário ainda não informa situação, categoria ou destinatário. O sistema prossegue ao passo 4 e salva o rascunho, sem apresentar geração como disponível até que seus dados obrigatórios estejam completos.
- **A2 — Escolher destinatário cadastrado (origem: passo 3):** o usuário escolhe um destinatário próprio. O sistema utiliza seu nome, tipo e proximidade para a fotografia do pedido e retorna à continuação do passo 3; a escolha não cria um novo cadastro.
- **A3 — Informar destinatário diretamente (origem: passo 3):** o usuário informa nome, tipo e proximidade sem cadastrar um destinatário reutilizável. O sistema usa esses dados no pedido e retorna à continuação do passo 3; não cria registro na coleção de destinatários.
- **A4 — Usar contexto de perfil (origem: passo 3):** quando disponível, o usuário pode aproveitar informações do próprio perfil como contexto complementar do pedido. O sistema guarda os valores efetivamente escolhidos na fotografia e retorna à continuação do passo 3. Sem perfil, o pedido pode ser salvo normalmente.
- **A5 — Corrigir dados (origem: passo 4):** título vazio ou dados fornecidos inválidos impedem o salvamento. O sistema informa o problema e retorna ao passo 3.
- **A6 — Cancelar (origem: passo 3):** o usuário desiste antes de confirmar; o caso termina sem criar pedido.

### Exceções

- **E1 — Sessão inválida ou destinatário não autorizado (origem: passos 1, 4 ou 5):** o sistema recusa a operação e encerra a tentativa sem criar pedido com dados de outra conta.
- **E2 — Falha ao salvar (origem: passo 5):** o sistema informa que não concluiu a criação e encerra a tentativa sem confirmar o pedido como registrado.

### Pós-condições

- **Sucesso:** pedido próprio registrado como rascunho, com título e fotografia dos dados disponíveis; nenhuma sugestão é gerada por este caso.
- **Falha/cancelamento:** nenhum pedido novo é confirmado.

### Rastreabilidade e verificação

- **RF-013:** permitir criar rascunho de pedido com título obrigatório e demais dados disponíveis, aceitando destinatário cadastrado ou informado diretamente.
- **Regras:** RN-003, RN-007, RN-008.
- **CA-013.1:** título suficiente permite salvar rascunho, sem liberar geração incompleta.
- **CT-013.1:** salvar título `Atraso na reunião`, omitindo situação, categoria e destinatário. Esperado: rascunho criado e indicação dos dados faltantes para gerar.
- **CA-013.2:** destinatário direto permite criar pedido sem cadastro reutilizável.
- **CT-013.2:** informar título, situação, categoria ativa e destinatário direto `Rita`/`superior`/`baixa`. Esperado: rascunho salvo com esses dados e nenhum novo destinatário na coleção.
- **CA-013.3:** escolha de destinatário guarda fotografia dos dados do momento.
- **CT-013.3:** criar rascunho escolhendo Rita/`colega`/`média`; alterar depois o cadastro para `amigo`/`alta`. Esperado: pedido mantém colega/média até eventual edição explícita do rascunho.

## UC-014 — Consultar pedidos

- **Objetivo:** localizar os próprios pedidos e consultar seus detalhes e sugestões.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita consultar seus pedidos.
- **Precondições:** sessão válida de Usuário.
- **Entradas:** filtros opcionais por título, estado e categoria; identificação do pedido próprio escolhido.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O usuário solicita consultar seus pedidos, podendo informar filtros.
2. O sistema consulta os pedidos da conta autenticada e aplica os filtros informados.
3. O sistema apresenta os resultados com identificação, título, estado e categoria quando informada.
4. O usuário escolhe um pedido para consultar seus detalhes.
5. O sistema verifica a titularidade e recupera a fotografia do pedido e as sugestões associadas.
6. O sistema apresenta título, estado, situação, categoria, dados preservados do destinatário, contexto complementar e sugestões existentes.

### Fluxos alternativos

- **A1 — Nenhum pedido encontrado (origem: passo 2):** o sistema apresenta lista vazia e orienta a rever os filtros ou criar um pedido. O caso termina com consulta concluída; o usuário pode retornar ao passo 1 ou iniciar UC-013.
- **A2 — Consultar somente a lista (origem: passo 4):** o usuário não escolhe um pedido; o caso termina com a lista consultada.
- **A3 — Pedido em rascunho (origem: passo 6):** o sistema identifica os campos ainda não preenchidos e a inexistência de sugestões, sem inventar informações. O caso termina com os detalhes do rascunho consultados.
- **A4 — Rever filtros (origem: passo 3):** o usuário modifica os filtros e retorna ao passo 1.

### Exceções

- **E1 — Sessão inválida ou pedido não autorizado (origem: passos 1, 2 ou 5):** o sistema recusa a consulta e encerra a tentativa sem revelar pedidos de terceiros.
- **E2 — Pedido removido durante a consulta (origem: passo 5):** o sistema informa que o registro não está mais disponível e retorna ao passo 2 para atualizar a lista.
- **E3 — Falha de consulta (origem: passos 2 ou 5):** o sistema informa que não pôde recuperar os dados e encerra a tentativa, sem confundir a falha com resultado vazio.

### Pós-condições

- **Sucesso:** lista ou detalhes de pedidos próprios consultados, inclusive lista vazia; fotografias e sugestões permanecem inalteradas.
- **Falha:** nenhum pedido é modificado e nenhum dado de terceiros é apresentado.

### Rastreabilidade e verificação

- **RF-014:** permitir listar, filtrar e consultar detalhes e sugestões dos pedidos próprios.
- **Regras:** RN-003, RN-009, RN-023.
- **CA-014.1:** a consulta de detalhes usa os dados preservados no pedido.
- **CT-014.1:** consultar pedido com sugestão cuja fotografia registra Rita como `colega`, após alterar seu cadastro para `amigo`. Esperado: detalhes ainda mostram `colega` e a sugestão originalmente gerada.
- **CA-014.2:** filtros podem produzir uma lista vazia sem erro.
- **CT-014.2:** filtrar por estado `com_sugestao` em conta que possui somente rascunhos. Esperado: lista vazia e orientação para rever filtros.
- **CA-014.3:** pedidos de terceiros permanecem inacessíveis.
- **CT-014.3:** solicitar um pedido de Bruno usando a sessão de Ana. Esperado: recusa sem apresentação de situação, destinatário ou sugestão desse pedido.

## UC-015 — Atualizar pedido em rascunho

- **Objetivo:** completar ou corrigir dados de um pedido antes da primeira geração.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita alterar um pedido próprio em rascunho.
- **Precondições:** sessão válida de Usuário; pedido próprio existente no estado `rascunho` e sem sugestão gerada.
- **Entradas:** identificação do pedido; título, situação, categoria, destinatário e contexto complementar desejados. O destinatário pode ser escolhido do cadastro próprio ou informado diretamente.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O usuário identifica um pedido próprio e solicita sua atualização.
2. O sistema verifica a titularidade e o estado `rascunho` e apresenta os dados registrados no pedido.
3. O usuário completa, corrige ou remove campos opcionais, mantendo um título, e confirma as alterações.
4. O sistema valida o título e os dados informados, inclusive o tipo e a proximidade do destinatário quando fornecido.
5. O sistema verifica novamente se o pedido continua em rascunho e grava a fotografia atualizada dos dados, preservando a propriedade do pedido.
6. O sistema confirma a atualização e informa se o pedido atende aos dados obrigatórios para gerar uma sugestão.

### Fluxos alternativos

- **A1 — Trocar a forma de informar destinatário (origem: passo 3):** o usuário escolhe outro destinatário cadastrado ou informa seus dados diretamente. O sistema utiliza os novos valores na fotografia do rascunho e retorna à continuação do passo 3; não exige cadastro reutilizável.
- **A2 — Limpar dados opcionais do rascunho (origem: passo 3):** o usuário remove situação, categoria, destinatário ou contexto complementar ainda não definitivos. O sistema permite salvar o rascunho com título e prossegue ao passo 4, indicando no passo 6 os dados que faltam para gerar.
- **A3 — Corrigir dados inválidos (origem: passo 4):** título vazio ou dados informados inválidos impedem a gravação. O sistema informa o problema e retorna ao passo 3, mantendo a fotografia anterior.
- **A4 — Cancelar (origem: passo 3):** o usuário abandona as alterações; o caso termina com o rascunho anterior preservado.

### Exceções

- **E1 — Sessão inválida, pedido inexistente ou não autorizado (origem: passos 1, 2 ou 5):** o sistema recusa a operação e encerra a tentativa sem alterar pedidos de terceiros.
- **E2 — Pedido já possui sugestão (origem: passos 2 ou 5):** o sistema recusa a edição, inclusive se a primeira geração foi concluída durante a tentativa. Encerra o caso preservando a fotografia utilizada na sugestão.
- **E3 — Falha ao atualizar (origem: passo 5):** o sistema informa que não concluiu a atualização e encerra a tentativa sem confirmar os novos dados.

### Pós-condições

- **Sucesso:** dados e fotografia do pedido em rascunho atualizados; estado permanece `rascunho`, sem geração automática de sugestão.
- **Falha/cancelamento:** fotografia anterior preservada; pedido com sugestão não é editado.

### Rastreabilidade e verificação

- **RF-015:** permitir completar ou corrigir pedido próprio somente enquanto estiver em rascunho, mantendo título obrigatório.
- **Regras:** RN-003, RN-007, RN-008, RN-009.
- **CA-015.1:** é possível completar um rascunho para torná-lo apto à geração.
- **CT-015.1:** em rascunho com título `Atraso`, acrescentar situação, categoria ativa e destinatário direto válido. Esperado: dados salvos e indicação de prontidão para geração, sem sugestão gerada por essa edição.
- **CA-015.2:** primeira sugestão concluída impede a edição da fotografia.
- **CT-015.2:** tentar alterar situação de pedido no estado `com_sugestao`. Esperado: recusa e fotografia da sugestão preservada.
- **CA-015.3:** o rascunho pode voltar a ter campos opcionais vazios, mantendo o título.
- **CT-015.3:** remover destinatário de rascunho anteriormente completo. Esperado: rascunho salvo sem destinatário e indicação de impedimento à geração até completar esse dado.

## UC-016 — Excluir pedido

- **Objetivo:** remover um pedido e os seus registros dependentes após confirmação.
- **Ator principal:** Usuário.
- **Atores secundários:** nenhum.
- **Gatilho:** o usuário solicita excluir um pedido próprio.
- **Precondições:** sessão válida de Usuário; pedido próprio existente, em `rascunho` ou `com_sugestao`.
- **Entradas:** identificação do pedido e confirmação explícita de exclusão.
- **Prioridade / status:** Alta / Proposto.

### Fluxo principal

1. O usuário identifica um pedido e solicita sua exclusão.
2. O sistema verifica a titularidade e informa que o pedido, suas sugestões, favoritos e avaliações dependentes serão removidos.
3. O usuário confirma a exclusão.
4. O sistema remove o pedido e os registros dependentes, preservando conta, perfil, destinatários cadastrados e catálogo compartilhado.
5. O sistema confirma a exclusão e atualiza os resultados das consultas afetadas.

### Fluxos alternativos

- **A1 — Cancelar (origem: passo 3):** o usuário não confirma a exclusão; o caso termina mantendo pedido, sugestões, favoritos e avaliações.
- **A2 — Excluir rascunho sem sugestões (origem: passo 4):** o sistema remove somente o pedido, pois não existem sugestões, favoritos ou avaliações dependentes, e prossegue ao passo 5.

### Exceções

- **E1 — Sessão inválida ou pedido não autorizado (origem: passos 1, 2 ou 4):** o sistema recusa a operação e encerra a tentativa sem remover pedido de outra conta.
- **E2 — Pedido já excluído (origem: passos 2 ou 4):** o sistema informa que o registro não está mais disponível e encerra o caso sem remover outros registros.
- **E3 — Falha na exclusão (origem: passo 4):** o sistema não confirma remoção parcial e preserva a consistência entre pedido e dependências. Informa que a exclusão não foi concluída e encerra a tentativa.

### Pós-condições

- **Sucesso:** pedido próprio e suas sugestões, favoritos e avaliações dependentes eliminados; nenhum registro dependente continua disponível no histórico ou nos favoritos.
- **Falha/cancelamento:** remoção não confirmada; dados alheios ao pedido não são eliminados.

### Rastreabilidade e verificação

- **RF-016:** permitir excluir qualquer pedido próprio após confirmação, removendo suas sugestões, favoritos e avaliações dependentes.
- **Regras:** RN-003, RN-009.
- **CA-016.1:** pedido com sugestão pode ser excluído com todas as dependências.
- **CT-016.1:** excluir pedido com duas sugestões, um favorito e uma avaliação. Esperado: pedido e todos esses registros ausentes das consultas; destinatário cadastrado e modelos do catálogo preservados.
- **CA-016.2:** pedido em rascunho também pode ser excluído.
- **CT-016.2:** confirmar a exclusão de rascunho que contém apenas título. Esperado: pedido removido sem afetar outro rascunho da mesma conta.
- **CA-016.3:** cancelamento preserva pedido e dependências.
- **CT-016.3:** iniciar exclusão de pedido com sugestão e cancelar a confirmação. Esperado: pedido, sugestão, favorito e avaliação ainda consultáveis.
