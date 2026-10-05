# Casos de uso — sugestões, avaliações e catálogo

Especificações UC-017 a UC-035 da proposta refatorada do VazaApp. Os fluxos representam comportamento esperado, ainda não implementado. “Sistema” identifica o VazaApp; repositórios internos e serviços internos não são atores.

## UC-017 — Gerar sugestão personalizada

**Status:** Proposto. **Prioridade:** Alta.

**Objetivo:** Obter uma sugestão personalizada e registrada para um pedido válido.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário solicita a geração em um pedido próprio; também pode ser invocado obrigatoriamente pelo UC-018.

**Pré-condições:** Conta ativa; pedido existente. A sessão e a propriedade do pedido são conferidas no fluxo, antes do acesso aos dados. A fotografia do pedido reúne situação, categoria, destinatário e contexto copiado do perfil quando o pedido foi criado ou alterado.

**Entradas:** Identificador do pedido e chave da operação; no UC-018, conjunto de modelos já usados a excluir. Os dados para personalização vêm da fotografia do pedido.

### Fluxo principal

1. **Usuário:** solicita a geração para um pedido.
2. **Sistema:** autentica a sessão e autoriza a propriedade do pedido.
3. **Sistema:** recupera a fotografia do pedido e seu contexto, incluindo o perfil copiado quando o pedido foi criado ou alterado; verifica também se a chave já possui comprovante de operação confirmada e recupera os modelos anteriormente usados, aplicando sua exclusão quando invocado pelo UC-018.
4. **Sistema:** valida situação, categoria ativa, destinatário com tipo e proximidade válidos e estado do pedido. Na invocação direta com chave nova, exige rascunho; quando incluído por UC-018, exige `com_sugestao`.
5. **Sistema:** consulta modelos ativos da categoria, compatíveis com o tipo e a proximidade do destinatário.
6. **Sistema:** ordena os candidatos pelo contexto informado e seleciona um modelo; quando chamado pelo UC-018, exclui os modelos já usados nesse pedido.
7. **Sistema:** personaliza os parâmetros permitidos com os dados fornecidos, omitindo contexto ausente e sem inventar fatos.
8. **Sistema:** persiste sugestão, vínculo, histórico e estado `com_sugestao` do pedido em uma transação atômica. Confere a chave da operação e os modelos já usados para controlar repetição e concorrência.
9. **Sistema:** apresenta a sugestão somente após a confirmação da transação.

### Fluxos alternativos

- **A1 — Sem modelo compatível. Origem: passo 6.** O sistema informa a indisponibilidade. **Retorno:** não retorna; termina sem persistir sugestão e sem alterar o pedido.
- **A2 — Operação já confirmada. Origem: passo 3.** A chave identifica uma geração já concluída. O sistema salta os passos 4 a 7 e, no passo 8, recupera a sugestão vinculada ao comprovante, sem nova gravação. **Retorno:** passo 9, apresentando o mesmo resultado mesmo que a categoria tenha sido desativada posteriormente. Se a confirmação concorrente só for detectada no passo 8, aplica o mesmo retorno.

### Exceções

- **E1 — Dados incompletos, categoria inativa ou estado incompatível. Origem: passo 4.** O sistema apresenta as pendências. Se uma nova solicitação direta encontrar pedido já gerado, orienta UC-018. **Fim:** encerra sem gerar; o usuário pode corrigir um rascunho pelo UC-015. Se o pedido já tem sugestão e a categoria está inativa, deve aguardar sua reativação ou criar outro pedido válido pelo UC-013.
- **E2 — Sessão inválida ou pedido de outro usuário. Origem: passo 2.** O sistema nega acesso sem revelar o conteúdo do pedido. **Fim:** encerra sem leitura ou alteração dos dados protegidos.
- **E3 — Falha de persistência ou conflito concorrente de estado/modelo. Origem: passo 8.** O sistema desfaz a transação e registra o erro sem dados pessoais; inclui o conflito em que outra operação confirmou o mesmo modelo antes desta tentativa. **Fim:** encerra sem sugestão parcial ou mudança de estado desta operação; nova tentativa conserva a mesma chave enquanto a operação não foi confirmada.

**Pós-condição de sucesso:** Sugestão, histórico e pedido são consistentes; o usuário recebe o resultado confirmado. Na repetição idempotente, recebe a sugestão existente.

**Pós-condição de falha:** Nenhum resultado parcial é apresentado como concluído; os registros anteriores permanecem íntegros.

**Rastreabilidade:** RF-017; RN-002, RN-003, RN-008, RN-009, RN-010, RN-011, RN-012, RN-024; RNF-001, RNF-002, RNF-003, RNF-004, RNF-006, RNF-007, RNF-008.

### Critérios de aceite

- **CA-017.1:** Com pedido válido, somente modelo ativo e compatível é personalizado; campos ausentes não produzem fatos inventados.
- **CA-017.2:** A sugestão só é apresentada como concluída quando sugestão, histórico e estado do pedido foram confirmados juntos.
- **CA-017.3:** Repetir a mesma chave devolve a mesma sugestão; um usuário não consegue gerar para pedido alheio.

### Casos de teste propostos

- **CT-017.1 → CA-017.1:** Primeiro gerar para pedido em rascunho com destinatário amigo/alta e contexto vazio; catálogo com modelo compatível e outro para superior. Depois, em outro pedido válido com contexto “prova acadêmica”, oferecer dois modelos compatíveis, um com tag `acadêmica` e outro sem tags. Esperado: no primeiro caso, somente o compatível e contexto omitido; no segundo, preferência pelo modelo com tag presente, sem inventar fatos. Nova chave direta em pedido já gerado é orientada para UC-018.
- **CT-017.2 → CA-017.2:** Induzir falha ao gravar o histórico no passo 8. Esperado: rollback da sugestão e do estado do pedido; nenhuma mensagem de geração concluída.
- **CT-017.3 → CA-017.3:** Confirmar a chave `geracao-017-1`, desativar a categoria e reenviar a mesma chave; depois solicitar o pedido com outra conta. Esperado: a mesma sugestão existente e um único registro de histórico na repetição, sem nova geração; acesso negado para a outra conta.

## UC-018 — Solicitar nova sugestão

**Status:** Proposto. **Prioridade:** Alta.

**Objetivo:** Obter outra alternativa para o mesmo pedido sem repetir os dados.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário solicita outra sugestão em um pedido que já possui resultado.

**Pré-condições:** Conta ativa; pedido próprio com pelo menos uma sugestão confirmada e fotografia preservada. A validade da sessão é conferida no UC-017.

**Entradas:** Identificador do pedido e nova chave da operação. O sistema recupera os modelos já usados e o contexto preservado.

### Fluxo principal

1. **Usuário:** solicita uma nova sugestão para o pedido.
2. **Sistema:** recebe somente a referência do pedido e a chave da operação e indica o modo alternativa, que exige excluir os modelos já usados; ainda não lê dados protegidos do pedido.
3. **Sistema:** inclui obrigatoriamente o UC-017, executando seus passos 2 a 9. Após autenticar e autorizar no passo 2 do caso incluído, recupera a fotografia, confirma que há resultado anterior e obtém os modelos já usados no passo 3; usa-os como exclusões na seleção do passo 6. A situação não é solicitada novamente.
4. **Usuário:** consulta a alternativa apresentada pelo UC-017; as sugestões anteriores continuam disponíveis no histórico.

### Fluxos alternativos

- **A1 — Sem alternativa disponível. Origem: passo 3, seleção do UC-017.** Após excluir os modelos anteriores, não resta candidato compatível. O sistema informa a indisponibilidade. **Retorno:** não retorna; encerra preservando as sugestões anteriores.
- **A2 — Repetição da mesma solicitação. Origem: passo 3, recuperação do comprovante ou persistência do UC-017.** A chave já foi confirmada. **Retorno:** passo 4, com a mesma alternativa existente, conforme A2 do UC-017.

### Exceções

- **E1 — Pedido sem sugestão anterior. Origem: passo 3, recuperação do pedido no passo 3 do UC-017.** O sistema orienta iniciar pelo UC-017. **Fim:** encerra sem geração alternativa.
- **E2 — Acesso, validação ou persistência rejeitados. Origem: passo 3.** Aplicam-se E1, E2 ou E3 do UC-017, incluindo conflito concorrente na gravação. **Fim:** encerra sem alterar os resultados anteriores; nenhuma alternativa parcial é apresentada.

**Pós-condição de sucesso:** Nova sugestão é registrada pelo UC-017 a partir da mesma fotografia, usando um modelo ainda não utilizado no pedido.

**Pós-condição de falha:** Sugestões anteriores permanecem acessíveis; não há novo registro incompleto.

**Rastreabilidade:** RF-017 e RF-018; RN-003, RN-010, RN-012, RN-013, RN-024; RNF-001, RNF-002, RNF-003, RNF-004, RNF-006, RNF-007. **Relacionamento:** UC-018 `<<include>>` UC-017.

### Critérios de aceite

- **CA-018.1:** A alternativa usa a fotografia preservada e exclui os modelos já usados no pedido.
- **CA-018.2:** A execução inclui a geração e a persistência do UC-017; a falta de candidatos mantém os resultados existentes.
- **CA-018.3:** Repetição da mesma chave não duplica a alternativa; duas operações concorrentes não confirmam o mesmo modelo como alternativas distintas para o pedido.

### Casos de teste propostos

- **CT-018.1 → CA-018.1:** Pedido com modelo M1 já usado; M2 compatível ainda disponível; perfil do usuário alterado após o pedido. Solicitar alternativa. Esperado: M2 com o contexto preservado do pedido, sem usar o perfil novo.
- **CT-018.2 → CA-018.2:** Catálogo contém somente M1, já usado. Solicitar alternativa. Esperado: mensagem de indisponibilidade, mesmo histórico e nenhuma nova sugestão.
- **CT-018.3 → CA-018.3:** Executar duas requisições com chaves diferentes, ambas selecionando M2 antes de qualquer confirmação, quando apenas M2 resta; repetir a chave vencedora. Esperado: uma operação confirma M2; a outra detecta conflito no passo 8 e executa E3 do UC-017 com rollback; a repetição retorna o resultado existente. Se a segunda seleção ocorrer após a primeira confirmação, aplica-se A1, pois já não há alternativa.

## UC-019 — Consultar histórico de sugestões

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Localizar sugestões próprias já geradas e seu contexto preservado.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário abre o histórico de sugestões.

**Pré-condições:** Conta ativa com sessão válida. Ter sugestões anteriores não é obrigatório.

**Entradas:** Filtros opcionais de período, categoria ou pedido; identificador da sugestão para consultar detalhes.

### Fluxo principal

1. **Usuário:** solicita o histórico e, opcionalmente, informa filtros.
2. **Sistema:** valida a sessão e restringe a consulta ao proprietário.
3. **Sistema:** recupera as sugestões confirmadas que correspondem aos filtros e apresenta data, pedido e texto, da mais recente para a mais antiga.
4. **Usuário:** seleciona uma sugestão da lista.
5. **Sistema:** verifica novamente a propriedade e apresenta o texto e a fotografia do pedido associada à geração.

### Fluxos alternativos

- **A1 — Lista vazia. Origem: passo 3.** O sistema informa que não há sugestões ou correspondências ao filtro. **Retorno:** passo 1, se o usuário alterar o filtro; caso contrário, encerra normalmente.
- **A2 — Apenas listar. Origem: passo 4.** O usuário encerra sem abrir detalhes. **Retorno:** não retorna; termina com a lista consultada.

### Exceções

- **E1 — Filtro inválido. Origem: passo 3.** Período final anterior ao inicial é rejeitado com orientação. **Fim:** não executa a consulta inválida; o usuário pode reiniciar no passo 1.
- **E2 — Sessão inválida ou sugestão alheia. Origem: passos 2 ou 5.** O sistema nega acesso aos dados protegidos. **Fim:** encerra sem exibir esses registros.
- **E3 — Consulta indisponível. Origem: passos 3 ou 5.** O sistema informa falha de acesso aos registros, distinguindo-a de uma lista vazia. **Fim:** encerra sem modificar dados.

**Pós-condição de sucesso:** Lista ou detalhe próprio consultado, inclusive lista vazia válida.

**Pós-condição de falha:** Histórico permanece inalterado e nenhum registro alheio é exposto.

**Rastreabilidade:** RF-019; RN-003, RN-012, RN-023; RNF-001, RNF-002, RNF-003, RNF-004.

### Critérios de aceite

- **CA-019.1:** A lista mostra somente sugestões confirmadas do usuário e respeita período, categoria e pedido selecionados.
- **CA-019.2:** Os detalhes preservam o texto e o contexto da geração, mesmo após mudanças posteriores no perfil ou no catálogo.
- **CA-019.3:** Lista vazia é informada como resultado válido; falha de consulta possui mensagem própria.

### Casos de teste propostos

- **CT-019.1 → CA-019.1:** Conta A tem sugestões em setembro e outubro; conta B tem sugestão em outubro. Consultar outubro na conta A. Esperado: somente os registros de outubro de A.
- **CT-019.2 → CA-019.2:** Gerar sugestão, alterar o perfil e o texto do modelo e reabrir seu detalhe. Esperado: texto e fotografia originais.
- **CT-019.3 → CA-019.3:** Consultar período sem registros e depois simular indisponibilidade do armazenamento. Esperado: lista vazia orientativa no primeiro caso e erro de consulta no segundo.

## UC-020 — Copiar sugestão

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Disponibilizar o texto de uma sugestão própria para uso em outro aplicativo.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário aciona “Copiar” em uma sugestão.

**Pré-condições:** Conta ativa com sessão válida; sugestão própria existente e exibida.

**Entradas:** Identificador da sugestão e ação explícita de cópia.

### Fluxo principal

1. **Usuário:** solicita a cópia da sugestão exibida.
2. **Sistema:** valida a sessão, a propriedade e a existência da sugestão.
3. **Sistema:** transfere o texto da sugestão para a área de transferência com a permissão do navegador.
4. **Sistema:** confirma que o texto foi copiado.

### Fluxos alternativos

- **A1 — Permissão solicitada pelo navegador. Origem: passo 3.** O usuário concede a permissão de escrita na área de transferência. **Retorno:** passo 3.

### Exceções

- **E1 — Área de transferência indisponível ou permissão negada. Origem: passo 3.** O sistema informa que a cópia automática não ocorreu e mantém o texto selecionável para cópia manual. **Fim:** encerra sem confirmação de cópia automática.
- **E2 — Sessão inválida, sugestão inexistente ou alheia. Origem: passo 2.** O sistema nega a operação sem apresentar texto protegido. **Fim:** encerra sem escrever na área de transferência.

**Pós-condição de sucesso:** Texto próprio está na área de transferência; nenhum pedido ou sugestão é alterado e nenhuma mensagem é enviada ao destinatário.

**Pós-condição de falha:** Registros permanecem inalterados e o sistema não afirma que a cópia foi concluída.

**Rastreabilidade:** RF-020; RN-003, RN-015; RNF-001, RNF-002, RNF-003, RNF-005.

### Critérios de aceite

- **CA-020.1:** A cópia contém exatamente o texto apresentado da sugestão própria.
- **CA-020.2:** A ação escreve somente na área de transferência, sem enviar e-mail ou mensagem ao destinatário.
- **CA-020.3:** Permissão negada apresenta orientação e não produz confirmação falsa.

### Casos de teste propostos

- **CT-020.1 → CA-020.1:** Copiar uma sugestão com acentos e duas linhas e colar em um editor. Esperado: conteúdo idêntico, incluindo acentos e quebras.
- **CT-020.2 → CA-020.2:** Copiar sugestão vinculada a destinatário cadastrado. Esperado: nenhuma chamada de envio de mensagem e nenhum registro de envio; texto disponível para colar.
- **CT-020.3 → CA-020.3:** Negar a permissão da área de transferência. Esperado: aviso de falha e texto selecionável, sem mensagem “Copiado”.

## UC-021 — Favoritar sugestão

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Guardar uma sugestão própria na coleção de favoritos.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário marca uma sugestão como favorita.

**Pré-condições:** Conta ativa com sessão válida; sugestão própria existente.

**Entradas:** Identificador da sugestão.

### Fluxo principal

1. **Usuário:** solicita favoritar a sugestão.
2. **Sistema:** valida a sessão e a propriedade da sugestão.
3. **Sistema:** verifica a existência do vínculo de favorito e registra um único vínculo entre usuário e sugestão.
4. **Sistema:** apresenta a sugestão como favorita.

### Fluxos alternativos

- **A1 — Já favorita. Origem: passo 3.** O vínculo já existe, inclusive após repetição da ação ou requisições simultâneas. **Retorno:** passo 4, sem duplicar registros.

### Exceções

- **E1 — Acesso inválido ou sugestão inexistente. Origem: passo 2.** O sistema nega a operação. **Fim:** encerra sem criar favorito.
- **E2 — Falha de gravação. Origem: passo 3.** O sistema informa que a marcação não foi concluída. **Fim:** preserva o estado anterior e não apresenta confirmação de favorito novo.

**Pós-condição de sucesso:** Existe exatamente um vínculo de favorito para a sugestão e o usuário.

**Pós-condição de falha:** Coleção anterior preservada; sugestão e histórico não são alterados.

**Rastreabilidade:** RF-021; RN-003, RN-014; RNF-001, RNF-002, RNF-003, RNF-006.

### Critérios de aceite

- **CA-021.1:** Somente o proprietário pode favoritar a sugestão.
- **CA-021.2:** Repetir a ação mantém um único vínculo, inclusive sob concorrência.
- **CA-021.3:** Falha de persistência mantém o estado anterior e não confirma a marcação.

### Casos de teste propostos

- **CT-021.1 → CA-021.1:** Conta A tenta favoritar uma sugestão da conta B por identificador. Esperado: acesso negado e nenhum vínculo criado.
- **CT-021.2 → CA-021.2:** Enviar duas solicitações simultâneas para favoritar a mesma sugestão própria. Esperado: um vínculo e estado final favorito.
- **CT-021.3 → CA-021.3:** Simular falha ao criar o vínculo. Esperado: aviso de falha, coleção inalterada e ausência de confirmação positiva.

## UC-022 — Consultar favoritos

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Localizar sugestões marcadas como favoritas.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário abre a coleção de favoritos.

**Pré-condições:** Conta ativa com sessão válida. A coleção pode estar vazia.

**Entradas:** Filtro opcional de categoria; identificador do favorito selecionado.

### Fluxo principal

1. **Usuário:** solicita a coleção de favoritos, com filtro opcional.
2. **Sistema:** valida a sessão e restringe a consulta aos vínculos do usuário.
3. **Sistema:** apresenta as sugestões favoritas que correspondem ao filtro, com texto, pedido e categoria preservados.
4. **Usuário:** seleciona uma sugestão favorita.
5. **Sistema:** verifica a propriedade e apresenta o detalhe da sugestão.

### Fluxos alternativos

- **A1 — Coleção vazia. Origem: passo 3.** O sistema informa a ausência de favoritos ou de resultados para o filtro. **Retorno:** passo 1, se o usuário mudar o filtro; caso contrário, termina normalmente.
- **A2 — Consulta apenas da lista. Origem: passo 4.** O usuário fecha a coleção. **Retorno:** não retorna; encerra sem abrir detalhe.

### Exceções

- **E1 — Sessão inválida ou favorito alheio. Origem: passos 2 ou 5.** O sistema nega acesso. **Fim:** encerra sem expor dados protegidos.
- **E2 — Consulta indisponível. Origem: passos 3 ou 5.** O sistema informa falha de consulta. **Fim:** encerra sem modificar a coleção.

**Pós-condição de sucesso:** Favoritos próprios consultados ou coleção vazia corretamente informada.

**Pós-condição de falha:** Favoritos, sugestões e histórico permanecem inalterados.

**Rastreabilidade:** RF-022; RN-003, RN-014, RN-023; RNF-001, RNF-002, RNF-003, RNF-004.

### Critérios de aceite

- **CA-022.1:** A coleção apresenta somente os favoritos do usuário autenticado.
- **CA-022.2:** A consulta por categoria respeita o filtro e permite abrir a sugestão própria.
- **CA-022.3:** A coleção vazia é um resultado válido, diferente de erro de consulta.

### Casos de teste propostos

- **CT-022.1 → CA-022.1:** A e B favoritam sugestões diferentes. Consultar como A. Esperado: somente os vínculos de A.
- **CT-022.2 → CA-022.2:** Marcar sugestões de duas categorias e filtrar uma delas. Esperado: apenas a categoria escolhida; detalhe com o texto armazenado.
- **CT-022.3 → CA-022.3:** Abrir coleção sem vínculos e depois repetir com falha de armazenamento. Esperado: orientação de coleção vazia e mensagem de erro distintas.

## UC-023 — Remover sugestão dos favoritos

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Retirar uma sugestão da coleção de favoritos mantendo o histórico.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário desmarca uma sugestão favorita.

**Pré-condições:** Conta ativa com sessão válida; sugestão própria existente. A ausência do vínculo de favorito é tratada de modo idempotente.

**Entradas:** Identificador da sugestão.

### Fluxo principal

1. **Usuário:** solicita remover a sugestão dos favoritos.
2. **Sistema:** valida a sessão e a propriedade da sugestão.
3. **Sistema:** remove somente o vínculo de favorito do usuário com a sugestão.
4. **Sistema:** apresenta a sugestão como não favorita e atualiza a coleção.

### Fluxos alternativos

- **A1 — Vínculo já ausente. Origem: passo 3.** A remoção já ocorreu ou a sugestão não estava favorita. **Retorno:** passo 4, confirmando o estado final sem erro nem alteração da sugestão.

### Exceções

- **E1 — Acesso inválido ou sugestão inexistente. Origem: passo 2.** O sistema nega a operação. **Fim:** encerra sem remoção de vínculos alheios.
- **E2 — Falha de gravação. Origem: passo 3.** O sistema informa que a remoção não foi concluída. **Fim:** preserva o vínculo anterior e não confirma a retirada.

**Pós-condição de sucesso:** O vínculo de favorito está ausente; sugestão e histórico continuam disponíveis.

**Pós-condição de falha:** Coleção anterior preservada, sem alteração dos dados da sugestão.

**Rastreabilidade:** RF-023; RN-003, RN-014; RNF-001, RNF-002, RNF-003, RNF-006.

### Critérios de aceite

- **CA-023.1:** A operação remove apenas o vínculo do proprietário.
- **CA-023.2:** Sugestão e histórico permanecem consultáveis após a retirada.
- **CA-023.3:** Repetir a remoção mantém o estado não favorito sem erro de duplicidade ou exclusão adicional.

### Casos de teste propostos

- **CT-023.1 → CA-023.1:** A tenta remover dos favoritos uma sugestão de B. Esperado: acesso negado; vínculo de B preservado.
- **CT-023.2 → CA-023.2:** Remover sugestão própria e consultar seu histórico. Esperado: sugestão com o mesmo texto ainda acessível e ausente da coleção de favoritos.
- **CT-023.3 → CA-023.3:** Solicitar a remoção duas vezes. Esperado: estado não favorito nas duas respostas; nenhuma exclusão da sugestão.

## UC-024 — Avaliar sugestão

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Registrar uma avaliação opcional de sugestão própria, com nota obrigatória de 1 a 5 e comentário opcional.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário decide avaliar uma sugestão. A avaliação é opcional e não condiciona geração, cópia ou consulta.

**Pré-condições:** Conta ativa com sessão válida; sugestão própria existente, sem avaliação anterior.

**Entradas:** Identificador da sugestão; nota inteira de 1 a 5; comentário opcional de até 500 caracteres.

### Fluxo principal

1. **Usuário:** solicita avaliar a sugestão.
2. **Sistema:** valida a sessão, a propriedade e a ausência de avaliação anterior.
3. **Sistema:** apresenta campos para nota e comentário opcional.
4. **Usuário:** informa a nota, opcionalmente um comentário, e confirma.
5. **Sistema:** valida a nota e o tamanho do comentário.
6. **Sistema:** registra uma única avaliação vinculada à sugestão e ao proprietário.
7. **Sistema:** apresenta a avaliação registrada.

### Fluxos alternativos

- **A1 — Avaliação já existente. Origem: passo 2.** O sistema apresenta a avaliação atual e orienta sua edição pelo UC-025. **Retorno:** não retorna; encerra sem duplicar avaliação.
- **A2 — Cancelamento. Origem: passo 4.** O usuário cancela o formulário. **Retorno:** não retorna; encerra sem gravar.

### Exceções

- **E1 — Nota ou comentário inválido. Origem: passo 5.** O sistema informa a exigência de nota inteira entre 1 e 5 ou limite de 500 caracteres. **Fim:** não grava; permite nova tentativa no passo 4.
- **E2 — Acesso inválido ou sugestão inexistente. Origem: passo 2.** O sistema nega a avaliação. **Fim:** encerra sem expor ou alterar a sugestão.
- **E3 — Falha de persistência ou criação concorrente. Origem: passo 6.** O sistema não confirma novo registro parcial; em disputa, informa que uma avaliação já existe. **Fim:** preserva o estado confirmado e orienta consultar ou atualizar pelo UC-025.

**Pós-condição de sucesso:** Uma avaliação válida é vinculada à sugestão; texto e histórico da sugestão permanecem iguais.

**Pós-condição de falha:** Nenhuma avaliação inválida ou duplicada é criada; sugestões anteriores permanecem íntegras.

**Rastreabilidade:** RF-024; RN-003, RN-016; RNF-001, RNF-002, RNF-003, RNF-006.

### Critérios de aceite

- **CA-024.1:** A nota é obrigatória quando o caso é invocado, inteira de 1 a 5; comentário é opcional e limitado a 500 caracteres.
- **CA-024.2:** Existe no máximo uma avaliação por sugestão e apenas seu proprietário pode registrá-la.
- **CA-024.3:** O usuário pode cancelar ou não avaliar, mantendo acesso à sugestão e às demais operações.

### Casos de teste propostos

- **CT-024.1 → CA-024.1:** Registrar nota 5 sem comentário e tentar nota 0, nota 2,5 e comentário de 501 caracteres em outras sugestões. Esperado: primeira aceita; entradas inválidas rejeitadas sem gravação.
- **CT-024.2 → CA-024.2:** Avaliar uma sugestão duas vezes e tentar avaliar a sugestão de outra conta. Esperado: um registro; orientação de atualização na repetição; acesso negado à conta alheia.
- **CT-024.3 → CA-024.3:** Abrir o formulário, cancelar e copiar a sugestão. Esperado: nenhuma avaliação e cópia disponível.

## UC-025 — Atualizar avaliação

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Corrigir uma avaliação anteriormente registrada.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário solicita editar uma avaliação própria.

**Pré-condições:** Conta ativa com sessão válida; sugestão própria e avaliação existente.

**Entradas:** Identificador da avaliação; nova nota inteira de 1 a 5 e comentário opcional de até 500 caracteres.

### Fluxo principal

1. **Usuário:** solicita editar a avaliação.
2. **Sistema:** valida a sessão, a propriedade da sugestão e a existência da avaliação.
3. **Sistema:** apresenta os valores atuais.
4. **Usuário:** altera nota ou comentário e confirma a atualização.
5. **Sistema:** valida os novos valores segundo os limites da avaliação.
6. **Sistema:** substitui os valores da avaliação existente sem criar outro vínculo.
7. **Sistema:** apresenta os valores confirmados.

### Fluxos alternativos

- **A1 — Cancelamento. Origem: passo 4.** O usuário cancela a edição. **Retorno:** não retorna; encerra mantendo os valores anteriores.
- **A2 — Comentário removido. Origem: passo 4.** O usuário limpa o comentário e conserva nota válida. **Retorno:** passo 5, tratando o comentário como ausente.

### Exceções

- **E1 — Valores inválidos. Origem: passo 5.** O sistema identifica nota fora de 1 a 5, não inteira, ou comentário acima de 500 caracteres. **Fim:** mantém a avaliação anterior e permite nova tentativa no passo 4.
- **E2 — Sessão inválida, avaliação alheia ou ausente. Origem: passo 2.** O sistema nega acesso ou informa que não há avaliação a editar; neste último caso, orienta UC-024. **Fim:** encerra sem criar avaliação implicitamente.
- **E3 — Falha de gravação. Origem: passo 6.** O sistema informa que a atualização não foi concluída. **Fim:** mantém os valores confirmados anteriormente.

**Pós-condição de sucesso:** A mesma avaliação contém os novos valores válidos; sugestão permanece inalterada.

**Pós-condição de falha:** Avaliação anterior preservada; nenhum vínculo adicional é criado.

**Rastreabilidade:** RF-025; RN-003, RN-016; RNF-001, RNF-002, RNF-003, RNF-006.

### Critérios de aceite

- **CA-025.1:** Atualizar substitui os valores da avaliação própria e mantém um único registro por sugestão.
- **CA-025.2:** É possível remover o comentário mantendo uma nota válida.
- **CA-025.3:** Dados inválidos ou falha de gravação preservam os valores anteriores.

### Casos de teste propostos

- **CT-025.1 → CA-025.1:** Alterar nota 2 para 4 em avaliação própria. Esperado: mesmo identificador, nota 4 e um registro; tentar com outra conta deve ser negado.
- **CT-025.2 → CA-025.2:** Limpar um comentário de 30 caracteres e manter nota 3. Esperado: comentário ausente e nota 3.
- **CT-025.3 → CA-025.3:** Tentar comentário de 501 caracteres; depois induzir falha ao salvar nota 5 válida. Esperado: nota e comentário originais preservados nos dois cenários.

## UC-026 — Excluir avaliação

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Remover uma avaliação própria mantendo a sugestão.

**Ator principal:** Usuário. **Atores secundários:** Nenhum.

**Gatilho:** O usuário solicita excluir sua avaliação.

**Pré-condições:** Conta ativa com sessão válida; sugestão própria com avaliação existente.

**Entradas:** Identificador da avaliação e confirmação de exclusão.

### Fluxo principal

1. **Usuário:** solicita excluir a avaliação.
2. **Sistema:** valida a sessão, a propriedade da sugestão e a existência da avaliação.
3. **Sistema:** apresenta nota e comentário da avaliação e solicita confirmação.
4. **Usuário:** confirma a exclusão.
5. **Sistema:** remove somente a avaliação.
6. **Sistema:** confirma a exclusão e mantém a sugestão disponível.

### Fluxos alternativos

- **A1 — Cancelamento. Origem: passo 4.** O usuário cancela a exclusão. **Retorno:** não retorna; encerra mantendo a avaliação.

### Exceções

- **E1 — Acesso inválido ou avaliação ausente. Origem: passo 2.** O sistema nega acesso ou informa que não há avaliação a excluir. **Fim:** encerra sem modificar registros.
- **E2 — Falha de gravação. Origem: passo 5.** O sistema informa que a exclusão não foi concluída. **Fim:** mantém a avaliação anterior e não confirma remoção.

**Pós-condição de sucesso:** Avaliação removida; sugestão, favoritos e histórico preservados. Uma nova avaliação pode ser criada pelo UC-024.

**Pós-condição de falha:** Avaliação e demais registros permanecem inalterados.

**Rastreabilidade:** RF-026; RN-003, RN-016; RNF-001, RNF-002, RNF-003, RNF-006.

### Critérios de aceite

- **CA-026.1:** Somente o proprietário pode confirmar a exclusão de sua avaliação.
- **CA-026.2:** A exclusão não remove a sugestão, seu histórico ou seus favoritos.
- **CA-026.3:** Cancelamento e falha de gravação mantêm a avaliação anterior.

### Casos de teste propostos

- **CT-026.1 → CA-026.1:** Tentar excluir a avaliação de outra conta e depois excluir avaliação própria com confirmação. Esperado: primeira negada; segunda removida.
- **CT-026.2 → CA-026.2:** Excluir avaliação de sugestão favorita e consultar o histórico e favoritos. Esperado: sugestão ainda disponível em ambos e sem avaliação.
- **CT-026.3 → CA-026.3:** Cancelar uma exclusão e simular falha de armazenamento em outra tentativa. Esperado: avaliação preservada nas duas tentativas.

## UC-027 — Cadastrar categoria

**Status:** Proposto. **Prioridade:** Alta.

**Objetivo:** Disponibilizar uma nova categoria para classificar pedidos e modelos.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador solicita cadastrar uma categoria.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador.

**Entradas:** Nome único e descrição da categoria.

### Fluxo principal

1. **Administrador:** solicita cadastrar uma categoria.
2. **Sistema:** valida a sessão e o papel administrativo e apresenta o formulário.
3. **Administrador:** informa nome e descrição e confirma.
4. **Sistema:** valida os dados e verifica a unicidade do nome.
5. **Sistema:** registra a categoria com estado ativo.
6. **Sistema:** apresenta os dados da categoria criada.

### Fluxos alternativos

- **A1 — Cancelamento. Origem: passo 3.** O administrador cancela o formulário. **Retorno:** não retorna; encerra sem cadastrar.

### Exceções

- **E1 — Nome vazio, duplicado ou descrição ausente. Origem: passo 4.** O sistema identifica os campos a corrigir. **Fim:** não grava; permite nova tentativa no passo 3.
- **E2 — Sessão inválida ou papel insuficiente. Origem: passo 2.** O sistema nega a operação administrativa. **Fim:** encerra sem cadastro.
- **E3 — Falha de gravação ou disputa pelo mesmo nome. Origem: passo 5.** O sistema informa a falha ou a duplicidade, mantendo a unicidade. **Fim:** não confirma categoria adicional.

**Pós-condição de sucesso:** Categoria ativa, com nome único, disponível para novos pedidos e modelos.

**Pós-condição de falha:** Catálogo permanece sem registro inválido ou duplicado.

**Rastreabilidade:** RF-027; RN-002, RN-017; RNF-001, RNF-003, RNF-006, RNF-008.

### Critérios de aceite

- **CA-027.1:** O cadastro exige papel Administrador e dados válidos.
- **CA-027.2:** O catálogo não admite duas categorias com o mesmo nome.
- **CA-027.3:** A categoria criada é ativa e fica disponível para novos pedidos e modelos.

### Casos de teste propostos

- **CT-027.1 → CA-027.1:** Uma conta comum tenta criar categoria; administrador tenta nome vazio. Esperado: acesso negado no primeiro caso e validação sem gravação no segundo.
- **CT-027.2 → CA-027.2:** Cadastrar “Compromisso social” duas vezes, incluindo solicitações concorrentes. Esperado: uma categoria e aviso de duplicidade.
- **CT-027.3 → CA-027.3:** Cadastrar “Compromisso acadêmico” com descrição válida. Esperado: estado ativo e categoria selecionável em novo pedido e novo modelo.

## UC-028 — Consultar categorias

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Localizar categorias ativas ou inativas e consultar seus dados.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador abre o catálogo de categorias.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador.

**Entradas:** Filtros opcionais de nome e estado; identificador para consulta de detalhes.

### Fluxo principal

1. **Administrador:** solicita a lista de categorias e informa filtros opcionais.
2. **Sistema:** valida a sessão e o papel administrativo.
3. **Sistema:** recupera as categorias que correspondem aos filtros e apresenta nome e estado.
4. **Administrador:** seleciona uma categoria.
5. **Sistema:** apresenta nome, descrição e estado da categoria.

### Fluxos alternativos

- **A1 — Nenhum resultado. Origem: passo 3.** O sistema apresenta lista vazia e orientação para alterar filtros ou cadastrar categoria. **Retorno:** passo 1, se os filtros forem alterados; caso contrário, encerra normalmente.
- **A2 — Consulta apenas da lista. Origem: passo 4.** O administrador encerra sem abrir detalhes. **Retorno:** não retorna; encerra normalmente.

### Exceções

- **E1 — Sessão inválida ou papel insuficiente. Origem: passo 2.** O sistema nega acesso à consulta administrativa. **Fim:** encerra sem apresentar a lista administrativa.
- **E2 — Consulta indisponível ou categoria inexistente. Origem: passos 3 ou 5.** O sistema informa erro de consulta ou ausência do registro. **Fim:** encerra sem alterar catálogo.

**Pós-condição de sucesso:** Lista ou detalhe consultado, incluindo categorias inativas e lista vazia válida.

**Pós-condição de falha:** Catálogo permanece inalterado.

**Rastreabilidade:** RF-028; RN-002, RN-017, RN-023; RNF-001, RNF-003, RNF-004, RNF-008.

### Critérios de aceite

- **CA-028.1:** A consulta administrativa exige o papel Administrador.
- **CA-028.2:** Filtro por estado diferencia categorias ativas e inativas e permite consultar seus dados.
- **CA-028.3:** Nenhum resultado é apresentado como lista vazia orientativa, sem erro de persistência.

### Casos de teste propostos

- **CT-028.1 → CA-028.1:** Abrir a lista administrativa com conta comum. Esperado: acesso negado.
- **CT-028.2 → CA-028.2:** Cadastrar duas categorias e desativar uma; filtrar estado inativo e abrir detalhe. Esperado: somente a inativa, com nome e descrição preservados.
- **CT-028.3 → CA-028.3:** Buscar nome inexistente. Esperado: lista vazia com orientação para alterar o filtro.

## UC-029 — Atualizar categoria

**Status:** Proposto. **Prioridade:** Alta.

**Objetivo:** Corrigir nome e descrição ou reativar uma categoria.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador solicita editar uma categoria existente.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador; categoria existente.

**Entradas:** Identificador da categoria; novo nome, descrição e, quando aplicável, solicitação de reativação. Desativação é realizada pelo UC-030.

### Fluxo principal

1. **Administrador:** solicita editar a categoria.
2. **Sistema:** valida a sessão e o papel administrativo e recupera os dados atuais.
3. **Administrador:** altera nome ou descrição e confirma.
4. **Sistema:** valida os dados e verifica a unicidade do nome entre as demais categorias.
5. **Sistema:** grava as alterações preservando o identificador e os vínculos existentes.
6. **Sistema:** apresenta os dados confirmados.

### Fluxos alternativos

- **A1 — Reativação. Origem: passo 3.** A categoria está inativa e o administrador solicita ativá-la. O sistema inclui o estado ativo na atualização. **Retorno:** passo 4; a ativação de modelos é uma decisão separada do UC-033.
- **A2 — Cancelamento. Origem: passo 3.** O administrador cancela a edição. **Retorno:** não retorna; encerra sem alterar a categoria.

### Exceções

- **E1 — Dados inválidos ou nome duplicado. Origem: passo 4.** O sistema aponta os campos a corrigir. **Fim:** mantém os dados anteriores e permite nova tentativa no passo 3.
- **E2 — Acesso insuficiente ou categoria inexistente. Origem: passo 2.** O sistema nega acesso ou informa indisponibilidade do registro. **Fim:** encerra sem alteração.
- **E3 — Falha de gravação. Origem: passo 5.** O sistema informa que a atualização não foi concluída. **Fim:** mantém os valores anteriores.

**Pós-condição de sucesso:** Categoria atualizada; em reativação, fica novamente elegível para novos pedidos e modelos. Sugestões históricas conservam seu texto e fotografia.

**Pós-condição de falha:** Categoria e vínculos anteriores permanecem íntegros.

**Rastreabilidade:** RF-029; RN-002, RN-017; RNF-001, RNF-003, RNF-006, RNF-008.

### Critérios de aceite

- **CA-029.1:** Nome continua único e o identificador da categoria é preservado.
- **CA-029.2:** Reativar a categoria não altera sugestões históricas nem reativa implicitamente modelos desativados.
- **CA-029.3:** Dados inválidos, cancelamento ou falha de gravação preservam a categoria anterior.

### Casos de teste propostos

- **CT-029.1 → CA-029.1:** Renomear uma categoria para nome livre e depois para nome de outra categoria. Esperado: primeira atualização conserva o identificador; segunda é rejeitada.
- **CT-029.2 → CA-029.2:** Reativar categoria com modelo desativado e sugestão histórica. Esperado: categoria ativa; modelo ainda desativado; texto histórico igual.
- **CT-029.3 → CA-029.3:** Alterar a descrição e cancelar; depois simular falha na gravação da mesma alteração. Esperado: descrição anterior preservada nos dois casos.

## UC-030 — Desativar categoria

**Status:** Proposto. **Prioridade:** Alta.

**Objetivo:** Impedir novas gerações na categoria preservando os registros históricos.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador solicita desativar uma categoria.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador; categoria existente.

**Entradas:** Identificador da categoria e confirmação da desativação.

### Fluxo principal

1. **Administrador:** solicita desativar a categoria.
2. **Sistema:** valida a sessão e o papel administrativo e recupera a categoria.
3. **Sistema:** informa que a categoria deixará de permitir novas gerações e cadastro ou ativação de modelos, preservando os históricos, e solicita confirmação.
4. **Administrador:** confirma a desativação.
5. **Sistema:** registra o estado inativo sem excluir fisicamente a categoria ou seus vínculos.
6. **Sistema:** confirma o estado inativo.

### Fluxos alternativos

- **A1 — Categoria já inativa. Origem: passo 2.** O sistema informa o estado atual. **Retorno:** passo 6, sem nova alteração.
- **A2 — Cancelamento. Origem: passo 4.** O administrador cancela. **Retorno:** não retorna; encerra mantendo o estado anterior.

### Exceções

- **E1 — Acesso insuficiente ou categoria inexistente. Origem: passo 2.** O sistema nega acesso ou informa ausência do registro. **Fim:** encerra sem alteração.
- **E2 — Falha de gravação. Origem: passo 5.** O sistema informa que a desativação não foi concluída. **Fim:** preserva o estado anterior e não confirma a mudança.

**Pós-condição de sucesso:** Categoria inativa impede novas gerações e cadastro ou ativação de modelos; pedidos e sugestões históricos permanecem consultáveis. Modelos nela deixam de ser elegíveis enquanto a categoria está inativa.

**Pós-condição de falha:** Estado anterior mantido; nenhum histórico é excluído.

**Rastreabilidade:** RF-030; RN-002, RN-017; RNF-001, RNF-003, RNF-006, RNF-008.

### Critérios de aceite

- **CA-030.1:** A mudança exige Administrador e confirmação explícita.
- **CA-030.2:** Categoria inativa bloqueia novas gerações e cadastro ou ativação de modelos nela.
- **CA-030.3:** Desativar não remove pedidos, sugestões, avaliações ou textos históricos.

### Casos de teste propostos

- **CT-030.1 → CA-030.1:** Conta comum tenta desativar; administrador cancela a confirmação. Esperado: nenhum dos casos altera o estado.
- **CT-030.2 → CA-030.2:** Desativar categoria e tentar gerar para ela, cadastrar modelo e ativar modelo existente nela. Esperado: três operações rejeitadas com orientação.
- **CT-030.3 → CA-030.3:** Desativar categoria com sugestão avaliada e favorita. Esperado: sugestão, avaliação, favorito e fotografia continuam consultáveis pelo proprietário.

## UC-031 — Cadastrar modelo de desculpa

**Status:** Proposto. **Prioridade:** Alta.

**Objetivo:** Adicionar um modelo elegível para personalização.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador solicita cadastrar um modelo de desculpa.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador; pelo menos uma categoria ativa disponível.

**Entradas:** Texto do modelo, categoria ativa, pelo menos um tipo de destinatário e uma proximidade permitidos; tags de contexto opcionais. Parâmetros opcionais limitados a `nome_destinatario`, `situacao`, `contexto` e `nome_usuario`.

### Fluxo principal

1. **Administrador:** solicita cadastrar um modelo.
2. **Sistema:** valida a sessão e o papel administrativo e apresenta categorias ativas e campos do modelo.
3. **Administrador:** informa texto, categoria, tipos de destinatário, proximidades e tags opcionais de contexto e confirma.
4. **Sistema:** valida texto, categoria ainda ativa, compatibilidades e parâmetros permitidos; normaliza tags, removendo termos vazios e duplicatas.
5. **Sistema:** registra o modelo com estado ativo, suas compatibilidades e tags de contexto.
6. **Sistema:** apresenta o modelo confirmado e disponível para seleção compatível.

### Fluxos alternativos

- **A1 — Texto sem parâmetros. Origem: passo 4.** O texto é válido sem substituições. **Retorno:** passo 5, cadastrando-o normalmente.
- **A2 — Cancelamento. Origem: passo 3.** O administrador cancela o formulário. **Retorno:** não retorna; encerra sem cadastrar.

### Exceções

- **E1 — Dados ou parâmetros inválidos. Origem: passo 4.** O sistema rejeita texto vazio, categoria inativa, ausência de tipo ou proximidade, valores fora do domínio ou parâmetro não permitido. **Fim:** não grava; permite corrigir no passo 3.
- **E2 — Acesso insuficiente. Origem: passo 2.** O sistema nega o cadastro. **Fim:** encerra sem criar modelo.
- **E3 — Falha de gravação ou categoria desativada antes da confirmação. Origem: passo 5.** O sistema não confirma o cadastro ativo. **Fim:** encerra sem modelo parcialmente persistido ou elegível em categoria inativa.

**Pós-condição de sucesso:** Modelo ativo, com categoria ativa e compatibilidades válidas, disponível para geração.

**Pós-condição de falha:** Nenhum modelo incompleto ou inválido fica disponível no catálogo.

**Rastreabilidade:** RF-031; RN-002, RN-011, RN-017, RN-018; RNF-001, RNF-003, RNF-006, RNF-008.

### Critérios de aceite

- **CA-031.1:** O modelo exige categoria ativa, texto e ao menos um tipo e uma proximidade válidos; tags são opcionais e persistidas sem termos vazios ou duplicados.
- **CA-031.2:** Somente os quatro parâmetros permitidos são aceitos; texto sem parâmetros também é válido.
- **CA-031.3:** Somente Administrador cadastra; categoria desativada não permite cadastro ativo.

### Casos de teste propostos

- **CT-031.1 → CA-031.1:** Cadastrar modelo para amigo/alta e categoria ativa, com tags `Acadêmica`, `acadêmica` e termo vazio; tentar outro sem proximidade. Esperado: primeiro disponível com uma tag normalizada `acadêmica`; segundo rejeitado sem registro.
- **CT-031.2 → CA-031.2:** Cadastrar texto com `{situacao}`; tentar `{telefone_destinatario}`; cadastrar texto sem parâmetros. Esperado: primeiro e terceiro aceitos; segundo rejeitado.
- **CT-031.3 → CA-031.3:** Tentar cadastro com conta comum; depois desativar a categoria entre preenchimento e confirmação administrativa. Esperado: nenhum modelo ativo criado nas duas tentativas.

## UC-032 — Consultar modelos de desculpa

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Localizar e consultar modelos do catálogo.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador abre o catálogo de modelos.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador.

**Entradas:** Filtros opcionais de categoria, estado, tipo de destinatário e proximidade; identificador de modelo para detalhe.

### Fluxo principal

1. **Administrador:** solicita a lista de modelos e informa filtros opcionais.
2. **Sistema:** valida a sessão e o papel administrativo.
3. **Sistema:** recupera os modelos correspondentes e apresenta texto, categoria e estado.
4. **Administrador:** seleciona um modelo.
5. **Sistema:** apresenta texto, parâmetros, categoria, estado, compatibilidades e tags de contexto do modelo, indicando também o estado da categoria.

### Fluxos alternativos

- **A1 — Nenhum resultado. Origem: passo 3.** O sistema apresenta lista vazia com orientação para alterar filtros ou cadastrar modelo. **Retorno:** passo 1, se houver mudança de filtros; caso contrário, encerra normalmente.
- **A2 — Consulta apenas da lista. Origem: passo 4.** O administrador fecha a lista sem abrir detalhes. **Retorno:** não retorna; encerra normalmente.

### Exceções

- **E1 — Acesso insuficiente. Origem: passo 2.** O sistema nega a consulta administrativa. **Fim:** encerra sem apresentar registros administrativos.
- **E2 — Consulta indisponível ou modelo inexistente. Origem: passos 3 ou 5.** O sistema informa a falha ou ausência do registro. **Fim:** encerra sem modificar o catálogo.

**Pós-condição de sucesso:** Lista ou detalhe do catálogo consultado; modelos inativos continuam acessíveis ao administrador.

**Pós-condição de falha:** Catálogo permanece inalterado.

**Rastreabilidade:** RF-032; RN-002, RN-018, RN-023; RNF-001, RNF-003, RNF-004, RNF-008.

### Critérios de aceite

- **CA-032.1:** A consulta administrativa exige papel Administrador.
- **CA-032.2:** Filtros selecionam modelos pelos dados do catálogo; detalhes informam compatibilidades e estado da categoria.
- **CA-032.3:** Modelos inativos são consultáveis; ausência de correspondência é uma lista vazia válida.

### Casos de teste propostos

- **CT-032.1 → CA-032.1:** Conta comum tenta abrir o catálogo administrativo por URL. Esperado: acesso negado.
- **CT-032.2 → CA-032.2:** Ter modelos para amigo/alta e superior/baixa; filtrar amigo/alta e abrir detalhe. Esperado: somente modelo compatível e todos os campos do catálogo apresentados.
- **CT-032.3 → CA-032.3:** Desativar um modelo, localizá-lo por estado inativo e depois buscar categoria sem modelos. Esperado: modelo inativo consultável e segunda consulta vazia orientativa.

## UC-033 — Atualizar modelo de desculpa

**Status:** Proposto. **Prioridade:** Alta.

**Objetivo:** Corrigir texto e compatibilidades ou reativar um modelo.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador solicita editar um modelo existente.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador; modelo existente. Para gravar a atualização, a categoria selecionada deve estar ativa.

**Entradas:** Identificador do modelo; texto, categoria ativa, tipos de destinatário, proximidades, tags de contexto opcionais e solicitação opcional de reativação. Desativação é realizada pelo UC-034.

### Fluxo principal

1. **Administrador:** solicita editar o modelo.
2. **Sistema:** valida a sessão e o papel administrativo e apresenta os dados atuais.
3. **Administrador:** altera texto, categoria, compatibilidades ou tags de contexto e confirma.
4. **Sistema:** valida texto, categoria ativa, pelo menos um tipo e uma proximidade e os parâmetros permitidos; normaliza tags e remove termos vazios ou duplicados.
5. **Sistema:** grava os novos dados do modelo, preservando seu identificador e o texto já armazenado nas sugestões históricas.
6. **Sistema:** apresenta os valores confirmados.

### Fluxos alternativos

- **A1 — Reativação. Origem: passo 3.** O administrador solicita tornar ativo um modelo inativo. **Retorno:** passo 4, exigindo categoria ativa antes da mudança.
- **A2 — Categoria atual inativa. Origem: passo 3.** O administrador seleciona uma categoria ativa alternativa ou reativa a categoria pelo UC-029 e reinicia a edição. **Retorno:** passo 4 na primeira opção; passo 1 na segunda.
- **A3 — Cancelamento. Origem: passo 3.** O administrador cancela a edição. **Retorno:** não retorna; encerra sem alterações.

### Exceções

- **E1 — Dados inválidos ou categoria inativa. Origem: passo 4.** O sistema rejeita texto vazio, parâmetro proibido ou compatibilidade incompleta. **Fim:** mantém o modelo anterior e permite nova tentativa no passo 3.
- **E2 — Acesso insuficiente ou modelo inexistente. Origem: passo 2.** O sistema nega acesso ou informa ausência do registro. **Fim:** encerra sem alterar o catálogo.
- **E3 — Falha de gravação ou categoria desativada antes da confirmação. Origem: passo 5.** O sistema não confirma a atualização. **Fim:** preserva os valores anteriores e não ativa o modelo em categoria inativa.

**Pós-condição de sucesso:** Modelo atualizado ou reativado com dados válidos; sugestões históricas mantêm o texto gerado anteriormente.

**Pós-condição de falha:** Modelo anterior preservado, sem alteração de sugestões existentes.

**Rastreabilidade:** RF-033; RN-002, RN-011, RN-017, RN-018, RN-019; RNF-001, RNF-003, RNF-006, RNF-008.

### Critérios de aceite

- **CA-033.1:** Atualização conserva o identificador e valida os mesmos campos e parâmetros do cadastro.
- **CA-033.2:** Alterar texto ou categoria não modifica sugestões já geradas.
- **CA-033.3:** Reativação exige categoria ativa; categoria inativa impede a confirmação.

### Casos de teste propostos

- **CT-033.1 → CA-033.1:** Alterar compatibilidade para amigo/alta com texto válido e substituir as tags por `trabalho`; consultar o modelo pelo UC-032; tentar parâmetro `{cpf}` na edição seguinte. Esperado: primeira salva com mesmo identificador e tags atualizadas visíveis no detalhe; segunda rejeitada sem mudança.
- **CT-033.2 → CA-033.2:** Gerar sugestão com modelo M1, alterar seu texto e categoria e consultar a sugestão antiga. Esperado: texto e fotografia históricos preservados; nova geração usa os dados atuais elegíveis.
- **CT-033.3 → CA-033.3:** Solicitar reativação de modelo em categoria inativa. Esperado: rejeição; após categoria reativada pelo UC-029, edição pode ativar o modelo.

## UC-034 — Desativar modelo de desculpa

**Status:** Proposto. **Prioridade:** Alta.

**Objetivo:** Retirar um modelo das próximas gerações mantendo o histórico.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador solicita desativar um modelo.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador; modelo existente.

**Entradas:** Identificador do modelo e confirmação da desativação.

### Fluxo principal

1. **Administrador:** solicita desativar o modelo.
2. **Sistema:** valida a sessão e o papel administrativo e recupera o modelo.
3. **Sistema:** informa que o modelo deixará de participar das próximas gerações, preservando sugestões já produzidas, e solicita confirmação.
4. **Administrador:** confirma a desativação.
5. **Sistema:** registra o estado inativo, sem excluir o modelo ou os textos das sugestões.
6. **Sistema:** confirma a desativação.

### Fluxos alternativos

- **A1 — Modelo já inativo. Origem: passo 2.** O sistema informa o estado atual. **Retorno:** passo 6, sem nova mudança.
- **A2 — Cancelamento. Origem: passo 4.** O administrador cancela. **Retorno:** não retorna; encerra mantendo o estado anterior.

### Exceções

- **E1 — Acesso insuficiente ou modelo inexistente. Origem: passo 2.** O sistema nega acesso ou informa ausência do registro. **Fim:** encerra sem alterar o catálogo.
- **E2 — Falha de gravação. Origem: passo 5.** O sistema informa que a desativação não foi concluída. **Fim:** conserva o estado anterior e não confirma a mudança.

**Pós-condição de sucesso:** Modelo inativo não participa das próximas seleções; continua disponível para consulta administrativa e vinculado aos históricos existentes.

**Pós-condição de falha:** Estado anterior e sugestões históricas preservados.

**Rastreabilidade:** RF-034; RN-002, RN-019; RNF-001, RNF-003, RNF-006, RNF-008.

### Critérios de aceite

- **CA-034.1:** A mudança exige papel Administrador e confirmação.
- **CA-034.2:** Modelo inativo deixa de ser candidato em novas gerações.
- **CA-034.3:** Sugestões, avaliações e favoritos que já referenciam o modelo permanecem preservados.

### Casos de teste propostos

- **CT-034.1 → CA-034.1:** Conta comum tenta desativar; administrador cancela a confirmação. Esperado: estado anterior nos dois cenários.
- **CT-034.2 → CA-034.2:** Desativar um modelo compatível e solicitar nova geração quando outro compatível ativo existe. Esperado: somente o modelo ativo pode ser selecionado.
- **CT-034.3 → CA-034.3:** Desativar modelo de sugestão avaliada e favorita e reabrir o histórico. Esperado: mesmo texto, avaliação e favorito, sem exclusão de registros.

## UC-035 — Consultar avaliações recebidas

**Status:** Proposto. **Prioridade:** Média.

**Objetivo:** Consultar avaliações do catálogo sem expor os dados pessoais dos pedidos.

**Ator principal:** Administrador. **Atores secundários:** Nenhum.

**Gatilho:** O administrador abre a consulta de avaliações recebidas.

**Pré-condições:** Conta ativa com sessão válida e papel Administrador. A existência de avaliações não é obrigatória.

**Entradas:** Filtros opcionais de categoria, modelo, nota e período.

### Fluxo principal

1. **Administrador:** solicita consultar as avaliações e informa filtros opcionais.
2. **Sistema:** valida a sessão e o papel administrativo.
3. **Sistema:** valida os filtros e recupera as avaliações correspondentes por uma projeção limitada aos campos permitidos.
4. **Sistema:** apresenta categoria, modelo, nota, comentário e data, sem campos estruturados de nome, e-mail, destinatário ou contexto pessoal, nem acesso aos pedidos dos usuários.
5. **Administrador:** consulta os resultados ou altera os filtros.

### Fluxos alternativos

- **A1 — Nenhum resultado. Origem: passo 4.** O sistema apresenta lista vazia e orientação para ampliar ou limpar filtros. **Retorno:** passo 1, se houver nova busca; caso contrário, encerra normalmente.
- **A2 — Alteração dos filtros. Origem: passo 5.** O administrador ajusta a consulta. **Retorno:** passo 3.

### Exceções

- **E1 — Filtros inválidos. Origem: passo 3.** O sistema rejeita nota fora de 1 a 5 ou período final anterior ao inicial. **Fim:** não executa a consulta inválida; permite reiniciar no passo 1.
- **E2 — Acesso insuficiente. Origem: passo 2.** O sistema nega a consulta administrativa. **Fim:** encerra sem apresentar avaliações.
- **E3 — Consulta indisponível. Origem: passo 3.** O sistema informa erro de consulta. **Fim:** encerra sem alterar avaliações e sem apresentar falha como lista vazia.

**Pós-condição de sucesso:** Avaliações consultadas somente com os campos permitidos; dados dos pedidos não ficam disponíveis ao administrador.

**Pós-condição de falha:** Avaliações permanecem inalteradas e nenhum campo pessoal estruturado é exposto.

**Rastreabilidade:** RF-035; RN-002, RN-016, RN-020, RN-023; RNF-001, RNF-002, RNF-003, RNF-004.

**Observação de privacidade:** A consulta omite campos estruturados de identificação. Comentários são texto livre e podem conter informações escritas pelo próprio usuário; portanto, a projeção não é apresentada como anonimização integral. A interface orienta o usuário a não incluir dados pessoais no comentário.

### Critérios de aceite

- **CA-035.1:** Apenas Administrador consulta; resultados respeitam categoria, modelo, nota e período.
- **CA-035.2:** A apresentação e os dados retornados à interface contêm somente categoria, modelo, nota, comentário e data; não incluem nome, e-mail, destinatário, contexto nem links para pedidos.
- **CA-035.3:** Lista vazia e erro de consulta são distintos; a interface não afirma que texto livre garante anonimato integral.

### Casos de teste propostos

- **CT-035.1 → CA-035.1:** Ter avaliações de notas 2 e 5 em duas categorias; filtrar nota 5 de uma categoria no período escolhido. Esperado: somente correspondências; conta comum não consegue executar a mesma consulta.
- **CT-035.2 → CA-035.2:** Avaliação vinculada a pedido com nome, e-mail, destinatário e contexto preenchidos. Consultar como administrador e inspecionar resposta da interface. Esperado: somente os cinco campos permitidos, sem identificadores pessoais estruturados ou acesso ao pedido.
- **CT-035.3 → CA-035.3:** Consultar filtro sem correspondência, simular falha de armazenamento e consultar comentário que menciona voluntariamente um nome. Esperado: mensagens distintas nos dois primeiros cenários; ausência de promessa de anonimização integral no terceiro.
