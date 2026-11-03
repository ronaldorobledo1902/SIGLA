# ADR-002 — Autenticação: CoreSSO via MS-AdminUsuarios + JWT (JSON Web Token)


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Accepted (as-is)                        |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Arquitetura SIGLA / Levantamento LOCUS  |
| **Sistema**      | SIGLA (Portal, ALVO, LOCUS)             |
| **Relacionadas** | ADR-003, ADR-009                        |


---

## Contexto

Os usuários do SIGLA usam a credencial única da SME (a mesma do antigo sistema ESCOLHAS). A wiki define o **CoreSSO** como provedor e cita o **Keycloak** como possibilidade em estudo pela SME-COTIC. O PO pediu verificação do IdP (Identity Provider) corporativo e recomenda MFA (autenticação multifator).

## Decisão

1. O **Portal SIGLA** autentica no **MS-AdminUsuarios** (`/api/v1/login`), que valida as credenciais na **API SME Integração** (`/api/v1/autenticacao/externa`, `codigoSistema=1`) e sincroniza o usuário local.
2. O MS-AdminUsuarios emite **JWT** (SimpleJWT); o frontend guarda em `localStorage` (`TOKEN`) e envia `Authorization: Bearer` a todos os MS.
3. Cada MS valida o JWT localmente; chamadas entre MS usam **`X-API-Key`** (`sigla-sdk`).
4. Keycloak e MFA permanecem **em estudo**; se adotados, substituem a emissão de JWT pelo MS-AdminUsuarios sem mudar os MS consumidores (validação por JWKS).

## Consequências

### Positivas

- Login único e reaproveitamento das credenciais existentes
- MS independentes do IdP (só validam o token)

### Negativas

- Chave HS256 compartilhada entre MS (`SECRET_KEY` / `JWT_SIGNING_KEY`): vazamento compromete todos
- Validade de 7 dias (AdminUsuarios) e 24 h (ImportaArquivos) — inconsistente e longa para dados LGPD
- Token em `localStorage` é exposto a XSS (Cross-Site Scripting)

## Alternativas consideradas

| Alternativa | Motivo de não adoção (atual) |
|-------------|------------------------------|
| Keycloak (OIDC (OpenID Connect)) já no MVP | Em estudo pela SME-COTIC; sem data |
| Sessão Django com cookie | Não atende múltiplos MS e fronts |
| Login próprio do LOCUS | Duplicaria credenciais e contraria a premissa de login único |

## Referências

- `SME-SIGLA-MS-AdminUsuarios` — `AutenticacaoService`, `SmeIntegracaoService`, `SIMPLE_JWT`
- `SME-SIGLA-LOCUS-Frontend/src/servicos/http.ts`
- Wiki SIGLA — Jornadas › Login
