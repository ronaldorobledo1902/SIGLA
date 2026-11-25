# Container: backends, worker e bases

O navegador chega ao SME-SIGLA-BFF. A partir daí a chamada fica na rede interna: o SME-SIGLA-BFF consulta o SME-LOCUS-MICROSERVICO, o MS-AdminUsuarios e os microsserviços do ALVO. A visão de conjunto está em [Containers](container.md).

## 1. SME-SIGLA-BFF

Um SME-SIGLA-BFF atende o front do ALVO e o front do Locus ([ADR-015](../../documentos/adr/ADR-015-bff-unico.md)).

| Responsabilidade | Como |
| --- | --- |
| Entrada da tela | Recebe a requisição dos dois fronts |
| Token | Valida o JWT. No login, chama o MS-AdminUsuarios |
| Consulta | Chama o SME-LOCUS-MICROSERVICO e os microsserviços do ALVO que a tela precisa |
| Resposta | Agrega e devolve um DTO da tela |

O Airflow da importação periódica do EOL não passa pelo SME-SIGLA-BFF.

## 2. SME-LOCUS-MICROSERVICO

Guarda o quadro que nenhum microsserviço do ALVO tem como dono ([ADR-004](../../documentos/adr/ADR-004-ms-locus.md)). Lê cargo, pessoa, nomeação, processo, escola e vaga de processo por HTTP, com `X-API-Key`.

| Bloco | Onde lê ou grava |
| --- | --- |
| Lotação, exercício, afastamento, módulo, vaga calculada, vínculo, movimentação, Data base DOC, congelado | Base do Locus |
| Escola, DRE, escolha, vaga do processo | MS-Escolha |
| Pessoa, CPF, RF, classificação de concursado | MS-Candidatos |
| Código e descrição do cargo | MS-ProcessosConcursos |
| Data de nomeação e sessão | MS-Agenda |
| Processo, convocação e corte de vagas | MS-ProcessosConvocacao |

A carga inicial copia o Banco Access para a base do Locus, uma vez. A vaga do módulo é calculada e gravada nessa mesma base.

## 3. Relatorio e WORKER

| Peça | Papel |
| --- | --- |
| Relatorio | Responde na hora o relatório de menor processamento. No de grande processamento, dispara a DAG e envia o job ao WORKER ([ADR-011](../../documentos/adr/ADR-011-relatorios-assincronos.md)) |
| WORKER | Fila `locus`. Recebe o job, lê e grava na base do Locus |
| Airflow | DAG do relatório pesado. Executa e grava o relatório na mesma base |

## 4. Airflow e a busca do EOL

| Peça | Papel |
| --- | --- |
| Agenda | Dispara a busca periódica |
| Busca | Consulta só o EOL |
| Gravação | Salva na base do Locus um JSON, um PDF ou um hash ([ADR-005](../../documentos/adr/ADR-005-integracao-importa-arquivos.md)) |

Essa DAG não passa pelo SME-SIGLA-BFF nem pelo MS-ImportaArquivos. O confronto automático entre o EOL e o quadro reutiliza o caminho do relatório de grande processamento.

## 5. Microsserviços do ALVO

| MS | O que o Locus lê |
| --- | --- |
| MS-Escolha | Escola, DRE, escolha e vaga do processo |
| MS-Candidatos | Pessoa habilitada, quando o RF existe |
| MS-ProcessosConcursos | Catálogo de cargo |
| MS-Agenda | `escolha_em` e `nomeacao_em` |
| MS-ProcessosConvocacao | Processo, data de convocação e corte de vagas |
| MS-AdminUsuarios | Login e permissão. O SME-SIGLA-BFF é quem chama |
| MS-ImportaArquivos | Porta de entrada do ALVO. O Locus não recebe lote dele |
| MS-Relatorios | Relatório de escolha e convocação. O relatório do módulo nasce no SME-LOCUS-MICROSERVICO |
| MS-Convocacao | Fora. O fluxo real de convocação é o MS-ProcessosConvocacao |

Cada um lê e grava a base do ALVO. A chamada entre microsserviços usa `X-API-Key`.

## 6. Bases

As bases são separadas ([ADR-006](../../documentos/adr/ADR-006-base-unica.md)).

| Base | Quem usa | O que guarda |
| --- | --- | --- |
| Base do ALVO | Microsserviços do ALVO | Concurso, convocação, escolha, importação do ALVO |
| Base do Locus | SME-LOCUS-MICROSERVICO, WORKER, DAG do Airflow, carga do Access | Quadro do módulo, relatório leve, relatório pesado, snapshot do EOL |

O SME-LOCUS-MICROSERVICO não abre conexão na base do ALVO. A referência ao ALVO é por chave natural.

## 7. Comunicação

| De | Para | Protocolo |
| --- | --- | --- |
| Front do ALVO e Front do Locus | SME-SIGLA-BFF | HTTPS |
| SME-SIGLA-BFF | MS-AdminUsuarios, SME-LOCUS-MICROSERVICO e microsserviços do ALVO | HTTPS na rede interna |
| SME-LOCUS-MICROSERVICO | Microsserviços do ALVO | REST + `X-API-Key` |
| SME-LOCUS-MICROSERVICO | WORKER | Job na fila `locus` (Celery + Redis) |
| SME-LOCUS-MICROSERVICO | Airflow | Disparo da DAG do relatório pesado |
| Airflow | EOL | Busca periódica |
| Airflow | Base do Locus | Grava JSON, PDF ou hash, e o relatório pesado |
| MS-ImportaArquivos | Microsserviços do ALVO | REST + `X-API-Key`. O lote permanece no ALVO |
| MS-AdminUsuarios | API SME Integração | HTTPS + `x-api-eol-key` |
| Microsserviços do ALVO | Base do ALVO | ORM, credencial do ALVO |
| SME-LOCUS-MICROSERVICO e WORKER | Base do Locus | ORM, credencial do Locus |

## 8. Configuração

| Peça | Variáveis |
| --- | --- |
| Microsserviços Django | `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS`, `DB_*` da base daquele subsistema |
| MS-AdminUsuarios | `SME_INTEGRACAO_URL`, `SME_INTEGRACAO_TOKEN`, `EMAIL_*`, `REDIS_URL` |
| MS-ImportaArquivos | `JWT_SIGNING_KEY`, `PRODAM_*`, `CELERY_REDIS_URL`, `API_KEY`, URLs e chaves dos MS do ALVO |
| SME-LOCUS-MICROSERVICO | `JWT_SIGNING_KEY` igual à do emissor, `DB_*` da base do Locus, `CELERY_REDIS_URL`, `API_KEY`, URLs e chaves dos MS do ALVO que ele lê |
| SME-SIGLA-BFF | Endereço publicado aos fronts, chave de validação do JWT, URLs internas dos microsserviços |

Segredos vão em Secret do Kubernetes. `env.example` fica com placeholder.

## 9. Diretrizes

1. A tela publica o contrato no SME-SIGLA-BFF. A regra do quadro fica no SME-LOCUS-MICROSERVICO.
2. Leitura do ALVO é HTTP com `X-API-Key`. Sem conexão cruzada de banco.
3. O MS-ImportaArquivos não ganha cliente do Locus.
4. A DAG do EOL e a DAG do relatório pesado gravam na base do Locus.
5. Tarefa Celery idempotente, com timeout.
6. `CORS_ALLOWED_ORIGINS` explícito por ambiente.
7. Health checks `/health/live` e `/health/ready`.
