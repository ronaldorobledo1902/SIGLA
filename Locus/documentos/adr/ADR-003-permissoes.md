# ADR-003 — Permissões de ALVO e LOCUS geridas no LOCUS, com perfis LOCUS


| Campo            | Valor                                   |
| ---------------- | --------------------------------------- |
| **Status**       | Proposed                                |
| **Data**         | 2026-10-27                              |
| **Decisores**    | Arquitetura SIGLA / PO LOCUS            |
| **Sistema**      | LOCUS + MS-AdminUsuarios                |
| **Relacionadas** | ADR-002                                 |


---

## Contexto

A wiki determina que **o cadastro de permissões de ALVO e LOCUS fica no LOCUS** (cadastrar, consultar, alterar e excluir usuários e permissões). O MS-AdminUsuarios hoje só tem grupos do ALVO (Admin, Convocador, Visualizador, SuperAdmin, inicial). O PO sugere três perfis mínimos para o LOCUS e acesso restrito ao mínimo necessário para outras áreas. As rotas LOCUS do frontend não possuem guarda de autenticação.

## Decisão

1. **Dono das permissões:** MS-AdminUsuarios (Django groups + permissions), com **telas no LOCUS** para administrar usuários de ALVO e LOCUS.
2. **Perfis LOCUS iniciais:**

| Perfil | Permissões |
|--------|------------|
| `locus_administrador` | Tudo do LOCUS + gestão de usuários/permissões + layouts de importação |
| `locus_operador_dicar` | Consultar e alterar servidores, UEs, lotação/exercício, movimentações; importar arquivos; tratar divergências |
| `locus_consulta` | Somente leitura e relatórios (dados pessoais mascarados) |

3. O JWT carrega os **grupos**; cada MS aplica `permission_classes` por endpoint; o frontend aplica **AuthGuard** + guarda por perfil nas rotas e menus.
4. Abrangência por DRE fica como evolução (claim adicional), se o PO confirmar.

## Consequências

### Positivas

- Um único lugar para governar acesso aos dois subsistemas
- Atende LGPD (Lei Geral de Proteção de Dados) com privilégio mínimo

### Negativas

- MS-AdminUsuarios passa a conhecer perfis dos dois subsistemas (acoplamento aceitável)
- Mudança de perfil só vale após renovação do token (mitigar com validade curta)

## Alternativas consideradas

| Alternativa | Motivo de não adoção |
|-------------|----------------------|
| Permissões dentro do MS-Locus | Duplicaria o cadastro de usuários e contraria a wiki |
| Papéis no Keycloak | Depende do estudo SME-COTIC; pode ser migração futura |

## Referências

- Wiki SIGLA — Jornadas › Cadastro de usuários do sistema
- `SME-SIGLA-MS-AdminUsuarios` — `load_initial_permissions`
- Questionário do PO — §2 e §8
