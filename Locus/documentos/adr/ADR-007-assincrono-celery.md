# ADR-007 — Processamento assíncrono: Celery + Redis com filas por MS


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Accepted (as-is)                        |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Time SIGLA                              |
| **Sistema**      | SIGLA                                   |
| **Relacionadas** | ADR-005, ADR-006, ADR-011               |


---

## Contexto

O MS-ImportaArquivos usa Celery 5.4 com Redis como broker (fila `importa_arquivos`, beat a cada 1 min, timeouts 300/360 s). O LOCUS precisa de conciliação diária, projeção de read model e relatórios assíncronos.

## Decisão

1. Manter **Celery + Redis** como mecanismo assíncrono do SIGLA.
2. **Uma fila por MS** (`importa_arquivos`, `locus`) e workers dedicados; Redis compartilhado.
3. Tarefas **idempotentes**, com `acks_late`, retry com backoff e limites de tempo explícitos.
4. Agendamentos em **Celery beat** (um beat por MS, réplica única).

## Consequências

### Positivas

- Stack já conhecida e operada
- Isolamento de carga por fila

### Negativas

- Redis compartilhado é ponto único de falha para todos os workers
- Sem DLQ (Dead Letter Queue) nativa: falhas precisam de registro próprio (ex.: tabela de execuções)

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| RabbitMQ | Não usado no SIGLA; operação adicional |
| Kafka | Volume não justifica |
| Cron no Kubernetes | Perde retry/monitoramento do Celery |

## Referências

- `SME-SIGLA-MS-ImportaArquivos` — `celery.py`, `tasks.py`, `settings` (`CELERY_*`)
