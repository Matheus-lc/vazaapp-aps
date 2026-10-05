# Catálogo de casos de uso — VazaApp 2.0

**35 casos de uso**, organizados por objetivo do ator. Cada item aponta para sua especificação, com gatilho, pré/pós-condições, principal, alternativas, exceções, regras, requisitos, critérios de aceite e cenários de teste.

## Atores

| Ator | Papel e limite |
|---|---|
| Visitante | Pessoa sem sessão ativa, inclusive quem já possui conta e deseja entrar ou recuperar acesso. |
| Usuário | Pessoa autenticada que mantém seus dados e solicita sugestões. |
| Administrador | Papel autenticado autorizado a manter o catálogo e consultar avaliações minimizadas. Uma pessoa pode ter ambos os papéis, sem implicar acesso aos pedidos de terceiros. |
| Serviço de e-mail | Sistema externo secundário usado para entregar o token de recuperação de acesso. |

Conta, banco de dados, categorias, modelos e repositórios internos não são atores. Os repositórios de persistência aparecem apenas como participantes do diagrama de sequência.

## Visão geral

| ID | Objetivo | Ator principal | Requisitos | Regras |
|---|---|---|---|---|
| [UC-001](docs/01-contas-contexto-pedidos.md#uc-001--cadastrar-conta) | Cadastrar conta | Visitante | RF-001 | RN-001 |
| [UC-002](docs/01-contas-contexto-pedidos.md#uc-002--autenticar-usuário) | Autenticar usuário | Visitante | RF-002 | RN-002 |
| [UC-003](docs/01-contas-contexto-pedidos.md#uc-003--encerrar-sessão) | Encerrar sessão | Usuário / Administrador | RF-003 | RN-002 |
| [UC-004](docs/01-contas-contexto-pedidos.md#uc-004--recuperar-acesso) | Recuperar acesso | Visitante | RF-004 | RN-004 |
| [UC-005](docs/01-contas-contexto-pedidos.md#uc-005--alterar-dados-da-conta) | Alterar dados da conta | Usuário | RF-005 | RN-001, RN-003, RN-021 |
| [UC-006](docs/01-contas-contexto-pedidos.md#uc-006--excluir-conta) | Excluir conta | Usuário | RF-006 | RN-003, RN-005 |
| [UC-007](docs/01-contas-contexto-pedidos.md#uc-007--consultar-perfil-de-contexto) | Consultar perfil de contexto | Usuário | RF-007 | RN-003, RN-006, RN-023 |
| [UC-008](docs/01-contas-contexto-pedidos.md#uc-008--atualizar-perfil-de-contexto) | Atualizar perfil de contexto | Usuário | RF-008 | RN-003, RN-006 |
| [UC-009](docs/01-contas-contexto-pedidos.md#uc-009--cadastrar-destinatário) | Cadastrar destinatário | Usuário | RF-009 | RN-003, RN-007 |
| [UC-010](docs/01-contas-contexto-pedidos.md#uc-010--consultar-destinatários) | Consultar destinatários | Usuário | RF-010 | RN-003, RN-007, RN-023 |
| [UC-011](docs/01-contas-contexto-pedidos.md#uc-011--atualizar-destinatário) | Atualizar destinatário | Usuário | RF-011 | RN-003, RN-007 |
| [UC-012](docs/01-contas-contexto-pedidos.md#uc-012--excluir-destinatário) | Excluir destinatário | Usuário | RF-012 | RN-003, RN-022 |
| [UC-013](docs/01-contas-contexto-pedidos.md#uc-013--criar-pedido-de-desculpa) | Criar pedido de desculpa | Usuário | RF-013 | RN-003, RN-007, RN-008 |
| [UC-014](docs/01-contas-contexto-pedidos.md#uc-014--consultar-pedidos) | Consultar pedidos | Usuário | RF-014 | RN-003, RN-009, RN-023 |
| [UC-015](docs/01-contas-contexto-pedidos.md#uc-015--atualizar-pedido-em-rascunho) | Atualizar pedido em rascunho | Usuário | RF-015 | RN-003, RN-007, RN-008, RN-009 |
| [UC-016](docs/01-contas-contexto-pedidos.md#uc-016--excluir-pedido) | Excluir pedido | Usuário | RF-016 | RN-003, RN-009 |
| [UC-017](docs/02-sugestoes-avaliacoes-catalogo.md#uc-017--gerar-sugestão-personalizada) | Gerar sugestão personalizada | Usuário | RF-017 | RN-002, RN-003, RN-008, RN-009, RN-010, RN-011, RN-012, RN-024 |
| [UC-018](docs/02-sugestoes-avaliacoes-catalogo.md#uc-018--solicitar-nova-sugestão) | Solicitar nova sugestão | Usuário | RF-018, RF-017 | RN-003, RN-010, RN-012, RN-013, RN-024 |
| [UC-019](docs/02-sugestoes-avaliacoes-catalogo.md#uc-019--consultar-histórico-de-sugestões) | Consultar histórico de sugestões | Usuário | RF-019 | RN-003, RN-012, RN-023 |
| [UC-020](docs/02-sugestoes-avaliacoes-catalogo.md#uc-020--copiar-sugestão) | Copiar sugestão | Usuário | RF-020 | RN-003, RN-015 |
| [UC-021](docs/02-sugestoes-avaliacoes-catalogo.md#uc-021--favoritar-sugestão) | Favoritar sugestão | Usuário | RF-021 | RN-003, RN-014 |
| [UC-022](docs/02-sugestoes-avaliacoes-catalogo.md#uc-022--consultar-favoritos) | Consultar favoritos | Usuário | RF-022 | RN-003, RN-014, RN-023 |
| [UC-023](docs/02-sugestoes-avaliacoes-catalogo.md#uc-023--remover-sugestão-dos-favoritos) | Remover sugestão dos favoritos | Usuário | RF-023 | RN-003, RN-014 |
| [UC-024](docs/02-sugestoes-avaliacoes-catalogo.md#uc-024--avaliar-sugestão) | Avaliar sugestão | Usuário | RF-024 | RN-003, RN-016 |
| [UC-025](docs/02-sugestoes-avaliacoes-catalogo.md#uc-025--atualizar-avaliação) | Atualizar avaliação | Usuário | RF-025 | RN-003, RN-016 |
| [UC-026](docs/02-sugestoes-avaliacoes-catalogo.md#uc-026--excluir-avaliação) | Excluir avaliação | Usuário | RF-026 | RN-003, RN-016 |
| [UC-027](docs/02-sugestoes-avaliacoes-catalogo.md#uc-027--cadastrar-categoria) | Cadastrar categoria | Administrador | RF-027 | RN-002, RN-017 |
| [UC-028](docs/02-sugestoes-avaliacoes-catalogo.md#uc-028--consultar-categorias) | Consultar categorias | Administrador | RF-028 | RN-002, RN-017, RN-023 |
| [UC-029](docs/02-sugestoes-avaliacoes-catalogo.md#uc-029--atualizar-categoria) | Atualizar categoria | Administrador | RF-029 | RN-002, RN-017 |
| [UC-030](docs/02-sugestoes-avaliacoes-catalogo.md#uc-030--desativar-categoria) | Desativar categoria | Administrador | RF-030 | RN-002, RN-017 |
| [UC-031](docs/02-sugestoes-avaliacoes-catalogo.md#uc-031--cadastrar-modelo-de-desculpa) | Cadastrar modelo de desculpa | Administrador | RF-031 | RN-002, RN-011, RN-017, RN-018 |
| [UC-032](docs/02-sugestoes-avaliacoes-catalogo.md#uc-032--consultar-modelos-de-desculpa) | Consultar modelos de desculpa | Administrador | RF-032 | RN-002, RN-018, RN-023 |
| [UC-033](docs/02-sugestoes-avaliacoes-catalogo.md#uc-033--atualizar-modelo-de-desculpa) | Atualizar modelo de desculpa | Administrador | RF-033 | RN-002, RN-011, RN-017, RN-018, RN-019 |
| [UC-034](docs/02-sugestoes-avaliacoes-catalogo.md#uc-034--desativar-modelo-de-desculpa) | Desativar modelo de desculpa | Administrador | RF-034 | RN-002, RN-019 |
| [UC-035](docs/02-sugestoes-avaliacoes-catalogo.md#uc-035--consultar-avaliações-recebidas) | Consultar avaliações recebidas | Administrador | RF-035 | RN-002, RN-016, RN-020, RN-023 |

## Especificações e modelos

- [UC-001 a UC-016: conta, contexto, destinatários e pedidos](docs/01-contas-contexto-pedidos.md)
- [UC-017 a UC-035: sugestões, favoritos, avaliações e catálogo](docs/02-sugestoes-avaliacoes-catalogo.md)
- [Regras de negócio e requisitos](docs/requisitos.md)
- [Diagramas de casos de uso e sequência em Mermaid](docs/diagramas.md)
- [Matriz de rastreabilidade](docs/rastreabilidade.md)
- [Revisão e critérios de qualidade](docs/validacao.md)
