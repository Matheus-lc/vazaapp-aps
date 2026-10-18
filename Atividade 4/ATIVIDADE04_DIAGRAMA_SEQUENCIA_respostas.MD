# Atividade 4: Diagramas de sequência

**Projeto da equipe:** VazaApp  
**Disciplina:** Análise e Projeto de Software  
**Professor:** Prof. Dr. João Choma

**Integrantes:** Isabella Barela Venancio (RA 144023), Carolina Yumi Toyota (RA 143210), Matheus Lara Campos (RA 126608) e Gabriela Borborema Paes Barreto (RA 144438).

[Enunciado do professor](https://github.com/JoaoChoma/analise_projeto_esoft_2026/blob/main/atividades/1_bim/ATIVIDADE04_DIAGRAMA_SEQUENCIA.MD) | [Exercício 24](exercicio24.md)

## Exercício 16: Caso de uso e diagrama de sequência

O caso de uso descreve como o ator alcança um objetivo: o que informa, o que o sistema faz, quais desvios podem ocorrer e qual resultado deve ser obtido.

O diagrama de sequência detalha um cenário desse caso de uso e mostra quem troca mensagens, em qual ordem e com quais condições. Ele não substitui a especificação, porque uma sequência normalmente apresenta apenas parte dos caminhos e não reúne todas as pré-condições, regras, alternativas e exceções. Os dois artefatos se complementam.

## Exercício 17: Participantes da transferência bancária

Uma distribuição possível para o cenário é:

| Papel e participante | Responsabilidade e justificativa |
|---|---|
| Ator: Cliente | Solicita a transferência, informa os dados e recebe o resultado. É quem busca o objetivo. |
| Interface: TelaTransferencia | Coleta os dados, verifica o formato e apresenta mensagens. Cuida da interação. |
| Aplicação: TransferenciaService | Coordena consultas, validação e registro da operação. Organiza o fluxo sem concentrar as regras de negócio. |
| Domínio: Conta e Transferencia | Representa saldo e movimentação e verifica as condições para transferir. Mantém as regras próximas dos dados do negócio. |
| Persistência: ContaRepository | Consulta contas e saldo e registra a movimentação. A gravação de débito e crédito deve ser atômica, para evitar uma transferência pela metade. |

## Exercício 18: Reservar sala

O diagrama acompanha o fluxo principal do UC03, desde a solicitação até a confirmação da gravação. A interface conversa com o professor, o controller recebe as ações, o service coordena a reserva e o repository consulta e grava os dados.

![Diagrama do exercício 18](Diagramas%20Sequencias/exercicio18.svg)

[Ampliar imagem](Diagramas%20Sequencias/exercicio18.svg) | [Editar no Mermaid](https://mermaid.live/edit#pako:eNp1k01ugzAQha9ieR0uwCJSRbuo1AUClt1MYJJYMh46NlHSKIepuugqp-BiNcFufssCyX6fZ9488F7W1KBMpcWPHk2NzwpWDO27Ef6B3pHp2wVyWNeOWOQCrMiZlmgtBaUDdqpWHRgnqlGvUEOBFnkD90Q2EkHNyDgmrfFBpfKCK_1L1XgPFRdQgR1Z5U3uJi5P5vMqFSVpTztgwZeWqsTLeSpybBSLBhwIFGvi4YsVXRV4NUviFv6BKg9lqfCT2F6PXRplOzJqobRqoAmeM0-VqXijGrT69JQFDQEdjhvUE1Z6rLgsFixbYUh0yMORmtC3SKaSRSRwq6xD49DGWpOz8mGnLAnpPFRjOE_daMC4YPgqFm9yqU653OQa8wjqaVK8cZ9F8ytvmm9rhBi8t82tdB77dJJEHRrFynHskMtZh-vBo8HhZ_im66lftv7r8d_JQMiZbNEvVSPTvXRrbMerswCL8jCT420pd6aWqeMeZ7Lv_N8SL9S0efgFHPsq6Q)

## Exercício 19: Decisão com a RN05

**RN05:** uma sala não pode ter duas reservas em períodos conflitantes. Consideramos um período com início e fim; períodos consecutivos não entram em conflito.

O bloco `alt` separa sala disponível de sala indisponível. No primeiro caminho, a reserva depende da confirmação do professor. O service verifica a regra, e a gravação também impede sobreposição de forma atômica, pois outra pessoa pode ocupar a sala depois da consulta. Se houver conflito, nenhuma reserva é confirmada.

![Diagrama do exercício 19](Diagramas%20Sequencias/exercicio19.svg)

[Ampliar imagem](Diagramas%20Sequencias/exercicio19.svg) | [Editar no Mermaid](https://mermaid.live/edit#pako:eNqNlL-O2zAMxl9F0JwAXbp4OKBwC7RLEdhGpy6KzVwJKKJLycEdDvcwRYdO9xR5sVKSHTv_0GYI4vAT-fFHWi-6pQ50oT38HMC18BHNI5v9d6fkY4ZAbthvgcfnNhCrjTJebZh24D2Nkd5wwBZ744JqYrwBayrwwAdzrSijYoyW5AKTtXAjU73Q1fKFLVyLqoWogp48isnnrNusHx6aQn1xO-K9YdWZYBSoHvj4Rh1lUSOislBixA82RBX6nhxu0WJnurFkKaq6UN-AcYftPVUtqmqZi7Mxrzq6KFutc8ZqUsAT-gAugJ9zSfxDb1O96uu79-L9cDLgacup3-Of4-8xqbFCzVgzuju-HcDmSM6YW00KiweGOVauM6tbsSbGNtFKbMfFvvzdIiNzQbDDBJ2Xa3AOfFSkZJdjWUCvSRBgxvkokJiWPSXiVQ7EZKAMus4oJ1QyVGTVSjErmzEfnPhPNhJFRcNJGn-0xBxHMh-LhMdiSZHOLj0vQY-znXXmXDdBP_NwLpnYf3qSVeNTpgslWGm7nIw7c093y1zi1DLedfeZ-PiLkZRgvTXxpc15VDREQtczBdflh-Q5rdudvJPRU1_YySzS9qe3SU7Gl-J6h_9l-P_MJqN6pfcgHLHTxYsOP2Afb8ut8aBfVzpekPWza3UReICVHnq5YaY7NP_5-hdersWT)

## Exercício 20: Repetição para os produtos do pedido

O `loop` executa a mesma sequência para cada produto: verificar a disponibilidade, adicionar o item aceito e calcular seu subtotal. Um item sem estoque vira uma pendência, sem ser adicionado como disponível.

![Diagrama do exercício 20](Diagramas%20Sequencias/exercicio20.svg)

[Ampliar imagem](Diagramas%20Sequencias/exercicio20.svg) | [Editar no Mermaid](https://mermaid.live/edit#pako:eNptUztuwzAMvQqhObmAhwCF2yGbawedujASmxKQJUWfAkHQCxXoKXqx0lGdpEg0WXzvkU8kfVTaG1KNSrQv5DQ9Mu4ijq8O5GDJ3pVxS_HvrrOP0AImaC2Ty1TjAWNmzQFdhs2EbshiR4aNvyUME6GCA8UP1neSdBfOLdhP4FPKXhz3FHxicXWovHa5Wm0aWLs3H0eMYDllBEMQojfymgQE-yJZ2KChVEUbEQ0NdJGkjIjCVWHrfYBOwqDR4JymYtMZRNs38EKR31iL2HAK3vGW7akEGH8r6pe14vPZyZ_s5_uD7IWHVlpRxfcJs4OugQcj_fFOHHCm8T-lq5QWrS5WGKlss89oJ3P32NXdWhBp10y-kMgmmgcA7FKRl19tw7UxSdPTToZQ--rMz5fTjLeFBaqX88ewrLPsKZVR3i-zE4mbJnhJdJ6gcFtpQoiUxIkUiyfZq1MLNZLsAhvVHFV-p3Fa9y0mUp8LNW34cHBaNTkWWqgSDOb5J6jBz1-kJgX4)

Usar `loop` permite representar qualquer quantidade de produtos sem copiar as mesmas mensagens várias vezes. Isso deixa o diagrama mais curto e facilita perceber o que se repete.

## Exercício 21: Distribuição das responsabilidades do pedido

A TelaPedido concentra interação, regras de negócio, gravação e comunicação externa. Isso aumenta o acoplamento: uma mudança no desconto pode exigir alterar a tela, e fica mais difícil testar as regras ou reutilizá-las em outra interface.

A solução proposta distribui essas tarefas conforme o papel de cada participante:

| Participante | Responsabilidade |
|---|---|
| TelaPedido | Receber dados, validar formato e apresentar o resultado. |
| PedidoService | Coordenar o caso de uso e solicitar a gravação e a notificação. |
| DominioPedido | Validar cliente e estoque, aplicar o desconto e calcular o total. |
| PedidoRepository | Consultar os dados e gravar o pedido com a atualização do estoque de forma atômica. |
| ServicoEmail | Enviar a confirmação após a gravação confirmada. |

- O diagrama representa esse recorte de responsabilidades. Uma falha na gravação impede a confirmação e o envio do email.
- A conferência do estoque também precisa permanecer válida na gravação, pois outra compra pode ocorrer depois da consulta.

![Diagrama do exercício 21](Diagramas%20Sequencias/exercicio21.svg)

[Ampliar imagem](Diagramas%20Sequencias/exercicio21.svg) | [Editar no Mermaid](https://mermaid.live/edit#pako:eNqdlM9O5DAMxl8lygmkYR-gB6QVM9rTIgQc92ISD0RK45KklRDi3XH-tJ3OdNBoe2na_vz5s-P0UyrSKBsZ8L1Hp3Br4NVD-88JvkBF8uLOGnQRy6sOfDTKdOCieEYLAkK-P6A2mk6Z3501ChRQAgv0hH4wakVvS61xJpN1eU71EbsDwfQUDFv9OCV3LRib0JKV8nPBal03t7epgEYocnvjW_BF9UqDpnBd2EQwOFXDtDer5IQwnoxl3dDbCL4m3D140n2ksIhL7M0yRf48hfP6NEPtUyMGsEaD392BVb0Fv9Cu1ApfLV39AKI3e87ndyESz8gPqKq5txjYdKQL0GeKYI-5ZRc81vK5H9V2GicU3cFwgI0M8uQGAREdf4DarfU9CWAHrp7asaoidj3H3FNEQVz9HL0psX88DGWiIxtWwDvUCkcD5AlCz8fIgGC_WLR_zaIrmzxOXRKkXuzBvsEckOp6HfONqD4AjsvL890IdINJ9U3apwWmK9Pn-01Zh85lm05O0Z79HUXkw8NwHbZGQMdJeLl0eOANbcAsZ_lYC41jK6cOCQdTXy615-C_LP6laIalO6fLQ7aZxy6Lj5O3NnjHZjyqPqQmo2hzgjnoYi_Zh9zIFrkio2XzKeMbtul3_gIB5ddGAv9nnj6ckk30PW5k32l2WX_y5eXXN_CEAeI)

## Exercício 22: Mensagem sem origem documentada

A mensagem bloquearCliente() precisa ser investigada antes de ser aceita. Há quatro possibilidades:

- Diagrama errado: o bloqueio foi acrescentado sem necessidade e deve ser removido.
- Requisito incompleto: o comportamento foi validado com os interessados, mas faltou registrá-lo no RF e no caso de uso.
- Regra de negócio ausente: existe uma política de bloqueio, mas suas condições e consequências precisam ser documentadas e relacionadas ao requisito.
- Decisão interna legítima: pode ser uma trava técnica temporária para proteger uma operação concorrente, sem bloquear comercialmente o cliente. Nesse caso, convém renomear a mensagem e registrar a justificativa de projeto.

Para decidir, verificamos a finalidade, o efeito sobre o cliente e a origem da decisão com os responsáveis. Se houver mudança no comportamento do negócio, atualizamos RN, RF e UC antes de manter a mensagem. Uma escolha técnica não justifica inventar um bloqueio de negócio.

## Exercício 23: Agendar consulta

### 1. Regras de negócio

- **RN01:** A consulta só pode ocupar um horário disponível para o profissional escolhido.
- **RN02:** Um profissional não pode ter duas consultas ativas no mesmo horário.
- **RN03:** O cancelamento pelo paciente é permitido quando faltam pelo menos 24 horas para a consulta, incluindo exatamente 24 horas.
- **RN04:** O histórico deve preservar as consultas e seus cancelamentos. Cancelar altera a situação da consulta, sem apagar seu registro.

### 2. Requisitos funcionais

- **RF01:** O sistema deve apresentar os horários disponíveis conforme especialidade, profissional e data.
- **RF02:** O sistema deve permitir a confirmação de uma consulta e informar o resultado do agendamento.
- **RF03:** O sistema deve permitir o cancelamento da consulta identificada, verificando o prazo e informando o resultado.
- **RF04:** O sistema deve registrar cada consulta e cada cancelamento com identificação e data/hora da operação.

### 3. Requisitos não funcionais propostos

- **RNF01:** Integridade e concorrência. Em duas confirmações simultâneas para a mesma vaga, no máximo uma deve ser aceita. Uma falha não pode deixar consulta sem registro ou registro de sucesso sem consulta.
- **RNF02:** Controle de acesso proposto. Um paciente deve conseguir cancelar apenas suas próprias consultas. A tentativa de cancelar uma consulta de outro paciente deve ser recusada, sem alteração dos dados.
- **RNF03:** Desempenho proposto. Na consulta de disponibilidade, 95% das respostas devem chegar em até 2 segundos, com 20 usuários simultâneos e uma base de 1.000 consultas. Ambiente e resultados devem ser documentados quando o teste for realizado.

### 4. Atores e casos de uso

Paciente: consulta a disponibilidade, agenda e cancela suas consultas. O profissional é uma entidade do domínio. Só será ator se forem definidos serviços que ele utilize diretamente.

- **UC01:** Consultar disponibilidade.
- **UC02:** Agendar consulta.
- **UC03:** Cancelar consulta.

### 5. Especificação de UC02: Agendar consulta

| Campo | Descrição |
|---|---|
| Objetivo | Confirmar uma consulta no horário escolhido, sem conflito na agenda do profissional. |
| Ator | Paciente |
| Pré-condições | Paciente identificado. Especialidade, profissional e agenda cadastrados e compatíveis. |
| Pós-condição de sucesso | Consulta confirmada e registrada; a vaga fica indisponível para outro agendamento. |
| Pós-condição de falha | Nenhuma consulta confirmada por esta tentativa; nenhum registro parcial de sucesso. |

**Fluxo principal**

1. O paciente solicita o agendamento.
2. O sistema solicita especialidade, profissional e data.
3. O paciente informa os filtros.
4. O sistema consulta e apresenta os horários disponíveis.
5. O paciente escolhe um horário e confirma.
6. O sistema revalida a disponibilidade e registra a consulta e a operação de forma atômica, impedindo dupla ocupação.
7. O sistema apresenta a confirmação com os dados da consulta.

**Fluxos alternativos**

- **A1, passo 4:** não há horários disponíveis. O paciente pode mudar os filtros e retornar ao passo 3, ou encerrar.
- **A2, passo 6:** o horário foi ocupado antes da confirmação. Nenhuma consulta é criada; o sistema atualiza as opções e retorna ao passo 4.
- **A3, passos 1 a 5, antes de confirmar:** o paciente desiste. O caso termina sem criar consulta.

**Exceções**

- **E1, passos 3 ou 5:** dados incompletos ou incompatíveis. O sistema informa o problema e retorna ao passo correspondente para correção.
- **E2, passo 6:** falha na persistência. A operação inteira é desfeita e o caso termina sem consulta confirmada.

**Premissas:** o paciente já está identificado. Cada horário representa uma vaga; duração das consultas e funcionamento da clínica ainda precisam ser definidos. Os RNFs são propostas, sem testes de implementação executados.

### 6. Rastreabilidade

| RN ou decisão | RF / RNF | UC | Onde aparece |
|---|---|---|---|
| RN01 | RF01, RF02, RNF03 | UC01, UC02 | Consulta de disponibilidade e validação na confirmação. |
| RN02 | RF02, RNF01 | UC02 | Reserva exclusiva e tratamento do conflito concorrente. |
| RN03 | RF03 | UC03 | Comparar o instante da solicitação com o início da consulta; aceitar diferença maior ou igual a 24 horas. |
| RN04 | RF04, RNF01 | UC02, UC03 | Registro consistente do agendamento e preservação do histórico no cancelamento. |
| Decisão proposta de segurança | RNF02 | UC03 | Verificar se a consulta pertence ao paciente antes de cancelar. |

### 7. Checklist e revisão

| Verificação | Resultado / ajuste |
|---|---|
| Objetivo, ator e condições | Definidos na especificação de UC02. |
| Alternativas e exceções | Cada desvio identifica a origem e informa retorno ou encerramento. |
| Concorrência | A confirmação revalida a vaga e usa gravação exclusiva e atômica. Consultar antes não basta. |
| Cancelamento | Exatamente 24 horas é permitido. O registro é preservado, em vez de ser apagado. |
| Limite do diagrama | O agendamento representa RN01/RN02/RN04. A regra de cancelamento pertence ao UC03. |

### 8. Participantes e responsabilidades

| Participante | Responsabilidade |
|---|---|
| Paciente | Escolher os dados da consulta e confirmar a solicitação. |
| TelaAgendamento | Coletar dados e apresentar opções, confirmações e erros. |
| AgendamentoController | Receber a solicitação e encaminhá-la ao serviço de aplicação. |
| AgendamentoService | Coordenar o caso de uso e aplicar as políticas de disponibilidade e conflito do domínio. |
| ConsultaRepository | Consultar a agenda e persistir a reserva com exclusividade e transação. A decisão de agendar pertence ao domínio; o repositório garante a restrição também diante de concorrência. |

### 9. Diagrama de sequência

![Diagrama do exercício 23](Diagramas%20Sequencias/exercicio23.svg)

[Ampliar imagem](Diagramas%20Sequencias/exercicio23.svg) | [Editar no Mermaid](https://mermaid.live/edit#pako:eNqdVktu2zAQvcpAKwewewAvDBh2imZTGLKX3UyocUyA4qgkZTQNcpiiiwIFsuwJfLEO9fcnblKtBOrNcOa9x6GeEsUZJdPE09eSrKKlxgeH-RcL8qAK7GAF6GGFSpMNVH8o0AWtdIE2wCZ-3pDB-QPZDHNB8TlqEVEDxIJtcGwMuXPs-gS7JrfX6sLWaQRKJl-agCkV7LUU_FgDV5PZbDOFNRuBB3SAp_VtJgJZTWFFmXZAviCl0egMMxpD4Xirvdds0QBBhgGP8t7ZLbtc0m61kU58m3I2W0xBNTW5pfYFW32v67SjBnxToxeCXr8ZvRZ0OoX70it0n9ih0-ybkD1pfwJPJ3V2QR5-RChkFfbwEsFtyrrea5jFpG74GqZlcl448kKw0MLF4dfhD3ngEtjpetHyXuKPGOMidOYSDRSbHUGZw67ZTrgXfrZauK4DBhosmg9ugK4y6Ix7cKNJLb8bFc1mxxKPY4rI6E0f2Mjz_sBGqdd0nYcSzehfSVr9ToIBY3QPQxM6aQbKDBDx-czCLu_JwXoslaW0r4wurgdHD9oH2b2rVxa5IIci4E8Gi5CTzxEEY329-OE4e9Nv1N7t0d2mbcp54Fyr6szRSKrvzHnaZKtkhuNKbiMHOTpni2aHxyGx4bTeqXNGdoIZerudD1fBrcmXsUY57B0X59DW67ffRBPXZa14OUaT8VR1VnUjQMXO9UP0UrGdkqzKQmqBQgYwl8Il-GaQXdho2EE_7-SwYXMerrTRzTFts2OnXWjmY5QjOkLs4UXiw28rI_M8eW83scWS_Ba_y3tgIfaaj4ZMDOY_2OjDyJ8pDy_Zlebr-mSX3r9v6LwyGXjKOzHdK_pLSf1CxUgnWEvfhcPXtpSSKv2li-h_NHyPfl3Z8pKMk5wkQGfJ9CkJO8rj9X-PnpLncYKlXLePViXT4EoaJ2UhF1_7U1AvPv8F5SurBQ)

## Exercício 24: Projeto da equipe

Escolhemos o **UC-017: Gerar sugestão personalizada**, uma das operações centrais do VazaApp. A análise, o checklist, o diagrama e a justificativa dos participantes estão no [arquivo do exercício 24](exercicio24.md).

## Questão final

O diagrama também ajuda a descobrir problemas e decisões que o texto ainda não deixou claros. Ao distribuir as mensagens, precisamos definir quem decide, quando decide e o que acontece se algo mudar.

Nos exercícios 18 e 19, duas pessoas podem consultar a mesma sala disponível e confirmar quase ao mesmo tempo. Isso mostra a necessidade de conferir e impedir o conflito na gravação, e não apenas na consulta inicial.

No exercício 21, colocar validação, desconto, gravação e email na TelaPedido revela concentração de responsabilidades. A sequência ajuda a separar esses papéis e a perceber que a confirmação do pedido deve depender da gravação bem-sucedida.
