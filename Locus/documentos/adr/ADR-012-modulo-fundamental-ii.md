# ADR-012 — Fundamental II e Médio no SME-LOCUS-MICROSERVICO

| Campo | Valor |
| --- | --- |
| **Status** | Proposed |
| **Data** | 2026-11-17 |
| **Revisão** | 2026-11-25 |
| **Decisores** | Arquitetura SIGLA |
| **Sistema** | LOCUS |
| **Relacionadas** | ADR-004, ADR-014 |

## Contexto

O levantamento do Fundamental II e Médio (05/11/2026) descreve cinco fluxos e não divide microsserviços. O desenho de containers mostra o backend como o único que acessa a base e os sistemas externos.

O EOL é sistema externo. A busca periódica é uma DAG do Airflow, que grava JSON, PDF ou hash na base do Locus. O SME-LOCUS-MICROSERVICO não chama o EOL. O Sistema Escolha ficou fora na entrevista de `transcricaoreuniao.vtt`: é o legado do ALVO, sem carga e sem retorno.

## Decisão

1. O backend do módulo é o **SME-LOCUS-MICROSERVICO**.
2. O SME-LOCUS-MICROSERVICO lê e grava a base do Locus. Cargo, pessoa, nomeação, processo, escola e vaga são lidos por HTTP nos microsserviços do ALVO.
3. Não há integração com o Sistema Escolha. A carga inicial do quadro é o Banco Access, uma vez, na base do Locus.
4. A busca periódica do EOL é a DAG do Airflow ([ADR-005](ADR-005-integracao-importa-arquivos.md)). O SME-LOCUS-MICROSERVICO não chama o EOL e não recebe lote do MS-ImportaArquivos.

## Consequências

### Positivas

- O módulo continua em um backend, e a origem do nomeado deixa de ser o legado

### Negativas

- O SME-LOCUS-MICROSERVICO não tem chamada direta ao EOL. O arquivo chega pela DAG do Airflow

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
| --- | --- |
| O SME-LOCUS-MICROSERVICO chamar o EOL | A busca periódica fica na DAG do Airflow |
| Manter o Sistema Escolha como sistema externo do módulo | A entrevista removeu essa ligação |

## Referências

- [Containers do reaproveitamento](../../cenarios/reaproveitamento-alvo/containers.md)
- Levantamento *Locus – Fundamental II e Médio: fluxo, containers e componentes*
- `Levantamento/transcricaoreuniao.vtt`
