# ADR-006 — Dados: Base de Dados SIGLA única (PostgreSQL)


| Campo            | Valor                         |
| ---------------- | ----------------------------- |
| **Status**       | Accepted (as-is)              |
| **Data**         | 2026-11-17                    |
| **Decisores**    | Arquitetura SIGLA             |
| **Sistema**      | SIGLA                         |
| **Relacionadas** | ADR-004, ADR-011              |


---

## Contexto

Cada microsserviço configura um único `DATABASES["default"]` em PostgreSQL (`db_sigla`). Não há segundo banco, router de leitura/escrita nem projeção de read model.

## Decisão

1. A **Base de Dados SIGLA (BASE SIGLA)** é um único PostgreSQL (`db_sigla`).
2. Consultas, gravações, importações e relatórios do LOCUS usam essa mesma base.
3. O MS-Locus segue o mesmo padrão dos MS existentes: Django ORM sobre `default`.
4. Credenciais de banco continuam **separadas por MS** (sem acesso cruzado a schemas de outros domínios).
5. Relatórios pesados continuam assíncronos (ADR-011), lendo a mesma base. O tempo de resposta (≤ 3 s) se resolve com paginação, índices e job, sem uma segunda base.

## Consequências

### Positivas

- A documentação fica alinhada ao que o time implementa
- Consulta e gravação enxergam o mesmo dado, sem atraso de projeção

### Negativas

- Relatório pesado disputa recurso com a operação se rodar de forma síncrona na mesma base

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Duas bases, uma só de leitura | Não existe no código |
| Réplica física de leitura | Não é o desenho atual; só faria sentido depois de medição |

## Referências

- `config/settings.py` dos MS SIGLA — `DATABASES["default"]`, nome `db_sigla`
