# Documentação Arquitetural — LOCUS (Lotação e Controle Unificado de Servidores)

**Sistema:** LOCUS (subsistema do SIGLA)  
**Data da análise:** Outubro/2026  
**Versão do documento:** 1.0  
**Fontes:** Repositórios `SME-SIGLA-LOCUS-Frontend`, `SME-SIGLA-Frontend`, `SME-SIGLA-MS-AdminUsuarios`, `SME-SIGLA-MS-ImportaArquivos`, `SME-SIGLA-SDK-Microsservico`; wiki Azure DevOps *COTIC - DevOps › SIGLA › Visão Arquitetural*; questionário do PO (Product Owner) LOCUS  

## 1. Visão Geral do Sistema

O **SIGLA** é composto por dois subsistemas: **ALVO** (Alocação de Vagas Online — convocação e escolha de vagas) e **LOCUS** (gestão do quadro funcional, lotação, importação de dados e permissões). Ambos compartilham login (Portal SIGLA + CoreSSO), base de dados `DB_SIGLA` e padrão de microsserviços Python/Django em Kubernetes (Rancher).

Em outubro/2026 o LOCUS tem **frontend em construção com dados estáticos**. A wiki atribui ao LOCUS dois microsserviços já existentes — **MS-ImportaArquivos** e **MS-AdminUsuarios** — e esta documentação propõe um terceiro, o **MS-Locus**, para o domínio de servidores, unidades e lotação (ADR-004).

### 1.1 — Mapa de produtos e repositórios

| Componente | Repositório | Tecnologia principal | Função no LOCUS | Situação |
|------------|-------------|----------------------|-----------------|----------|
| **Portal SIGLA** | `SME-SIGLA-Frontend` (espelhado em `modules/sigla` do LOCUS) | React 19 / Vite | Login, recuperação de senha, seleção ALVO / LOCUS | Existente |
| **LOCUS Frontend** | `SME-SIGLA-LOCUS-Frontend` | React 19, Vite 7, TypeScript, Ant Design 5, React Query, Axios, Zod, styled-components; Nginx alpine | Telas de UE, servidor, atualizações, menus do domínio | **Em construção**; contratos prontos, respostas estáticas |
| **MS-AdminUsuarios** | `SME-SIGLA-MS-AdminUsuarios` | Django 5.2, DRF (Django REST Framework) 3.15, SimpleJWT, `sigla-sdk` | Login via CoreSSO, JWT (JSON Web Token), usuários, grupos, senha, e-mail | Existente; grupos orientados ao ALVO |
| **MS-ImportaArquivos** | `SME-SIGLA-MS-ImportaArquivos` | Django 5.2 / DRF + Celery 5.4 + Redis | Layouts, importação CSV/TXT, PRODAM, exportações SIGPEC | Existente (uso ALVO); **reaproveitado** (ADR-005) |
| **MS-Locus** | *a criar* | Django 5.2 / DRF + Celery + `sigla-sdk` | Servidores, UE, módulos, lotação/exercício, movimentações, vacância, histórico, conciliação, relatórios | **TO-BE** |
| **sigla-sdk** | `SME-SIGLA-SDK-Microsservico` (tag `1.1.1`) | Python | `http_client`, `CorrelationIdMiddleware`, `AuditlogJWTMiddleware`, `ApiKeyAuthentication` (`X-API-Key`), `CustomJsonFormatter` | Existente |

---

## 2. Diagrama de Arquitetura (containers)

```{image} ../diagramas/assets/03-locus-containers.svg
:width: 100%
:alt: Diagrama de containers do LOCUS
```

---

## 3. Respostas às Questões Solicitadas (visão consolidada)

### 3.1 — Qual tipo de conexão?

Comunicação **síncrona HTTP/REST** (HTTPS) entre frontend e microsserviços e entre microsserviços; **assíncrona via Celery + Redis** para cargas, conciliação e relatórios.

#### Clientes → APIs

| Origem | Destino | Protocolo |
|--------|---------|-----------|
| Portal SIGLA | MS-AdminUsuarios `/api/v1/login`, `esqueci-minha-senha`, `criar-nova-senha` | HTTPS + JSON |
| LOCUS SPA (Single Page Application) | MS-AdminUsuarios `/meus-dados`, permissões | HTTPS + `Authorization: Bearer <JWT>` |
| LOCUS SPA | MS-Locus `/api/v1/unidades-educacionais/*`, `/api/v1/servidores/*` (TO-BE) | HTTPS + JWT |
| Telas de importação (TO-BE) | MS-ImportaArquivos (layouts, importações) | HTTPS multipart + JWT |
| Todos | Ingress Nginx `api-sigla.sme.prefeitura.sp.gov.br/ms-*` | Roteamento por path |

#### APIs / Workers → dados e integrações

| Conexão | Driver / padrão |
|---------|-----------------|
| PostgreSQL SIGLA | Django ORM (psycopg) — write model por domínio; read model (CQRS (Command Query Responsibility Segregation)) TO-BE |
| Redis | Broker Celery (filas dedicadas por MS, ex.: `importa_arquivos`; proposta `locus`) |
| CoreSSO / API SME Integração | `requests` — `POST /api/v1/autenticacao/externa` (header `x-api-eol-key`), `AutenticacaoSgp/*` |
| PRODAM | `api_prodam` (timeout 300 s) |
| MS → MS | `sigla-sdk.http_client` + `X-API-Key`, timeout 30 s |

**Não há GraphQL, gRPC nem RabbitMQ** nos repositórios analisados.

---

### 3.2 — Utiliza cache no sistema? Qual?

**Não há cache de aplicação** nos MS analisados; Redis é usado apenas como **broker Celery**.

| Camada | Mecanismo | Uso |
|--------|-----------|-----|
| LOCUS SPA | React Query (memória do navegador) | Cache de consultas por chave, invalidação após mutação |
| LOCUS SPA | `localStorage` | `TOKEN` JWT e dados do usuário |
| MS-AdminUsuarios / MS-ImportaArquivos | — | Sem cache |
| MS-Locus (TO-BE) | **Read model** PostgreSQL | Consultas e relatórios; cache Redis só se medição indicar (≤ 3 s) |

---

### 3.3 — Existe padrão de observabilidade?

**Sim, padronizado pelo `sigla-sdk`** e pela wiki (Grafana + Elastic).

| Capacidade | MS-AdminUsuarios | MS-ImportaArquivos | MS-Locus (TO-BE) | Frontend |
|------------|------------------|--------------------|------------------|----------|
| Logs JSON estruturados | Sim (`CustomJsonFormatter`) | Sim | Obrigatório | Console |
| Correlation id | Sim (`CorrelationIdMiddleware`) | Sim | Obrigatório | Não propaga |
| APM (Application Performance Monitoring) | `elastic-apm` | `elastic-apm` | Obrigatório | — |
| Log de chamadas HTTP | — | `LogRequestHttp` | Recomendado | — |
| Auditoria de dados | `django-auditlog` | `django-auditlog` | `django-auditlog` + versões (ADR-008) | — |
| Health checks / probes | Não evidenciado | Não evidenciado | `/health/live`, `/health/ready` | Nginx |

---

### 3.4 — Utiliza padrões de projeto? Quais?

| Padrão | Onde |
|--------|------|
| Microsserviço por domínio | SIGLA (wiki) — LOCUS: AdminUsuarios, ImportaArquivos, MS-Locus |
| Camadas `api → serializers → services → repository → models` | MS SIGLA existentes; adotado no MS-Locus |
| Repository | `repository.py` nos MS |
| Service layer | `services/*` (ex.: `importacao_lotes`, `AutenticacaoService`) |
| CQRS | `DB_SIGLA` (wiki): write models por domínio + read models por processos assíncronos |
| Worker / fila | Celery + beat |
| Strategy (validação por layout) | `validacao_*` no MS-ImportaArquivos |
| Frontend: páginas → hooks (React Query) → serviços → cliente HTTP; validação Zod | LOCUS Frontend |
| Contrato primeiro | Schemas Zod e `servicos/*` do frontend definem a API esperada do MS-Locus |

---

### 3.5 — Pontos de entrada têm padrão de segurança?

| Ponto | Mecanismo | Lacuna |
|-------|-----------|--------|
| Portal → login | CoreSSO via MS-AdminUsuarios → JWT | Sem MFA |
| LOCUS SPA | Bearer do `localStorage`; 401 limpa sessão, 403 → `/403` | **Rotas LOCUS sem AuthGuard** (só `modules/sigla` tem) |
| MS-AdminUsuarios | Session, Token, JWT, ApiKey | **`CORS_ALLOW_ALL_ORIGINS=True`**; access token de **7 dias** |
| MS-ImportaArquivos | JWT HS256 (24 h) + ApiKey | `CORS_ALLOW_ALL_ORIGINS=True`; chave JWT cai para `SECRET_KEY` |
| MS → MS | `X-API-Key` (`sigla-sdk`) | Rotação não documentada |
| Ingress | TLS | Rate limiting / WAF (Web Application Firewall) não evidenciados |

Detalhes e plano em [Segurança](../seguranca/index.md).

---

### 3.6 — Utiliza mensageria?

**Sim, via Celery com Redis como broker** (não há RabbitMQ/Kafka).

| Fila | Produtor | Consumidor | Uso |
|------|----------|------------|-----|
| `importa_arquivos` | MS-ImportaArquivos / beat | Worker ImportaArquivos | Importações (beat `enfileirar_importacoes_escolhas` a cada 1 min); timeouts 300/360 s |
| `locus` (TO-BE) | MS-Locus / beat | Worker MS-Locus | Conciliação diária, relatórios assíncronos, projeção do read model |

---

### 3.7 — Padrão de branches (`test` / `homolog` / `master`)

Os repositórios LOCUS já usam `test` → `homolog` → `master`, com `CODEOWNERS`. Ambientes: `qa-sigla` / `qa-api-sigla`, `hom-sigla` / `hom-api-sigla`, `sigla` / `api-sigla` (`.sme.prefeitura.sp.gov.br`). Ver ADR-010.

---

### 3.8 — Infra Kubernetes e dados (síntese)

| Item | Situação |
|------|----------|
| Orquestração | Kubernetes via Rancher (wiki) — **a confirmar** para o LOCUS |
| API Gateway | Ingress Nginx (wiki); Keycloak, TLS, rate limiting previstos |
| CI/CD | Jenkins (Jenkinsfile mantido pela Infra Ágil, fora dos repositórios) + Registry SME |
| Banco | PostgreSQL `DB_SIGLA` com CQRS |
| Arquivos | `media/` local no pod do MS-ImportaArquivos — **precisa de PVC ou object storage** |
| Imagens | Python 3.12 (MS), `nginx:alpine` (front, `VITE_BASE_PATH=/locus/`) |
| Testes | Cobertura mínima de **80%** (wiki); frontend com Jest |

---

## 4. Sistemas em detalhe

### 4.1 — LOCUS Frontend — `SME-SIGLA-LOCUS-Frontend`

- Rotas (`src/rotas/caminhos.ts`): `/cadastro/gestao-unidades-educacionais`, `/cadastro/registrar-unidade-educacional`, `/cadastro/unidade-educacional/:codigoLotacao`, `/cadastro/atualizacao/registro-funcional[/:rf]`, `/sigla/*` (espelho do portal).
- Menu: Início; Cadastro (Inclusão; Atualização: Unidade de exercício, Unidade de lotação, Sem RF ou código da unidade, Tornar um ato sem efeito, Por RF, Vacância; Classificação); Relatórios consultas; Data base; Vagas; Remoção; Integração.
- Serviços (`src/servicos/recursos/*`) já desenham a API do MS-Locus: `GET /api/v1/unidades-educacionais/lotacao/{codigo}/`, `POST /api/v1/unidades-educacionais/`, `GET|DELETE /{codigo}/`, `/{codigo}/modulos/`, `/{codigo}/componentes/{id}/lotados/`, `/{codigo}/componentes/{id}/afastados/`, `/{codigo}/historico/`, `GET /api/v1/servidores/`.
- Hoje as páginas usam `src/paginas/*/dados/dadosEstaticos.ts`; usuário logado é mock.
- Lacunas: sem AuthGuard nas rotas LOCUS; `VITE_LOCUS_API_URL` não é passada no build Docker nem injetada em runtime (`window.__ENV__` declarado, entrypoint só substitui `SERVER_NAME`); cliente SIGLA com URL de QA fixa.

### 4.2 — MS-AdminUsuarios — `SME-SIGLA-MS-AdminUsuarios`

- Endpoints: `login/`, `esqueci-minha-senha/`, `criar-nova-senha/`, `criar-usuario/`, `buscar-usuario-eol/`, `meus-dados/`, `alterar-senha/`, `alterar-email/`, permissões.
- `AutenticacaoService` → `{SME_INTEGRACAO_URL}/api/v1/autenticacao/externa` (`codigoSistema=1`); `SmeIntegracaoService` → `AutenticacaoSgp`.
- Grupos atuais: Admin, Convocador, Visualizador, SuperAdmin, inicial — **sem perfis LOCUS** (ADR-003).

### 4.3 — MS-ImportaArquivos — `SME-SIGLA-MS-ImportaArquivos`

- Apps `importa_arquivos` (layouts, importações de habilitados, vagas, lotes, escolhas; `Layout`, `Importacao*`, `ImportacaoErro`, `LogRequestHttp`) e `exporta_arquivo` (vagas SIGPEC, vagas/candidatos por processo, lotes).
- Clientes para os MS do ALVO e para a PRODAM. Para o LOCUS: **novos tipos de importação** (servidores/unidades EOL, dados funcionais SIGPEC) e cliente `api_locus` (ADR-005).

### 4.4 — MS-Locus (TO-BE)

Ver [componente backend](../diagramas/componente/componente-backend.md). Apps sugeridas: `unidades`, `servidores`, `lotacao`, `movimentacoes`, `conciliacao`, `relatorios`, `core` (motivos, portarias, DRE, componentes curriculares).

---

## 5. Fluxos principais

### 5.1 — Login e acesso ao LOCUS

```text
Usuário → Portal SIGLA → MS-AdminUsuarios /login → CoreSSO (API SME Integração)
       ← JWT (localStorage TOKEN) → seleção de sistema → /locus/ → APIs com Bearer
```

### 5.2 — Integração e conciliação diária (proposta)

```text
Beat (madrugada) → Worker ImportaArquivos → EOL/PRODAM → valida layout → MS-Locus /integracao (X-API-Key)
→ Worker MS-Locus (ConciliacaoService) → divergências + histórico → tela de divergências (DICAR)
SIGPEC → arquivo → upload pelo operador → mesmo caminho a partir da validação
```

Detalhe em [Integrações EOL / SIGPEC](integracoes_eol_sigpec.md).

### 5.3 — Atualização funcional com histórico

```text
Operador → DetalheServidor → POST atualização (motivo, portaria, data, documento)
→ MS-Locus valida (serializer) → LotacaoService/MovimentacaoService → write model + auditlog/versão
→ projeção assíncrona no read model
```

### 5.4 — Relatório pesado

```text
Operador solicita → MS-Locus cria job (202 Accepted) → Worker gera arquivo a partir do read model
→ tela consulta status → download
```

---

## 6. Avaliação consolidada

| Dimensão | Situação | Nota |
|----------|----------|------|
| Padronização de stack | MS seguem o mesmo padrão Django/DRF + `sigla-sdk` | Boa |
| Contratos de API | Frontend já define contratos (Zod) | Boa — falta OpenAPI do MS-Locus |
| Backend de domínio | Inexistente | **Crítico** para o MVP |
| Integração EOL/SIGPEC | Mecanismo indefinido | **Crítico** |
| Segurança | CORS aberto, rotas sem guarda, JWT longo | **Alta prioridade** |
| Observabilidade | APM + logs JSON + correlation id | Boa — faltam health checks e dashboards |
| Esteira | Branches corretas; Jenkinsfile fora do repo | Média |

### Páginas do drawio

| Página | Conteúdo | SVG |
|--------|----------|-----|
| C4 | Legenda | `01-c4-legenda.svg` |
| C1 - Contexto LOCUS | Contexto do sistema | `02-locus-contexto.svg` |
| C2 - Containers LOCUS | Containers | `03-locus-containers.svg` |
| C3 - Componentes LOCUS Frontend | Componentes do frontend | `04-locus-componente-frontend.svg` |
| C3 - Componentes MS-Locus (TO-BE) | Componentes do MS de domínio | `05-locus-componente-ms-locus.svg` |
| C3 - Componentes MS-ImportaArquivos | Componentes (uso LOCUS) | `06-locus-componente-ms-importa-arquivos.svg` |
| C3 - Componentes MS-AdminUsuarios | Componentes | `07-locus-componente-ms-admin-usuarios.svg` |
| FLUXO - Integração e conciliação | Fluxo EOL/SIGPEC proposto | `08-locus-fluxo-integracao-eol-sigpec.svg` |
| SISTEMA - Pipeline CI/CD | Esteira | `09-locus-pipeline-cicd.svg` |
| SISTEMA - Matriz Risco de Segurança | Matriz de risco | `10-locus-matriz-risco.svg` |

---

## 7. Pontos de falha e mitigação

| Ponto de falha | Efeito | Mitigação |
|----------------|--------|-----------|
| CoreSSO / API SME Integração indisponível | Ninguém faz login | Timeout curto + mensagem clara; sessão vigente continua |
| PRODAM/EOL fora na janela de carga | Conciliação do dia não roda | Retry com backoff; alerta; LOCUS segue com dados do dia anterior |
| Arquivo SIGPEC fora do layout | Carga rejeitada | Validação por layout + `ImportacaoErro` exibido ao operador |
| Redis indisponível | Filas param | Probes; alertas; tarefas idempotentes |
| Pod do MS-ImportaArquivos reinicia | Perda de arquivos em `media/` | PVC (PersistentVolumeClaim) ou object storage |
| Read model desatualizado | Relatórios divergentes da tela | Monitorar atraso da projeção; reprocessamento |
| Chave JWT compartilhada vazada | Token forjado em todos os MS | Chave dedicada (`JWT_SIGNING_KEY`), rotação, migrar para RS256 / Keycloak |
