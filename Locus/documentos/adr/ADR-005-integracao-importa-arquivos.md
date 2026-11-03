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

O LOCUS precisa consumir EOL (PRODAM) e SIGPEC (SEGES) ao menos 1x/dia, com o mecanismo ainda indefinido. O **MS-ImportaArquivos** — atribuído ao LOCUS na wiki — já possui layouts posicionais, validação, registro de erros, histórico, cliente PRODAM, Celery beat e clientes HTTP para outros MS.

## Decisão

1. Toda **ingestão** de EOL e SIGPEC passa pelo **MS-ImportaArquivos**: novos tipos de importação (servidores, vínculos, unidades, dados funcionais) e novo cliente `api_locus`.
2. O MS-ImportaArquivos **não aplica regra de negócio**: valida formato e entrega lotes ao **MS-Locus** (`/api/v1/integracao/`, `X-API-Key`), que faz a conciliação.
3. Cargas automáticas via **Celery beat**; SIGPEC por **upload** enquanto não houver automação.
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
- `visao-arquitetural/integracoes_eol_sigpec.md`
- Wiki SIGLA — Jornadas › Importação de dados LOCUS
