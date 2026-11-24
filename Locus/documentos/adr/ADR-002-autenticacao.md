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

O login do SIGLA valida a credencial na **API SME Integração** (`/api/v1/autenticacao/externa`, `codigoSistema=1`) e o MS-AdminUsuarios emite o JWT. Não há outro provedor de identidade no código. O PO recomenda MFA (autenticação multifator).

## Decisão

1. O **Portal SIGLA** autentica no **MS-AdminUsuarios** (`/api/v1/login`), que valida as credenciais na **API SME Integração** (`/api/v1/autenticacao/externa`, `codigoSistema=1`) e sincroniza o usuário local.
2. O MS-AdminUsuarios emite **JWT** (SimpleJWT); o frontend guarda em `localStorage` (`TOKEN`) e envia `Authorization: Bearer` a todos os MS.
3. Cada MS valida o JWT localmente; chamadas entre MS usam **`X-API-Key`** (`sigla-sdk`).
4. MFA não está implementado.

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
| Sessão Django com cookie | Não atende múltiplos MS e fronts |
| Login próprio do LOCUS | Duplicaria credenciais e contraria a premissa de login único |

## Referências

- `SME-SIGLA-MS-AdminUsuarios` — `AutenticacaoService`, `SmeIntegracaoService`, `SIMPLE_JWT`
- `SME-SIGLA-LOCUS-Frontend/src/servicos/http.ts`
