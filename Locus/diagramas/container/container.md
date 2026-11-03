# Container da Solução — LOCUS (Lotação e Controle Unificado de Servidores)

Diagrama de **contêineres (C2)** do LOCUS: frontends, microsserviços e workers em Kubernetes (Rancher), persistência e sistemas externos. Elementos com **borda tracejada** são TO-BE.

```{image} ../assets/03-locus-containers.svg
:width: 100%
:alt: Diagrama de Containers (C2) — LOCUS
```

---

## Descrição das fronteiras e fluxos

### 1. Camada de apresentação (LOCUS-Frontend)

* **Portal SIGLA** — React; login, recuperação de senha e seleção ALVO / LOCUS em `/sigla/` (hoje espelhado em `modules/sigla` do LOCUS).
* **LOCUS SPA** — React 19 / Vite / Ant Design; servido por Nginx em `/locus/`. Telas de UE e servidor prontas com dados estáticos.
* **Telas de Importação e Permissões (TO-BE)** — previstas na wiki; consomem MS-ImportaArquivos e MS-AdminUsuarios.

### 2. Camada de ingresso

* **Ingress Nginx (API Gateway)** — `sigla.sme.prefeitura.sp.gov.br/locus` e `api-sigla.sme.prefeitura.sp.gov.br/ms-admin-usuarios`, `/ms-importa-arquivos`, `/ms-locus` (TO-BE).

### 3. Camada de API e workers (LOCUS-Backend)

| Container | Tecnologia | Papel |
|-----------|------------|-------|
| MS-AdminUsuarios | Django 5.2 / DRF | Login CoreSSO, JWT, usuários, grupos, permissões |
| MS-ImportaArquivos | Django 5.2 / DRF | Layouts, importações, PRODAM, exportações SIGPEC |
| Worker ImportaArquivos | Celery worker + beat | Cargas agendadas (fila `importa_arquivos`) |
| MS-Locus (TO-BE) | Django 5.2 / DRF + `sigla-sdk` | Domínio de servidores, UE, lotação, movimentações, histórico |
| Worker MS-Locus (TO-BE) | Celery worker + beat | Conciliação, read model, relatórios (fila `locus`) |
| Redis | Redis 7 | Broker Celery compartilhado |

### 4. Camada de dados (Persistência)

* **PostgreSQL SIGLA — escrita** — write model por domínio (usuários, importações, LOCUS).
* **PostgreSQL SIGLA — leitura (TO-BE)** — read model CQRS (Command Query Responsibility Segregation) para telas, relatórios e exportações.
* **Arquivos importados** — hoje `media/` local; alvo: volume persistente / object storage.

### 5. Operação

* **Jenkins + Registry SME** — build e deploy (Infra Ágil).
* **Elastic APM + Grafana** — APM (Application Performance Monitoring), logs JSON e telemetria.

### 6. Serviços externos

* CoreSSO / API SME Integração, Keycloak (em estudo), EOL / PRODAM, SIGPEC (SEGES), E-mail SME (SMTP); ALVO como consumidor da base SIGLA.

---

## O que existe neste C2

| Padrão | LOCUS |
|--------|-------|
| BFF (Backend for Frontend) único | **Ausente** — o SPA fala com cada MS via Ingress |
| Mensageria | **Celery + Redis** (sem RabbitMQ) |
| CQRS | Escrita existente; leitura **TO-BE** |
| Login | CoreSSO via MS-AdminUsuarios → JWT |
| Origem oficial dos dados funcionais | **EOL / SIGPEC** (LOCUS concilia) |
