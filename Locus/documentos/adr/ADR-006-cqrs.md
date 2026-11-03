# ADR-006 — Dados: PostgreSQL `DB_SIGLA` com CQRS (Command Query Responsibility Segregation)


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Accepted (as-is) — read model do LOCUS Proposed |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Arquitetura SIGLA (wiki)                |
| **Sistema**      | SIGLA                                   |
| **Relacionadas** | ADR-004, ADR-011                        |


---

## Contexto

A wiki define `DB_SIGLA` em PostgreSQL com CQRS: write models por domínio e read models atualizados de forma assíncrona para telas, exportações e dashboards. O LOCUS terá 100 mil+ servidores, histórico crescente e relatórios pesados; o PO pede consultas em até 3 s.

## Decisão

1. MS-Locus grava em **schema próprio** (write model LOCUS) no `DB_SIGLA`.
2. Um **read model** (schema/base de leitura) é projetado pelo Worker MS-Locus após cada alteração (evento Celery) e por reconciliação periódica.
3. Telas de listagem e relatórios leem do read model; detalhes e edição leem do write model (consistência imediata para quem altera).
4. Credenciais de banco **separadas por MS** (sem acesso cruzado a schemas de outros domínios).

## Consequências

### Positivas

- Relatórios não disputam recursos com a operação
- Alinhado ao padrão SIGLA

### Negativas

- Consistência eventual em listagens (segundos)
- Exige monitorar o atraso da projeção e ter reprocessamento

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Base única sem separação | Contraria a wiki; relatórios pesados degradam a operação |
| Réplica física de leitura (streaming) | Possível evolução; não substitui modelos de leitura otimizados |

## Referências

- Wiki SIGLA — Banco de Dados `DB_SIGLA` e uso de CQRS
