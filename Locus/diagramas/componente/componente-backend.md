# Componente: backends e workers do LOCUS

Os backends do LOCUS seguem o padrão dos MS SIGLA: `api` → `serializers` → `services` → `repository` → `models`, com `sigla-sdk` para middlewares, autenticação entre MS e logs. Arquivo único, separado por microsserviço.

---

## 1. MS-Locus (TO-BE)

```{image} ../assets/05-locus-componente-ms-locus.svg
:width: 100%
:alt: Componentes (C3) do MS-Locus
```

| Componente | Responsabilidade |
|------------|------------------|
| Middlewares `sigla-sdk` | `CorrelationIdMiddleware`, `AuditlogJWTMiddleware` |
| Autenticação e permissões | SimpleJWT (token do MS-AdminUsuarios), `ApiKeyAuthentication` entre MS, perfis LOCUS |
| OpenAPI | drf-spectacular (`/api/schema`, Swagger) |
| Observabilidade | elastic-apm, logs JSON sem dados sensíveis |
| API Unidades | `/api/v1/unidades-educacionais/*` |
| API Servidores | `/api/v1/servidores/*` |
| API Integração interna | `/api/v1/integracao/` (`X-API-Key`) — lotes do MS-ImportaArquivos |
| Serializers | Contratos alinhados aos schemas Zod do front |
| Services de domínio | `UnidadeService`, `ServidorService`, `LotacaoService`, `MovimentacaoService` |
| Serviço de Conciliação | Confronto LOCUS × EOL/SIGPEC, divergências |
| Histórico e versões | `django-auditlog` + snapshots (ADR-008) |
| Repositórios | Escrita no write model, leitura no read model |
| Models | Servidor, Vínculo, Cargo, UnidadeEducacional, DRE, ComponenteCurricular, Módulo, Lotação, Exercício, Afastamento, Movimentação, Divergência |
| Tasks Celery | Conciliação diária, projeção do read model, relatórios |
| Relatórios e exportações | Quadro funcional e lotação (ADR-011) |

---

## 2. MS-ImportaArquivos (uso LOCUS)

```{image} ../assets/06-locus-componente-ms-importa-arquivos.svg
:width: 100%
:alt: Componentes (C3) do MS-ImportaArquivos
```

| Componente | Responsabilidade |
|------------|------------------|
| API Layouts | Layout posicional (txt/csv), campos, posições, tamanhos |
| API Importações | Habilitados, vagas, lotes SIGPEC, escolhas PRODAM; histórico e erros |
| Importações LOCUS (TO-BE) | Servidores/unidades EOL, dados funcionais SIGPEC |
| API Exportações | Vagas SIGPEC, candidatos e lotes por processo (ALVO) |
| Validação de layout | `validacao_*` + schema; `ImportacaoErro` |
| Services de importação | `importacao_lotes`, `importacao_escolhas_service` |
| Tasks Celery | `enfileirar_importacoes_escolhas`; cargas LOCUS (proposta) |
| Repositório | `Layout`, `Importacao*`, `ImportacaoErro`, `LogRequestHttp` |
| Clientes HTTP de MS | `api_candidatos`, `api_escolhas`, `api_concursos`, `api_agendas`, `api_processos_convocacao`; `api_locus` (TO-BE) |
| Cliente PRODAM | `api_prodam` |

---

## 3. MS-AdminUsuarios

```{image} ../assets/07-locus-componente-ms-admin-usuarios.svg
:width: 100%
:alt: Componentes (C3) do MS-AdminUsuarios
```

| Componente | Responsabilidade |
|------------|------------------|
| LoginView | `/api/v1/login` |
| AutenticacaoService | API SME Integração `/api/v1/autenticacao/externa` |
| SimpleJWT | Access/refresh (7 dias hoje) |
| Views de senha e dados | esqueci/nova senha, alterar senha/e-mail, meus dados, buscar usuário EOL |
| SmeIntegracaoService | `AutenticacaoSgp` (dados, senha, e-mail) |
| TokenService + EmailService | Token de redefinição e envio |
| Views de usuários e permissões | Criar usuário, grupos, permissões |
| Grupos e permissões | `load_initial_permissions` — incluir perfis LOCUS |
| UserRepository | Usuários e grupos locais |

---

## 4. Diretrizes para o desenvolvedor

1. **Regra de negócio só no MS-Locus**; MS-ImportaArquivos valida formato e entrega.
2. **Toda alteração funcional gera versão** (motivo, portaria, data, autor).
3. **Idempotência** em `/integracao/` (chave `importacao_id` + chave natural).
4. **CORS explícito** e `JWT_SIGNING_KEY` dedicada; não usar `CORS_ALLOW_ALL_ORIGINS`.
5. **Segredos** fora do Git; `env.example` só com placeholders.
6. **Health checks** `/health/live` e `/health/ready`.
7. **Cobertura ≥ 80%** (wiki).

---

## 5. Relação com ADRs

| ADR | Tema |
|-----|------|
| ADR-001 | Django / DRF + `sigla-sdk` |
| ADR-002 | CoreSSO + JWT |
| ADR-003 | Perfis LOCUS |
| ADR-004 | MS-Locus |
| ADR-005 | Integração via MS-ImportaArquivos |
| ADR-006 | CQRS |
| ADR-007 | Celery + Redis |
| ADR-008 | Histórico e versões |
| ADR-011 | Relatórios assíncronos |
