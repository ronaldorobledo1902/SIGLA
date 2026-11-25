# Container da Solução — LOCUS (Lotação e Controle Unificado de Servidores)

Diagrama de **contêineres (C2)** do LOCUS: frontends, microsserviços e workers. Elementos com **borda tracejada** são proposta ou não estão nos repositórios.

```{image} ../assets/03-locus-containers.svg
:width: 100%
:alt: Diagrama de Containers (C2) — LOCUS
```

---

## Descrição das fronteiras e fluxos

### 1. Camada de apresentação (LOCUS-Frontend)

* **Portal SIGLA** — React; login, recuperação de senha e seleção ALVO / LOCUS em `/sigla/` (hoje espelhado em `modules/sigla` do LOCUS).
* **LOCUS SPA** — React 19 / Vite / Ant Design; servido por Nginx em `/locus/`. Telas de UE e servidor prontas com dados estáticos.
* **Telas de Importação e Permissões (TO-BE)** — previstas; consomem MS-ImportaArquivos e MS-AdminUsuarios.

### 2. Camada de ingresso

* **Ingress** — não há manifesto nos repositórios. Rate limit e gateway não estão no código.

### 3. Camada de API e workers (LOCUS-Backend)

| Container | Tecnologia | Papel |
|-----------|------------|-------|
| MS-AdminUsuarios | Django 5.2 / DRF | Login CoreSSO, JWT, usuários, grupos, permissões |
| MS-ImportaArquivos | Django 5.2 / DRF | Layouts, importações, PRODAM, exportações SIGPEC |
| Worker ImportaArquivos | Celery worker + beat | Cargas agendadas (fila `importa_arquivos`) |
| MS-Locus (TO-BE) | Django 5.2 / DRF + `sigla-sdk` | Domínio de servidores, UE, lotação, movimentações, histórico |
| Worker MS-Locus (TO-BE) | Celery worker + beat | Conciliação e relatórios (fila `locus`) |
| Redis | Redis 7 | Broker Celery compartilhado |

### 4. Camada de dados (Persistência)

* **Base de Dados SIGLA** (PostgreSQL, `db_sigla`) — única base: usuários, importações, domínio LOCUS, consultas e relatórios.
* **Arquivos importados** — hoje `media/` local; alvo: volume persistente / object storage.

### 5. Operação

* **Elastic APM** — cliente nas settings, desligado por padrão. Grafana não está nos repositórios.
* **Jenkins** — não há Jenkinsfile nos repositórios do LOCUS.

### 6. Serviços externos

* CoreSSO / API SME Integração, EOL / PRODAM, SIGPEC (SEGES), E-mail SME (SMTP).

---

## O que existe neste C2

| Padrão | LOCUS |
|--------|-------|
| BFF (Backend for Frontend) único | O front do ALVO e o do Locus falam só com ele. Os microsserviços ficam internos ([ADR-015](../../documentos/adr/ADR-015-bff-unico.md)) |
| Mensageria | **Celery + Redis** (sem RabbitMQ) |
| Base única | PostgreSQL `db_sigla`; consultas e gravações na mesma base |
| Login | CoreSSO via MS-AdminUsuarios → JWT |
| Importação em execução | Escolhas PRODAM e exportação SIGPEC, no MS-ImportaArquivos (uso ALVO) |
