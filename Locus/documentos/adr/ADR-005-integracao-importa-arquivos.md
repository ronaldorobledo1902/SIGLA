# ADR-005 — Integrações EOL / SIGPEC: reaproveitar o MS-ImportaArquivos


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Proposed                                |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Time SIGLA (decisão no levantamento LOCUS) |
| **Sistema**      | LOCUS + MS-ImportaArquivos              |
| **Relacionadas** | ADR-004, ADR-007                        |


---

## Contexto

O **MS-ImportaArquivos** não é um serviço do LOCUS. Hoje ele atende o ALVO: layout de arquivo (tipo padrão `HABILITADOS`), importação de escolhas da PRODAM para o MS-Escolha, e exportação de vagas e lotes no formato SIGPEC. Tem validação, registro de erros, histórico, Celery beat (`enfileirar_importacoes_escolhas`) e clientes HTTP para os MS do ALVO. Não importa SIGPEC nem grava quadro funcional do LOCUS.

## Decisão

1. Toda **ingestão** de EOL e SIGPEC passa pelo **MS-ImportaArquivos**: novos tipos de importação (servidores, vínculos, unidades, dados funcionais) e novo cliente `api_locus`.
2. O MS-ImportaArquivos **não aplica regra de negócio**: valida formato e entrega lotes ao **MS-Locus** (`/api/v1/integracao/`, `X-API-Key`), que faz a conciliação.
3. A carga que existe hoje é a de escolhas PRODAM, via **Celery beat**. Ingestão de arquivo SIGPEC para o LOCUS não existe no serviço.
4. Arquivos passam a ficar em **armazenamento persistente** (PVC ou object storage), não em `media/` do pod.

## Consequências

### Positivas

- Reuso de código e operação já existentes; um único ponto de ingestão no SIGLA
- Separação clara: ingestão × domínio

### Negativas

- MS-ImportaArquivos atende dois subsistemas — mudanças precisam de regressão do ALVO
- Uma chamada a mais (ImportaArquivos → MS-Locus) por lote

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Integração dentro do MS-Locus | Duplicaria layout/validação/erros já existentes |
| Novo MS de integração dedicado | Mais um serviço sem ganho claro |
| ETL (Extract, Transform, Load) externo (SSIS ou similar) | Fora do padrão SIGLA; operação em outro time |

## Referências

- `SME-SIGLA-MS-ImportaArquivos/importa_arquivos/*`
