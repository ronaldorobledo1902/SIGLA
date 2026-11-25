# ADR-005 — Importações: ALVO no MS-ImportaArquivos e EOL do Locus no Airflow


| Campo            | Valor                                      |
| ---------------- | ------------------------------------------ |
| **Status**       | Proposed                                   |
| **Data**         | 2026-10-27                                 |
| **Revisão**      | 2026-11-25                                 |
| **Decisores**    | Arquitetura SIGLA                          |
| **Sistema**      | SIGLA (ALVO e LOCUS)                       |
| **Relacionadas** | ADR-004, ADR-007, ADR-012                  |


---

## Contexto

O **MS-ImportaArquivos** atende o ALVO: layout de arquivo (tipo padrão `HABILITADOS`), importação de escolhas da PRODAM para o MS-Escolha e exportação de vagas e lotes no formato SIGPEC. Valida o arquivo, registra o erro, guarda histórico e usa Celery beat (`enfileirar_importacoes_escolhas`). O resultado fica na base do ALVO.

O Locus precisa do quadro que esse serviço não guarda e de uma busca periódica do EOL. A decisão anterior, de criar tipos de importação de servidores e um cliente `api_locus` para entregar lotes ao SME-LOCUS-MICROSERVICO, não se aplica.

## Decisão

1. O MS-ImportaArquivos permanece a porta de entrada das importações do ALVO. Habilitados, escolhas PRODAM e exportação SIGPEC gravam na base do ALVO.
2. O Locus lê o resultado já gravado nos microsserviços do ALVO (MS-ProcessosConcursos, MS-Candidatos, MS-Agenda, MS-ProcessosConvocacao e MS-Escolha), por HTTP com `X-API-Key`. Não recebe lote do MS-ImportaArquivos.
3. A carga inicial do quadro copia o Banco Access para a base do Locus, uma vez.
4. A busca periódica do EOL é uma DAG do Airflow. O Airflow consulta só o EOL e grava na base do Locus um JSON, um PDF ou um hash. Essa DAG não passa pelo SME-SIGLA-BFF nem pelo MS-ImportaArquivos.
5. O confronto automático entre o EOL e o quadro reutiliza o caminho do relatório de grande processamento: a DAG grava o resultado na base do Locus.

## Consequências

### Positivas

- O ALVO conserva o fluxo de importação que já opera
- A busca do EOL fica no Airflow, no mesmo arranjo do relatório pesado

### Negativas

- O SME-LOCUS-MICROSERVICO depende dos microsserviços do ALVO para escola, cargo e pessoa
- A DAG do Airflow é mais um agendamento para operar

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Toda ingestão de EOL e SIGPEC pelo MS-ImportaArquivos, com cliente `api_locus` | O Locus não recebe lote desse serviço. O EOL entra pela DAG do Airflow |
| Integração do EOL dentro do SME-LOCUS-MICROSERVICO | A busca periódica fica no Airflow |
| Novo MS de integração | A importação do ALVO já tem dono; a do EOL já tem a DAG |

## Referências

- [Containers do reaproveitamento](../../cenarios/reaproveitamento-alvo/containers.md)
- [Documento arquitetural](../../visao-arquitetural/documento_arquitetural_sistema.md)
- `SME-SIGLA-MS-ImportaArquivos/importa_arquivos/*`
