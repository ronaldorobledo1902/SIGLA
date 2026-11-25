# Componentes dos backends

Os backends deste cenário são o SME-SIGLA-BFF, o SME-LOCUS-MICROSERVICO e o WORKER do Locus, mais os microsserviços do ALVO que a tela e o quadro consultam. O Airflow executa a busca do EOL e o relatório de grande processamento. Os containers estão em [Backend](../container/container-backend.md).

## 1. SME-SIGLA-BFF

Um componente só para o front do ALVO e o front do Locus ([ADR-015](../../documentos/adr/ADR-015-bff-unico.md)). O código do ALVO ainda não tem este componente.

| Componente | Responsabilidade |
| --- | --- |
| Entrada da tela | Recebe a requisição e devolve um DTO |
| Validação do token | Confere o JWT antes da consulta. No login, chama o MS-AdminUsuarios |
| Agregador do Cadastro | Junta unidade, cargo, pessoa e quadro |
| Agregador de Relatórios | Devolve o relatório leve na hora e o pesado quando a gravação termina |
| Agregador da Data base | Junta agenda, processo, habilitado e escolha. Confirma o vínculo no Locus |
| Agregador de Vagas | Pede o cálculo e o ajuste de EXC ao SME-LOCUS-MICROSERVICO |
| Agregador da Remoção | Junta quadro do Locus e escola do MS-Escolha |
| Clientes HTTP | Chamam os microsserviços na rede interna. A chave `X-API-Key` não sai para o navegador |

## 2. SME-LOCUS-MICROSERVICO

Dono do quadro do módulo ([ADR-004](../../documentos/adr/ADR-004-ms-locus.md)). Padrão dos MS SIGLA: `api` → `serializers` → `services` → `repository` → `models`, com `sigla-sdk`.

| Componente | Responsabilidade |
| --- | --- |
| Middlewares `sigla-sdk` | `CorrelationIdMiddleware`, `AuditlogJWTMiddleware` |
| Autenticação entre MS | `ApiKeyAuthentication`. O token da tela é validado no SME-SIGLA-BFF |
| OpenAPI | drf-spectacular (`/api/schema`, Swagger) do contrato que o SME-SIGLA-BFF consome |
| API do quadro | Vínculo, lotação, exercício, afastamento, módulo, movimentação, Data base DOC, congelado |
| API de vagas | Cálculo com módulo, lotados e afastados da base do Locus. Ajuste de EXC e recálculo |
| API de relatórios | Resposta síncrona do relatório leve. Disparo da DAG e envio do job no relatório pesado |
| Carga inicial | Copia o Banco Access para a base do Locus, uma vez |
| Clientes do ALVO | Leitura HTTP em MS-Escolha, MS-Candidatos, MS-ProcessosConcursos, MS-Agenda e MS-ProcessosConvocacao |
| Services de domínio | `UnidadeService`, `ServidorService`, `LotacaoService`, `MovimentacaoService`, cálculo de vaga |
| Histórico e versões | `django-auditlog` e snapshots ([ADR-008](../../documentos/adr/ADR-008-historico-versoes.md)) |
| Repositórios e models | Consulta e gravação na base do Locus. Chave natural para o ALVO, sem chave estrangeira entre bases |
| Observabilidade | elastic-apm. Log JSON sem dado sensível |

Models do quadro: vínculo, cargo em exercício, lotação, exercício, afastamento, movimentação, módulo da unidade, vaga calculada, data base DOC, congelado.

## 3. WORKER e Airflow

| Componente | Responsabilidade |
| --- | --- |
| WORKER (fila `locus`) | Recebe o job do relatório pesado. Lê e grava na base do Locus |
| DAG de relatório | Executa o relatório de grande processamento e grava o resultado na base do Locus |
| DAG do EOL | Consulta só o EOL e grava na base do Locus um JSON, um PDF ou um hash |
| Confronto EOL × quadro | Reutiliza a DAG de relatório. O resultado fica na base do Locus |

A DAG do EOL não passa pelo SME-SIGLA-BFF nem pelo MS-ImportaArquivos ([ADR-005](../../documentos/adr/ADR-005-integracao-importa-arquivos.md), [ADR-011](../../documentos/adr/ADR-011-relatorios-assincronos.md)).

## 4. MS-ImportaArquivos

Porta de entrada do ALVO. Habilitados, escolhas PRODAM e exportação SIGPEC gravam na base do ALVO. Este serviço não entrega lote ao SME-LOCUS-MICROSERVICO.

| Componente | Responsabilidade |
| --- | --- |
| API Layouts | Tipo e estrutura do arquivo do ALVO |
| API Importações | Habilitados e escolhas PRODAM. Histórico e erros |
| API Exportações | Vagas e lotes SIGPEC a partir do MS-Escolha |
| Validação de layout | `validacao_*`, schema e `ImportacaoErro` |
| Services de importação | `importacao_lotes`, `importacao_escolhas_service` |
| Tasks Celery | `enfileirar_importacoes_escolhas` na fila `importa_arquivos` |
| Clientes HTTP | `api_candidatos`, `api_escolhas`, `api_concursos`, `api_agendas`, `api_processos_convocacao` |
| Cliente PRODAM | `api_prodam` |

## 5. MS-AdminUsuarios

Identidade e permissão. O SME-SIGLA-BFF chama este serviço em nome da tela.

| Componente | Responsabilidade |
| --- | --- |
| LoginView | `/api/v1/login` |
| AutenticacaoService | API SME Integração `/api/v1/autenticacao/externa` |
| SimpleJWT | Access e refresh |
| Views de senha e dados | Esqueci e nova senha, alterar senha e e-mail, meus dados |
| SmeIntegracaoService | Dados, senha e e-mail no CoreSSO |
| Views de usuários e permissões | Usuário, grupo e permissão. Inclui os perfis do Locus |
| UserRepository | Usuários e grupos locais |

## 6. Diretrizes

1. Regra de negócio do quadro só no SME-LOCUS-MICROSERVICO. O SME-SIGLA-BFF agrega e devolve o DTO.
2. Toda alteração funcional do quadro gera versão: motivo, portaria, data e autor.
3. Leitura do ALVO por HTTP com `X-API-Key`. A credencial da base do Locus não alcança a base do ALVO ([ADR-006](../../documentos/adr/ADR-006-base-unica.md)).
4. Tarefa da fila `locus` idempotente e com timeout.
5. `CORS_ALLOWED_ORIGINS` explícito. `JWT_SIGNING_KEY` dedicada.
6. Segredo fora do Git. `env.example` só com placeholder.
7. Health checks `/health/live` e `/health/ready`.

## 7. Relação com ADRs

| ADR | Tema |
| --- | --- |
| ADR-001 | Django / DRF + `sigla-sdk` |
| ADR-002 | CoreSSO + JWT, chamado pelo SME-SIGLA-BFF |
| ADR-003 | Perfis do Locus |
| ADR-004 | SME-LOCUS-MICROSERVICO |
| ADR-005 | Importação do ALVO e EOL no Airflow |
| ADR-006 | Bases separadas |
| ADR-007 | Celery + Redis na fila `locus` e na fila do ALVO |
| ADR-008 | Histórico e versões |
| ADR-011 | Relatório leve na hora e relatório pesado na DAG |
| ADR-015 | SME-SIGLA-BFF único |
