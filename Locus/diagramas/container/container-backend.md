# Container: Backend LOCUS

O backend do LOCUS é formado por três microsserviços Django/DRF (Django REST Framework) — dois existentes e um TO-BE — e seus workers Celery, publicados atrás do Ingress em `api-sigla.sme.prefeitura.sp.gov.br`.

```{image} ../assets/03-locus-containers.svg
:width: 100%
:alt: Containers do backend LOCUS
```

---

## 1. Microsserviços

| MS | Rota | Responsabilidade | Situação |
|----|------|------------------|----------|
| MS-AdminUsuarios | `/ms-admin-usuarios` | Identidade, JWT, usuários, grupos/permissões de ALVO e LOCUS | Existente |
| MS-ImportaArquivos | `/ms-importa-arquivos` | Ingestão: layouts, importações, PRODAM, SIGPEC; exportações do ALVO | Existente |
| MS-Locus | `/ms-locus` | Domínio: servidores, UE, módulos, lotação/exercício, movimentações, vacância, histórico, conciliação, relatórios | **TO-BE** |

## 2. Workers

| Worker | Fila | Tarefas |
|--------|------|---------|
| Worker ImportaArquivos | `importa_arquivos` | `enfileirar_importacoes_escolhas` (1 min), cargas EOL/SIGPEC (proposta) |
| Worker MS-Locus | `locus` | Conciliação diária, projeção do read model, relatórios assíncronos |

## 3. Comunicação

| De | Para | Protocolo |
|----|------|-----------|
| Frontends | MS | HTTPS + JWT via Ingress |
| MS-ImportaArquivos | MS-Locus | REST + `X-API-Key` (`sigla-sdk.http_client`, timeout 30 s) |
| MS-ImportaArquivos | MS do ALVO | REST + `X-API-Key` |
| MS-AdminUsuarios | API SME Integração | HTTPS + `x-api-eol-key` |
| MS | Redis | Protocolo Redis (Celery) |
| MS | PostgreSQL | Django ORM |

## 4. Configuração (variáveis principais)

| MS | Variáveis |
|----|-----------|
| Todos | `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS`, `DB_*` |
| MS-AdminUsuarios | `SME_INTEGRACAO_URL`, `SME_INTEGRACAO_TOKEN`, `EMAIL_*`, `REDIS_URL` |
| MS-ImportaArquivos | `JWT_SIGNING_KEY`, `PRODAM_ESCOLHAS_API_URL`, `PRODAM_API_*`, `CELERY_REDIS_URL`, `API_KEY` / `API_KEY_HEADER`, `<MS>_API_URL` e `<MS>_API_KEY` dos MS de destino, `ELASTIC_APM_*` |
| MS-Locus (TO-BE) | `JWT_SIGNING_KEY` (igual ao emissor), `DB_*` e `DB_READ_*`, `CELERY_REDIS_URL`, `API_KEY`, `ELASTIC_APM_*`; no MS-ImportaArquivos: `LOCUS_API_URL` / `LOCUS_API_KEY` |

Segredos vão em **Secret** do Kubernetes; `env.example` não deve conter valores reais.

## 5. Diretrizes

1. Não acessar tabelas de outro MS diretamente — usar API.
2. Toda tarefa Celery deve ser idempotente e ter timeout.
3. `CORS_ALLOWED_ORIGINS` explícito por ambiente (sem `CORS_ALLOW_ALL_ORIGINS`).
4. Health checks `/health/live` e `/health/ready` com dependências (banco, Redis).
