# ADR-006 — Dados: Base de Dados SIGLA (BASE SIGLA) com CQRS (Command Query Responsibility Segregation)


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Accepted (as-is) — read model do LOCUS Proposed |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Arquitetura SIGLA                       |
| **Sistema**      | SIGLA                                   |
| **Relacionadas** | ADR-004, ADR-011                        |


---

## Contexto

A **Base de Dados SIGLA (BASE SIGLA)** está em PostgreSQL, separada em **Base Sigla Escrita** e **Base Sigla Leitura**, ligadas por CQRS. A Escrita guarda os write models por domínio; a Leitura é atualizada de forma assíncrona para telas, exportações e dashboards. O LOCUS terá 100 mil+ servidores, histórico crescente e relatórios pesados; o PO pede consultas em até 3 s.

## Decisão

1. MS-Locus grava o write model do LOCUS (schema do domínio) na **Base Sigla Escrita**.
2. O read model é projetado na **Base Sigla Leitura** pelo Worker MS-Locus após cada alteração (evento Celery) e por reconciliação periódica.
3. Telas de listagem e relatórios leem da **Base Sigla Leitura**; detalhes e edição leem da **Base Sigla Escrita** (consistência imediata para quem altera).
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
| Base única sem separação | Contraria a separação entre Base Sigla Escrita e Base Sigla Leitura; relatórios pesados degradam a operação |
| Réplica física de leitura (streaming) | Possível evolução; não substitui modelos de leitura otimizados |

## Referências

- Wiki SIGLA — Diagramas › Visão BACKEND (`documentos/anexos/wiki/Visao_BACKEND_1.png`): Base de Dados SIGLA (BASE SIGLA), Base Sigla Escrita e Base Sigla Leitura
