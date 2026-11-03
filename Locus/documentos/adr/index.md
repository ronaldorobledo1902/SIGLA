# ADRs (Architecture Decision Records) — LOCUS (Lotação e Controle Unificado de Servidores)

**Pacote inicial** de Architecture Decision Records do LOCUS (outubro/2026), derivado da wiki SIGLA, dos repositórios existentes e das decisões tomadas no levantamento.

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
ADR-006-cqrs
ADR-007-assincrono-celery
ADR-008-historico-versoes
ADR-009-infra-gateway
ADR-010-branches
ADR-011-relatorios-assincronos
```

---

## Índice

| ADR | Título | Status |
|-----|--------|--------|
| [ADR-001](./ADR-001-backend-django.md) | Backend — Python / Django 5.2 + DRF (Django REST Framework) + `sigla-sdk` | Accepted (as-is) |
| [ADR-002](./ADR-002-autenticacao.md) | Autenticação — CoreSSO via MS-AdminUsuarios + JWT (JSON Web Token) | Accepted (as-is) |
| [ADR-003](./ADR-003-permissoes.md) | Permissões de ALVO e LOCUS geridas no LOCUS, com perfis LOCUS | Proposed |
| [ADR-004](./ADR-004-ms-locus.md) | MS-Locus — microsserviço de domínio de servidores e lotação | Proposed |
| [ADR-005](./ADR-005-integracao-importa-arquivos.md) | Integrações EOL / SIGPEC — reaproveitar o MS-ImportaArquivos | Proposed |
| [ADR-006](./ADR-006-cqrs.md) | Dados — PostgreSQL `DB_SIGLA` com CQRS (Command Query Responsibility Segregation) | Accepted (as-is) / leitura Proposed |
| [ADR-007](./ADR-007-assincrono-celery.md) | Processamento assíncrono — Celery + Redis com filas por MS | Accepted (as-is) |
| [ADR-008](./ADR-008-historico-versoes.md) | Histórico — `django-auditlog` + versões de registro | Proposed |
| [ADR-009](./ADR-009-infra-gateway.md) | Infra — Ingress como API Gateway, Rancher, Jenkins | Accepted (as-is) — a confirmar |
| [ADR-010](./ADR-010-branches.md) | Branches — padrão `test` / `homolog` / `master` | Accepted (as-is) |
| [ADR-011](./ADR-011-relatorios-assincronos.md) | Relatórios — síncronos até 3 s, assíncronos acima | Proposed |

---

## Mapa de dependências

```text
ADR-001 (Django/DRF)
   ├── ADR-004 (MS-Locus) [Proposed]
   │      ├── ADR-008 (histórico) [Proposed]
   │      └── ADR-011 (relatórios) [Proposed]
   └── ADR-007 (Celery/Redis)

ADR-002 (CoreSSO + JWT)
   └── ADR-003 (permissões/perfis LOCUS) [Proposed]

ADR-005 (integração via MS-ImportaArquivos) [Proposed]
   ├── ADR-007 (Celery/Redis)
   └── ADR-004 (entrega ao MS-Locus)

ADR-006 (CQRS)
   └── ADR-011 (relatórios no read model)

ADR-009 (Ingress/Rancher/Jenkins)
   └── ADR-010 (branches)
```

---

## Documentos de origem

- `visao-arquitetural/documento_arquitetural_sistema.md`
- `visao-arquitetural/integracoes_eol_sigpec.md`
- `seguranca/index.md`
- `diagramas/assets/ARQUITETURA - LOCUS - C4.drawio`
- Wiki SIGLA *Visão Arquitetural* (ADR-01 Django × FastAPI), questionário do PO LOCUS

---

*Gerado em 27/10/2026 — pacote inicial (11 ADRs).*
