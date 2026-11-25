# Contexto — Acesso, usuários e permissões

## Contexto

Todo acesso ao SIGLA começa no **Portal SIGLA** com a credencial única da SME (CoreSSO). O **MS-AdminUsuarios** valida na API SME Integração e emite o JWT (JSON Web Token). O front do ALVO e o front do Locus falam com um **BFF** único, que valida o token e chama os microsserviços. Os grupos existentes são do ALVO. O LOCUS não administra as permissões dos dois subsistemas.

```{image} ../assets/07-locus-componente-ms-admin-usuarios.svg
:width: 100%
:alt: Componentes do MS-AdminUsuarios
```

---

## Objetivos

- Login único, sem credencial nova
- Perfis LOCUS com privilégio mínimo (LGPD (Lei Geral de Proteção de Dados))
- Um único ponto de governança de acesso para ALVO e LOCUS

---

## Escopo

### Em escopo

- Login, esqueci minha senha, criar nova senha, alterar senha/e-mail, meus dados
- Cadastro de usuário a partir do CoreSSO/EOL (`buscar-usuario-eol`, `criar-usuario`)
- Grupos e permissões; perfis LOCUS `locus_administrador`, `locus_operador_dicar`, `locus_consulta` (ADR-003)
- AuthGuard e guarda por perfil no frontend LOCUS

### Fora de escopo

- Gestão de credenciais (continua no CoreSSO)
- Outro provedor de identidade (não há no código)

---

## Fluxo resumido

```text
Usuário → Front do ALVO ou do Locus → BFF (valida o token)
        → MS-AdminUsuarios /login → API SME Integração (CoreSSO)
        ← JWT → BFF consulta os microsserviços e devolve o DTO da tela
Administrador → MS-AdminUsuarios (grupos do ALVO) → Base de Dados SIGLA (auth_user, grupos)
Tela de permissões do LOCUS → proposta (ADR-003)
```

---

## Sistemas envolvidos

| Sistema | Interação |
|---------|-----------|
| Portal SIGLA | Login e seleção de sistema |
| BFF | Recebe a tela, valida o token, agrega e devolve o DTO. Único para ALVO e Locus |
| MS-AdminUsuarios | Autenticação, emissão de JWT, usuários, grupos, senha |
| API SME Integração / CoreSSO | Valida credenciais, dados do usuário, redefinição de senha |
| E-mail SME | Envio do link de redefinição |
| Base de Dados SIGLA | Usuários e grupos locais |

---

## Considerações

- Hoje os grupos são só do ALVO; perfis LOCUS precisam existir **antes** de expor dados reais.
- O access token de 7 dias deve ser reduzido (ver [Segurança](../../seguranca/index.md)).
- MFA (autenticação multifator) não está implementado.

## Referências

- `SME-SIGLA-MS-AdminUsuarios` — `AutenticacaoService`, grupos em `load_initial_permissions`
