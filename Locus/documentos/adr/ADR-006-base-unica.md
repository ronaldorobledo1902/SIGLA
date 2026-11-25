# ADR-006 — Dados: bases separadas de ALVO e LOCUS


| Campo            | Valor                         |
| ---------------- | ----------------------------- |
| **Status**       | Proposed                      |
| **Data**         | 2026-10-27                    |
| **Revisão**      | 2026-11-25                    |
| **Decisores**    | Arquitetura SIGLA             |
| **Sistema**      | SIGLA                         |
| **Relacionadas** | ADR-004, ADR-011              |


---

## Contexto

A leitura anterior tratava o SIGLA como um único PostgreSQL (`db_sigla`), compartilhado por todos os microsserviços. O desenho vigente separa a persistência dos dois subsistemas.

Os microsserviços do ALVO leem e gravam a base do ALVO. O SME-LOCUS-MICROSERVICO lê e grava a base do Locus. Relatório leve e relatório pesado usam a base do Locus. Não há uma terceira base só de relatório.

## Decisão

1. A base do ALVO e a base do Locus são bancos separados.
2. Os microsserviços do ALVO leem e gravam a base do ALVO, com a credencial desse subsistema.
3. O SME-LOCUS-MICROSERVICO lê e grava somente a base do Locus. A carga inicial do Banco Access, o arquivo do EOL gravado pelo Airflow e os relatórios ficam nessa base.
4. O SME-LOCUS-MICROSERVICO não abre conexão na base do ALVO. Cargo, pessoa, nomeação, processo, escola e vaga de processo são lidos por HTTP nos microsserviços do ALVO, com `X-API-Key`. A referência é por chave natural, sem chave estrangeira entre as bases.
5. Cada base tem credencial própria.
6. Os relatórios do módulo usam a base do Locus ([ADR-011](ADR-011-relatorios-assincronos.md)). Não se cria base só de relatório.

## Consequências

### Positivas

- O quadro do módulo e o processo de concurso não compartilham tabelas
- A credencial do Locus não alcança a base do ALVO

### Negativas

- Consulta que junta quadro e concurso passa pelos microsserviços, por HTTP
- Relatório pesado disputa recurso com a operação do quadro quando roda na base do Locus

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Um único PostgreSQL (`db_sigla`) para ALVO e Locus | As bases dos dois subsistemas são separadas |
| Base só de relatório | O relatório leve e o pesado gravam na base do Locus |
| O SME-LOCUS-MICROSERVICO consultar o banco do ALVO | A leitura do ALVO é HTTP, com `X-API-Key` |

## Referências

- [Documento arquitetural](../../visao-arquitetural/documento_arquitetural_sistema.md)
- [Containers do reaproveitamento](../../cenarios/reaproveitamento-alvo/containers.md)
