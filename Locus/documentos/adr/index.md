# ADRs (Architecture Decision Records) — LOCUS (Lotação e Controle Unificado de Servidores)

**Pacote inicial** de Architecture Decision Records do LOCUS (outubro/2026), derivado dos repositórios existentes e das decisões tomadas no levantamento.

Formato: decisões com Contexto → Decisão → Consequências → Alternativas → Referências.

| Status | Significado |
|--------|-------------|
| **Accepted (as-is)** | Decisão já vigente no SIGLA; documentada retrospectivamente |
| **Proposed** | Decisão recomendada; aguarda aceite e implementação |

```{toctree}
:maxdepth: 1
:hidden:

ADR-001-backend-django
ADR-002-autenticacao
ADR-003-permissoes
ADR-004-ms-locus
ADR-005-integracao-importa-arquivos
ADR-006-base-unica
ADR-007-assincrono-celery
ADR-008-historico-versoes
ADR-009-infra-gateway
ADR-010-branches
ADR-011-relatorios-assincronos
ADR-012-modulo-fundamental-ii
ADR-013-calculo-vagas-confronto
ADR-014-escolha-precarios
ADR-015-bff-unico
```

---

## Índice

| ADR | Título | Status |
|-----|--------|--------|
| [ADR-001](./ADR-001-backend-django.md) | Backend — Python / Django 5.2 + DRF (Django REST Framework) + `sigla-sdk` | Accepted (as-is) |
| [ADR-002](./ADR-002-autenticacao.md) | Autenticação — CoreSSO via MS-AdminUsuarios + JWT (JSON Web Token) | Accepted (as-is) |
| [ADR-003](./ADR-003-permissoes.md) | Permissões de ALVO e LOCUS geridas no LOCUS, com perfis LOCUS | Proposed |
| [ADR-004](./ADR-004-ms-locus.md) | MS-Locus — quadro em exercício, sem recriar cadastros do ALVO | Proposed |
| [ADR-005](./ADR-005-integracao-importa-arquivos.md) | Integrações EOL / SIGPEC — reaproveitar o MS-ImportaArquivos | Proposed |
| [ADR-006](./ADR-006-base-unica.md) | Dados — Base de Dados SIGLA única (PostgreSQL `db_sigla`) | Accepted (as-is) |
| [ADR-007](./ADR-007-assincrono-celery.md) | Processamento assíncrono — Celery + Redis com filas por MS | Accepted (as-is) |
| [ADR-008](./ADR-008-historico-versoes.md) | Histórico — `django-auditlog` + versões de registro | Proposed |
| [ADR-009](./ADR-009-infra-gateway.md) | Infra — o que os repositórios mostram | Accepted (as-is) |
| [ADR-010](./ADR-010-branches.md) | Branches — padrão `test` / `homolog` / `master` | Accepted (as-is) |
| [ADR-011](./ADR-011-relatorios-assincronos.md) | Relatórios — síncronos até 3 s, assíncronos acima | Proposed |
| [ADR-012](./ADR-012-modulo-fundamental-ii.md) | Fundamental II e Médio no MS-Locus | Proposed |
| [ADR-013](./ADR-013-calculo-vagas-confronto.md) | Vagas e confronto com o EOL | Proposed |
| [ADR-014](./ADR-014-escolha-precarios.md) | Acesso aos sistemas externos do Fundamental II | Proposed |
| [ADR-015](./ADR-015-bff-unico.md) | BFF único para ALVO e LOCUS | Proposed |

---

## Mapa de dependências

```text
ADR-001 (Django/DRF)
   ├── ADR-004 (MS-Locus) [Proposed]
   │      ├── ADR-008 (histórico) [Proposed]
   │      ├── ADR-011 (relatórios) [Proposed]
   │      ├── ADR-012 (módulo Fund. II) [Proposed]
   │      │      ├── ADR-013 (vagas e confronto) [Proposed]
   │      │      └── ADR-014 (acesso aos sistemas externos) [Proposed]
   └── ADR-007 (Celery/Redis)

ADR-002 (CoreSSO + JWT)
   ├── ADR-003 (permissões/perfis LOCUS) [Proposed]
   └── ADR-015 (BFF único) [Proposed]

ADR-005 (integração via MS-ImportaArquivos) [Proposed]
   ├── ADR-007 (Celery/Redis)
   └── ADR-004 (entrega ao MS-Locus)

ADR-006 (base única)
   └── ADR-011 (relatório síncrono ou DAG do Airflow na base do Locus)

ADR-009 (infra observada nos repositórios)
   └── ADR-010 (branches)
```

---

## Documentos de origem

- `visao-arquitetural/documento_arquitetural_sistema.md`
- `seguranca/index.md`
- `diagramas/assets/ARQUITETURA - LOCUS - C4.drawio`
- Questionário do PO LOCUS

---

*Pacote inicial em 27/10/2026 (ADR-001 a ADR-011). ADR-012 a ADR-014, de 17/11/2026, registram o módulo Fundamental II e Médio. Em 24/11/2026 a entrevista tirou o Sistema Escolha da integração. EOL e CI seguem sem contrato. No mesmo dia, o ADR-015 coloca um BFF único na frente do ALVO e do Locus.*
