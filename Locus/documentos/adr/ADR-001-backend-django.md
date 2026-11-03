# ADR-001 — Backend: Python / Django 5.2 + DRF (Django REST Framework) + `sigla-sdk`


| Campo            | Valor                                             |
| ---------------- | ------------------------------------------------- |
| **Status**       | Accepted (as-is)                                  |
| **Data**         | 2026-10-27                                        |
| **Decisores**    | Arquitetura SIGLA (wiki ADR-01) / Levantamento LOCUS |
| **Sistema**      | LOCUS (todos os MS)                               |
| **Relacionadas** | ADR-004, ADR-007                                  |


---

## Contexto

A wiki SIGLA registrou no ADR-01 a escolha de Django em vez de FastAPI para os microsserviços. Todos os MS existentes (AdminUsuarios, ImportaArquivos e os do ALVO) seguem Django 5.2.5, DRF 3.15, SimpleJWT, drf-spectacular, django-auditlog, elastic-apm, python-json-logger e a biblioteca comum `sigla-sdk` (tag `1.1.1`), em Python 3.12 / Docker.

## Decisão

1. Todo backend do LOCUS (inclusive o novo MS-Locus) usa **Django 5.2 + DRF** com a mesma estrutura de camadas: `api` → `serializers` → `services` → `repository` → `models`.
2. Uso obrigatório do `sigla-sdk` para `http_client`, `CorrelationIdMiddleware`, `AuditlogJWTMiddleware`, `ApiKeyAuthentication` e `CustomJsonFormatter`.
3. OpenAPI via **drf-spectacular**; cobertura de testes ≥ **80%**.

## Consequências

### Positivas

- Mesmo padrão para o time inteiro (ALVO e LOCUS); onboarding e revisão mais simples
- Admin, ORM, migrations e auditlog prontos para um domínio cadastral com histórico

### Negativas

- Throughput menor que stacks assíncronas — irrelevante para o volume da DICAR
- Dependência de uma versão do `sigla-sdk` por git tag: atualizações exigem coordenação

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| FastAPI | Rejeitado no ADR-01 da wiki (menos baterias incluídas: admin, ORM, auth) |
| Node/NestJS | Fora do padrão do SIGLA |

## Referências

- Wiki SIGLA — *Visão Arquitetural* (ADR-01)
- `SME-SIGLA-MS-AdminUsuarios/requirements.txt`, `SME-SIGLA-MS-ImportaArquivos/requirements.txt`
- `SME-SIGLA-SDK-Microsservico`
